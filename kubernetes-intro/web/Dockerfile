FROM alpine
RUN adduser -D -u 1001 -g 'www' www
RUN apk update && apk add nginx 
RUN mkdir /app /run/nginx && \
    chown -R www:www /run/nginx /var/lib/nginx /var/log/nginx /app
ADD nginx.conf /etc/nginx/nginx.conf
USER 1001
EXPOSE 8000
CMD ["nginx", "-g", "daemon off;"]
#ENTRYPOINT [ "nginx" ]
#CMD [ "-c", "/etc/nginx/nginx.conf"]
