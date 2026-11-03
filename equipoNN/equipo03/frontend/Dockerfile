FROM nginxinc/nginx-unprivileged:alpine

COPY default.conf /etc/nginx/conf.d/default.conf

COPY index.html /usr/share/nginx/html/

HEALTHCHECK --interval=15s --timeout=3s --start-period=5s --retries=3 \
  CMD wget -qO /dev/null http://127.0.0.1:8080/ || exit 1

EXPOSE 8080