FROM mongo:7.0

# 한국 시간대 설정
RUN apt-get update && \
    apt-get install -y tzdata && \
    ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime && \
    echo "Asia/Seoul" > /etc/timezone && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# MongoDB 설정 파일 복사
COPY mongod.conf /etc/mongod.conf

# 초기화 스크립트 복사
COPY init/ /docker-entrypoint-initdb.d/

# 데이터 디렉토리 생성
RUN mkdir -p /data/db /data/configdb && \
    chown -R mongodb:mongodb /data/db /data/configdb

# 헬스체크
HEALTHCHECK --interval=10s --timeout=5s --retries=5 \
    CMD mongosh --eval "db.adminCommand('ping')" --quiet || exit 1

EXPOSE 27017

# 레이블 추가
LABEL maintainer="EATi Project"
LABEL version="7.0"
LABEL description="MongoDB 7.0 for EATi Project"

# MongoDB 설정 파일로 실행
CMD ["mongod", "--config", "/etc/mongod.conf"]
