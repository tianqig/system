FROM nginx

COPY nginx.conf /etc/nginx/
COPY nginx.conf.d/default.conf /etc/nginx/conf.d
COPY docker-entry.sh /
RUN chmod +x /docker-entry.sh
ENTRYPOINT ["/docker-entry.sh"]

EXPOSE 80
EXPOSE 443
VOLUME /etc/nginx
VOLUME /etc/nginx/conf.d
WORKDIR /etc/nginx

CMD ["nginx", "-g", "daemon off;"]
