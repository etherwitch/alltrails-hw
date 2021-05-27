FROM nginx:latest
RUN apt-get update && \
    apt-get install -y ca-certificates && \
    /usr/sbin/update-ca-certificates && \
    apt-get upgrade -y && \
    apt-get clean && \
    apt autoremove -y && \
    rm -rf /var/lib/apt/lists/*
ADD --chown=nginx:root nginx.conf /etc/nginx/nginx.conf
RUN chown -R nginx:root /etc/nginx /var/cache/nginx && \
    chmod -R g=u /etc/nginx /var/cache/nginx

USER nginx

