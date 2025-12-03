FROM redis:7.2-alpine

ENV TZ=Asia/Seoul

COPY redis.conf /usr/local/etc/redis/redis.conf

EXPOSE 6379

CMD ["redis-server", "/usr/local/etc/redis/redis.conf"]
