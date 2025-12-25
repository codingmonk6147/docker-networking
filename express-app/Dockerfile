FROM alpine:3.18

RUN apk update && \
    apk add --no-cache nodejs npm nginx supervisor

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY app.js .
COPY nginx.conf /etc/nginx/nginx.conf
COPY supervisord.conf /etc/supervisord.conf

EXPOSE 80

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
