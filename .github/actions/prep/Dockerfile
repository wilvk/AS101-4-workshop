FROM alpine
RUN apk add --update ansible
COPY entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]