FROM nginx:1.21-alpine as dev

COPY .docker/nginx/service.conf /etc/nginx/conf.d/default.conf
COPY .docker/nginx/nginx.conf /etc/nginx/nginx.conf

FROM dev as prod
RUN mkdir -p /app/public
