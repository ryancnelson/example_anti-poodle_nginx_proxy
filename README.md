###Procedure to spin up a small zone running an nginx reverse-proxy to forward https requests to another host, with SSL3 **disabled**, to avoid the "poodle" security bug.

This nginx proxy will:
 * listen on port 80 and forward all requests to itself, at the https port 443
 * listen on port 443 with SSLv3 disabled, and proxy connections to another internal service that can't itself be configured to disable SSLv3



----
This procedure was specifically developed to front an SDC 6.5 cloudapi zone, since that service ships with a version of node.js that can't be configured to disable SSLv3,
and as of late 2014, SmartDataCenter 6.5 is on extended support only and isn't shipping new builds.

However, this procedure may be useful as a general case, to provide a protected HTTPS listener, and act as a trusted proxy to an unprotected HTTPS service.

### NOTE: If you are running SDC in production, you may wish to use commercially-supported load-balancers or firewalls to protect web services exposed to the internet.

---

###The procedure:

On your headnode:
* provision a new SmartOS zone with owner "admin", with primary nic on "external" network, and an additional nic on the "admin" network
 * I used a 1GB machine, and the latest base64 image


* log into the newly-provisioned SmartOS zone
 * run ```pkgin up```
 * run ```pkgin install nginx```


* download the ```nginx.conf``` file from this repo, and copy it to ```/opt/local/etc/nginx/nginx.conf```
 * edit this file as necessary, to specify your own IP addresses and key/certificate locations.

* if necessary, generate a self-signed key and certificate:
 * ```cd /opt/local/etc/nginx```
 * ``` openssl req -x509 -nodes -subj '/CN=*' -newkey rsa:4096 -days 730 -keyout ./cert.key -out ./cert.pem```


* enable nginx in your zone
 * ```svcadm enable nginx```


* protect the external listener on the original service from connections from the general internet, using firewall rules or by disabling the external-facing network interface
