Documentation is available at http://nginx.org

This version implemented `proxy_protocol` to mail module

nginx version 1.18.0

### Build dependencies

```
root@build-server:~# apt-get -q -y install --no-install-recommends build-essential git libgd3 libgd-dev libgeoip1 libgeoip-dev geoip-bin libpcre3 libpcre3-dev zlib1g zlib1g-dev openssl libssl-dev autotools-dev libxml2 libxml2-dev libxslt1.1 libxslt1-dev wget libsystemd-dev zlib1g-dev wget ca-certificates
```

### Build

```
root@build-server:~# bash ./build
```

### Example config

```
mail {
  server_name mail.server.com;
  auth_http 127.0.0.1:8080/mail/auth;
  proxy on;
  proxy_pass_error_message on;
  error_log /var/log/nginx/mail_proxy_error.log info;

  starttls on;
  smtp_capabilities PIPELINING ETRN ENHANCEDSTATUSCODES DSN 8BITMIME "SIZE 51200000";
  smtp_auth login plain;
  xclient off;

  pop3_auth plain;
  pop3_capabilities LAST TOP USER PIPELINING UIDL;

  imap_auth login plain;
  imap_capabilities IMAP4rev1 LITERAL+ SASL-IR LOGIN-REFERRALS ID ENABLE IDLE AUTH=PLAIN AUTH=LOGIN;

  ssl_certificate /etc/nginx/ssl/cert.crt;
  ssl_certificate_key /etc/nginx/ssl/cert.key;
  ssl_dhparam /etc/nginx/ssl-dhparams.pem;
  ssl_session_cache shared:mail_SSL:1m;
  ssl_session_timeout 1440m;

  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_prefer_server_ciphers on;

  ssl_ciphers "ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS";

  server {
    listen           127.0.0.1:2525 proxy_protocol;

    protocol         smtp;
    auth_http_header User-Agent "Nginx SMTP proxy";
  }

  server {
    listen           127.0.0.1:2587 proxy_protocol;
    protocol         smtp;
    auth_http_header User-Agent "Nginx SMTP proxy";
  }

  server {
    listen           127.0.0.1:2143 proxy_protocol;
    protocol         imap;
    auth_http_header User-Agent "Nginx POP3/IMAP4 proxy";
  }

  server {
    listen           127.0.0.1:2110 proxy_protocol;
    protocol         pop3;
    auth_http_header User-Agent "Nginx POP3/IMAP4 proxy";
  }

}
```

