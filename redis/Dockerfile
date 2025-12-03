FROM redis:7.2-alpine

# 한국 시간대 설정
RUN apk add --no-cache tzdata && \
    cp /usr/share/zoneinfo/Asia/Seoul /etc/localtime && \
    echo "Asia/Seoul" > /etc/timezone

# Redis 설정 파일 복사
COPY redis.conf /usr/local/etc/redis/redis.conf

# 데이터 디렉토리 생성
RUN mkdir -p /data && chown redis:redis /data

# 헬스체크
HEALTHCHECK --interval=10s --timeout=5s --retries=5 \
    CMD redis-cli ping || exit 1

EXPOSE 6379

# 레이블 추가
LABEL maintainer="EATi Project"
LABEL version="7.2"
LABEL description="Redis 7.2 for EATi Project"

# Redis 설정 파일로 실행
CMD ["redis-server", "/usr/local/etc/redis/redis.conf"]
