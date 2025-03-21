# 🚀 MySQL + Spring Boot + 자동 백업 인프라 구축

## 📌 프로젝트 개요

이 프로젝트는 Docker Compose를 이용하여 MySQL과 Spring Boot 애플리케이션을 컨테이너로 실행하고, 1분마다 데이터베이스를 백업하는 시스템을 구축합니다. 🛠️

- `docker-compose.yml`을 이용해 데이터베이스와 백업, 애플리케이션을 컨테이너화 💾
- `jpa.jar` 자동 실행으로 Spring Boot 애플리케이션 실행 🌱
- `backup.sh`를 활용한 1분 단위 MySQL 자동 백업 🕐

---

## 📂 프로젝트 구조

```
.
├── Dockerfile.web           # Spring Boot 컨테이너 빌드 파일
├── Dockerfile.backup        # DB 백업 컨테이너 빌드 파일
├── docker-compose.yml       # 서비스 구성 파일
├── backup.sh                # MySQL 백업 스크립트
├── compose-up.sh            # Docker Compose 실행 스크립트
├── jpa.jar                  # Spring Boot 실행 파일
└── backup/                  # 백업 파일 저장 디렉토리
```

---

## 🛠️ 사용 기술

- **Docker**: 컨테이너 오케스트레이션 🐳
- **MySQL 8.0**: 데이터베이스 관리 시스템 🗄️
- **Spring Boot**: 백엔드 애플리케이션 프레임워크 ☕
- **Cron & Shell Script**: 자동 백업 스케줄링 및 스크립팅 ⏳

---

## 🏗️ 설치 및 실행 방법

### 1️⃣ Docker Compose 실행

```sh
./compose-up.sh
```

위 명령어를 실행하면 `docker-compose.yml`이 실행되며, 모든 컨테이너가 실행됩니다. 🚀

### 2️⃣ 백업 시스템 동작 확인

1. 백업 파일이 정상적으로 저장되는지 확인:
   ```sh
   ls -lh backup/
   ```
2. 백업 로그 확인:
   ```sh
   docker logs db_backup
   ```

---

## 📝 주요 파일 설명

### 🔹 `compose-up.sh`

```sh
#!/bin/bash
cd 06.dockerCompose
docker-compose up -d
```

Docker Compose를 실행하는 간단한 스크립트입니다.

### 🔹 `backup.sh`

```sh
#!/bin/sh
DB_NAME=fisa
DB_USER=${MYSQL_USER}
DB_PASSWORD=${MYSQL_PASSWORD}
BACKUP_DIR=/backup
MYSQL_HOST=mysqldb2

mkdir -p $BACKUP_DIR
BACKUP_FILE="$BACKUP_DIR/$DB_NAME-$(date +%Y%m%d%H%M%S).sql"
mysqldump -h $MYSQL_HOST -u $DB_USER -p$DB_PASSWORD $DB_NAME > $BACKUP_FILE

echo "Backup completed: $BACKUP_FILE"
```

1분마다 실행되며 MySQL 데이터를 덤프하여 `/backup` 디렉토리에 저장합니다. 🗂️

### 🔹 `docker-compose.yml`

```yaml
services:
  db:
    container_name: mysqldb2
    image: mysql:8.0
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: fisa
      MYSQL_USER: user01
      MYSQL_PASSWORD: user01
    networks:
      - spring-mysql-net
    volumes:
      - db_data:/var/lib/mysql
    healthcheck:
      test: ['CMD-SHELL', 'mysqladmin ping -h 127.0.0.1 -u root --password=$${MYSQL_ROOT_PASSWORD} || exit 1']
      interval: 10s
      timeout: 2s
      retries: 100
```

MySQL 컨테이너를 설정하며, `healthcheck`를 통해 서비스 상태를 모니터링합니다. ⚡

---

## 🎯 마무리

이 프로젝트를 통해 MySQL과 Spring Boot 애플리케이션을 효과적으로 컨테이너화하고, 데이터 백업 자동화까지 구현할 수 있습니다! 🚀

기여 또는 문의 사항이 있다면 언제든지 PR 또는 Issue를 남겨주세요! 💬

