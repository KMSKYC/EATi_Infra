FROM postgres:16-alpine

# 한국 시간대 설정
RUN apk add --no-cache tzdata && \
    cp /usr/share/zoneinfo/Asia/Seoul /etc/localtime && \
    echo "Asia/Seoul" > /etc/timezone

# PostgreSQL 설정 최적화
RUN echo "# Custom PostgreSQL Configuration" >> /usr/share/postgresql/postgresql.conf.sample && \
    echo "timezone = 'Asia/Seoul'" >> /usr/share/postgresql/postgresql.conf.sample && \
    echo "log_timezone = 'Asia/Seoul'" >> /usr/share/postgresql/postgresql.conf.sample && \
    echo "listen_addresses = '*'" >> /usr/share/postgresql/postgresql.conf.sample

# 초기화 스크립트 복사
COPY init/ /docker-entrypoint-initdb.d/

# 헬스체크
HEALTHCHECK --interval=10s --timeout=5s --retries=5 \
    CMD pg_isready -U ${POSTGRES_USER:-postgres} || exit 1

EXPOSE 5432

# 레이블 추가
LABEL maintainer="EATi Project"
LABEL version="16"
LABEL description="PostgreSQL 16 for EATi Project"
