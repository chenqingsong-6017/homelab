FROM alpine:3.20
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/' /etc/apk/repositories && apk add --no-cache busybox-extras
COPY hello.txt /app/index.html
EXPOSE 80
CMD ["httpd", "-f", "-p", "80", "-h", "/app"]
