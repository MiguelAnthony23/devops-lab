FROM nginx:1.27-alpine

RUN apk add --no-cache curl

COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY entrypoint.sh /usr/local/bin/mission-entrypoint
RUN chmod +x /usr/local/bin/mission-entrypoint

ENTRYPOINT ["/usr/local/bin/mission-entrypoint"]
CMD ["nginx", "-g", "daemon off;"]
