FROM tutum/nginx
RUN rm /etc/nginx/sites-enabled/default
ADD default.conf /etc/nginx/sites-enabled/default
WORKDIR /etc/nginx/certs
COPY ./certs /etc/nginx/certs/