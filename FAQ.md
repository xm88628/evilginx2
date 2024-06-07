# Frequently Asked Questions

### Unable to retrieve certificates from LetsEncrypt / Error 403 / Timeout error (likely firewall problem)

LetsEncrypt is not able to contact your server for verification of domain ownership.

1. You need to own a registered domain (use Namecheap for example). Let's say you registered a domain `mydomain.com` and your external server where Evilginx is installed is at IP `123.123.123.123`.
2. In domain configuration you need to add custom nameservers. Add `ns1.mydomain.com` and `ns2.mydomain.com` nameservers and set both to point to IP `123.123.123.123`.
3. Change option for the domain to use `Custom Nameservers` and set the nameserver names to `ns1.mydomain.com` and `ns2.mydomain.com`.

Now Evilginx server will properly respond to any DNS requests coming from the outside and will handle the resolution of any subdomains that you need to manage for your phishlets.

If you want to test Evilginx locally, run it with `-developer` parameter. Then after you set up the phishlet you want to test, get the hosts entries with `phishlets get-hosts <phishlet_name>` and add the output to your `/etc/hosts` file. This will route the hostnames to your local machine.