\#user nobody;

worker\_processes 1;

\#error\_log logs/error.log;

\#error\_log logs/error.log notice;

\#error\_log logs/error.log info;

\#pid logs/nginx.pid;

events {

 worker\_connections 1024;

}

http {

 include mime.types;

 default\_type application/octet-stream;

 \#log\_format main '$remote\_addr - $remote\_user [$time\_local] "$request" '

 \# '$status $body\_bytes\_sent "$http\_referer" '

 \# '"$http\_user\_agent" "$http\_x\_forwarded\_for"';

 \#access\_log logs/access.log main;

 sendfile on;

 \#tcp\_nopush on;

 \#keepalive\_timeout 0;

 keepalive\_timeout 65;

 \#gzip on;

 server {

 listen 443;

 server\_name localhost;

 ssl on;

 ssl\_certificate /usr/local/nginx/cert/214068728840046.pem;

 ssl\_certificate\_key /usr/local/nginx/cert/214068728840046.key;

 ssl\_session\_timeout 5m;

 ssl\_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;

 ssl\_protocols TLSv1 TLSv1.1 TLSv1.2;

 ssl\_prefer\_server\_ciphers on;

 location / {

 root html;

 index index.html index.htm;

 }

 \#error\_page 404 /404.html;

 \# redirect server error pages to the static page /50x.html

 \#

 error\_page 500 502 503 504 /50x.html;

 location = /50x.html {

 root html;

 }

 \# proxy the PHP scripts to Apache listening on 127.0.0.1:80

 \#

 \#location ~ \\.php$ {

 \# proxy\_pass http://127.0.0.1;

 \#}

 \# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000

 \#

 \#location ~ \\.php$ {

 \# root html;

 \# fastcgi\_pass 127.0.0.1:9000;

 \# fastcgi\_index index.php;

 \# fastcgi\_param SCRIPT\_FILENAME /scripts$fastcgi\_script\_name;

 \# include fastcgi\_params;

 \#}

 \# deny access to .htaccess files, if Apache's document root

 \# concurs with nginx's one

 \#

 \#location ~ /\\.ht {

 \# deny all;

 \#}

 }

 \# another virtual host using mix of IP-, name-, and port-based configuration

 \#

 \#server {

 \# listen 8000;

 \# listen somename:8080;

 \# server\_name somename alias another.alias;

 \# location / {

 \# root html;

 \# index index.html index.htm;

 \# }

 \#}

 \# HTTPS server

 \#

 \#server {

 \# listen 443 ssl;

 \# server\_name localhost;

 \# ssl\_certificate cert.pem;

 \# ssl\_certificate\_key cert.key;

 \# ssl\_session\_cache shared:SSL:1m;

 \# ssl\_session\_timeout 5m;

 \# ssl\_ciphers HIGH:!aNULL:!MD5;

 \# ssl\_prefer\_server\_ciphers on;

 \# location / {

 \# root html;

 \# index index.html index.htm;

 \# }

 \#}

}