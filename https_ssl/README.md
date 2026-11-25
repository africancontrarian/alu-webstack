# HTTPS SSL

DNS subdomains for the web stack, and SSL termination on the HAproxy load balancer (lb-01).

## Requirements

- Ubuntu 16.04 LTS
- Bash scripts start with `#!/usr/bin/env bash`, have a comment on line 2, are executable and pass Shellcheck
- HAproxy 1.5 or higher (SSL termination)

## Files

| # | File | Description |
|---|------|-------------|
| 0 | `0-world_wide_web` | Displays the record type and destination of the `www`, `lb-01`, `web-01` and `web-02` subdomains of a domain, or of one subdomain |
| 1 | `1-haproxy_ssl_termination` | `/etc/haproxy/haproxy.cfg` with HAproxy listening on TCP 443 and terminating SSL for `www` |
| 2 | `2-redirect_http_to_https` | Same configuration, with HTTP traffic redirected to HTTPS with a 301 |

## DNS records

| Subdomain | Type | Points to |
|-----------|------|-----------|
| www | A | lb-01 IP |
| lb-01 | A | lb-01 IP |
| web-01 | A | web-01 IP |
| web-02 | A | web-02 IP |

```
$ ./0-world_wide_web DOMAIN
The subdomain www is a A record and points to LB01_IP
The subdomain lb-01 is a A record and points to LB01_IP
The subdomain web-01 is a A record and points to WEB01_IP
The subdomain web-02 is a A record and points to WEB02_IP
```

## Creating the certificate on lb-01

```
root@7310-lb-01:~# apt-get install -y certbot
root@7310-lb-01:~# service haproxy stop
root@7310-lb-01:~# certbot certonly --standalone -d www.DOMAIN
root@7310-lb-01:~# mkdir -p /etc/haproxy/certs
root@7310-lb-01:~# cat /etc/letsencrypt/live/www.DOMAIN/fullchain.pem /etc/letsencrypt/live/www.DOMAIN/privkey.pem > /etc/haproxy/certs/www.DOMAIN.pem
root@7310-lb-01:~# cp 2-redirect_http_to_https /etc/haproxy/haproxy.cfg
root@7310-lb-01:~# service haproxy start
```
