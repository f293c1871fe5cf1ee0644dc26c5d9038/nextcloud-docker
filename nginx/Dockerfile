FROM nginx:1.23.2-alpine

ARG HOST
ARG CERTIFICATE_DURATION
ARG DNS1
ARG DNS2

RUN apk update \
&& apk add openssl

RUN printf ".\n.\n.\n.\n.\n${HOST}\n.\n" \
| openssl req -x509 -nodes -days ${CERTIFICATE_DURATION} -newkey rsa:2048 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt \
&& openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048

RUN mkdir -p /etc/nginx/snippets

# snippets/locations.conf
COPY locations.conf /etc/nginx/snippets

# snippets/self-signed.conf
RUN printf "ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;\n\
ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;" > /etc/nginx/snippets/self-signed.conf

# snippets/ssl-params.conf
RUN printf "ssl_protocols TLSv1 TLSv1.1 TLSv1.2;\n\
ssl_prefer_server_ciphers on;\n\
ssl_ciphers \"EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH\";\n\
ssl_ecdh_curve secp384r1;\n\
ssl_session_cache shared:SSL:10m;\n\
ssl_session_tickets off;\n\
ssl_stapling on;\n\
ssl_stapling_verify on;\n\
resolver ${DNS1} ${DNS2} valid=300s;\n\
resolver_timeout 5s;\n\
add_header Strict-Transport-Security \"max-age=63072000; includeSubdomains\";\n\
add_header X-Frame-Options DENY;\n\
add_header X-Content-Type-Options nosniff;\n\
ssl_dhparam /etc/ssl/certs/dhparam.pem;" > /etc/nginx/snippets/ssl-params.conf

RUN mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.bak

# conf.d/default.conf
RUN printf "server {\n\
    listen 80 default_server;\n\
    listen [::]:80 default_server;\n\
    server_name ${HOST};\n\
    return 301 https://\$server_name\$request_uri;\n\
}\n\
\n\
server {\n\
    listen 443 ssl http2 default_server;\n\
    listen [::]:443 ssl http2 default_server;\n\
    include snippets/self-signed.conf;\n\
    include snippets/ssl-params.conf;\n\
\n\
    server_name ${HOST};\n\
    error_log /var/log/nginx/error.log debug;\n\
\n\
    include snippets/locations.conf;\n\
}" > /etc/nginx/conf.d/default.conf

EXPOSE 80
EXPOSE 443