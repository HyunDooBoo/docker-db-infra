# 🚀 자동화된 MySQL 데이터 보호 시스템 구축

## 📌 프로젝트 개요
이 프로젝트는 Docker Compose를 이용하여 MySQL과 Spring Boot 애플리케이션을 컨테이너로 실행하고, 1분마다 데이터베이스를 백업해주는 서버를 구축하는 것이 목표다. 🛠️

- `docker-compose.yml`을 이용해 데이터베이스와 백업, 애플리케이션을 컨테이너화 💾
- `jpa.jar` 자동 실행으로 Spring Boot 애플리케이션 실행 🌱
- `backup.sh`를 활용한 1분 단위 MySQL 자동 백업 🕐
- **Docker 네트워크**를 활용하여 컨테이너 간 통신 최적화 🌐

---

## 📂 프로젝트 구조
```
.
├── docker_compose_infra/    # Docker 및 백업 관련 파일 폴더
│   ├── Dockerfile.web       # Spring Boot 컨테이너 빌드 파일
│   ├── Dockerfile.backup    # DB 백업 컨테이너 빌드 파일
│   ├── docker-compose.yml   # 서비스 구성 파일
│   ├── backup.sh            # MySQL 백업 스크립트
│   ├── jpa.jar              # Spring Boot 실행 파일
│   ├── backup/              # 백업 파일 저장 디렉토리
├── compose-up.sh            # Docker Compose 실행 스크립트
```

---

## 🛠️ 사용 기술
- **Docker**: 컨테이너 오케스트레이션 🐳
- **MySQL 8.0**: 데이터베이스 관리 시스템 🗄️
- **Spring Boot**: 백엔드 애플리케이션 프레임워크 ☕
- **Cron & Shell Script**: 자동 백업 스케줄링 및 스크립팅 ⏳
- **Docker Network**: 컨테이너 간 통신을 위한 브리지 네트워크 구성 🌐

---

## 🏗️ 설치 및 실행 방법
### 1️⃣ Docker Compose 실행
```
sh
./compose-up.sh
```
위 명령어를 실행하면 `docker-compose.yml`이 실행되며, 모든 컨테이너가 실행된다. 🚀

### 2️⃣ 백업 시스템 동작 확인
1. 백업 파일이 정상적으로 저장되는지 확인:
   ```sh
   ls -lh docker_compose_infra/backup/
   ```
   ![image](https://github.com/user-attachments/assets/a3b39051-66b1-443d-a82d-9542738c07e7)

   cron 설정에 의해 1분에 한 번씩 mysql 백업 파일을 생성해준다.

3. 백업 로그 확인:
   ```sh
   docker logs db_backup
   ```
   ![image](https://github.com/user-attachments/assets/269c9213-0f82-4761-88d2-8a0dde1bb2f6)

   1분에 한 번씩 로그가 뜨는 걸 볼 수 있다!
   
   [Warning]이 뜨는 이유
   - cron이 실행시키는 backup.sh 파일에 패스워드를 하드코딩 해둬서 그렇다.
   - 환경변수로 수정하면 오류가 뜨지 않는다!

---

## 📝 주요 파일 설명
### 🔹 `compose-up.sh`
```sh
#!/bin/bash

# docker_compose_infra 폴더로 이동
cd docker_compose_infra

# docker-compose.yml 파일을 실행
docker-compose up -d
```
Docker Compose를 실행하는 간단한 스크립트다.

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
1분마다 실행되며 MySQL 데이터를 덤프하여 `/backup` 디렉토리에 저장해준다. 🗂️

### 🔹 `docker-compose.yml`
```yaml
services:
  db:
    container_name: mysqldb2
    image: mysql:8.0
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    networks:
      - spring-mysql-net
    volumes:
      - db_data:/var/lib/mysql
    healthcheck:
      test: ['CMD-SHELL', 'mysqladmin ping -h 127.0.0.1 -u root --password=$${MYSQL_ROOT_PASSWORD} || exit 1']
      interval: 10s
      timeout: 2s
      retries: 100
  
  db_backup:
    container_name: db_backup
    build:
      context: ./docker_compose_infra
      dockerfile: Dockerfile.backup
    volumes:
      - ./docker_compose_infra/backup:/backup
      - db_data:/var/lib/mysql
    depends_on:
      db:
        condition: service_healthy
    networks:
      - spring-mysql-net

  app:
    container_name: springbootapp
    build:
      context: ./docker_compose_infra
      dockerfile: Dockerfile.web
    ports:
      - "8080:8080"
    environment:
      MYSQL_HOST: db
      MYSQL_PORT: 3306
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    depends_on:
      db:
        condition: service_healthy
    networks:
      - spring-mysql-net

   app2:
     container_name: springbootapp2
     build:
       context: ./docker_compose_infra
       dockerfile: Dockerfile.web
     ports:
       - "8088:8080"
     environment:
       MYSQL_HOST: db
       MYSQL_PORT: 3306
       MYSQL_DATABASE: ${MYSQL_DATABASE}
       MYSQL_USER: ${MYSQL_USER}
       MYSQL_PASSWORD: ${MYSQL_PASSWORD}
     depends_on:
       db:
         condition: service_healthy
     networks:
       - spring-mysql-net

networks:
  spring-mysql-net:
    driver: bridge

volumes:
  db_data:
```

**db**
- 웹 서버와 통신하는 MySQL 컨테이너 🗄️
- docker volume을 사용하여 상시 백업을 진행한다. 💾

**db_backup**
- 매 분마다 mysql을 덤프 시켜서 백업 해주는 컨테이너 🕐
- Dockerfile.backup를 통해 빌드된다. 🔨
  ```
  Dockerfile.backup

  FROM oraclelinux:9

  # 필요한 패키지 설치
  RUN dnf install -y mysql curl bash cronie && \
      dnf clean all
   
  # 백업 스크립트 복사
  COPY backup.sh /usr/local/bin/backup.sh
  RUN chmod +x /usr/local/bin/backup.sh
   
  # cron 설정
  RUN touch /var/log/cron.log && \
     echo "* * * * * root /usr/local/bin/backup.sh >> /var/log/cron.log 2>&1" >> /etc/crontab
   
  # cron 실행
  CMD ["crond", "-n"]
  ```
  - 🐧**oraclelinux:9을 사용한 이유**
    
    - sh 파일을 cron을 사용해 실행 시키기 위해서는 cron 기능이 포함된 패키지를 설치해줘야 한다.
    - MySQL:8.0은 패키지 설치 툴을 지원하지 않기 때문에 패키지 설치 툴이 있는 oraclelinux:9을 사용했다. 🐳
  
- 컨테이너 내부에서 backup.sh를 crontab을 사용해 매 분마다 실행해준다. ⏳

**app, app2**
- 웹 서버 컨테이너 🌐
- Dockerfile.web을 통해 빌드된다. 🔧
  ```
  Dockerfile.web

  FROM eclipse-temurin:17-jre-alpine
  WORKDIR /app
  COPY --from=0 /app/app.jar app.jar

  ENTRYPOINT ["java", "-XX:+UnlockExperimentalVMOptions", "-XX:+UseContainerSupport", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"]
  ```
  - 🐧**jdk가 아닌 jre를 사용한 이유**
    
    - jdk는 개발 도구가 포함되어 있어 jre에 비해 무겁다. 🏋️‍♂️ 따라서 jre로 빌드를 하면 이미지 용량이 가벼워진다. 💨
      
- 각 웹 서버는 호스트에서 사용하는 포트를 **다르게** 해줘야 **포트 충돌**이 일어나지 않는다. ⚡
  - app -> 8080 🎯
  - app2 -> 8088 🎯
  
모든 컨테이너는 Docker 네트워크 `spring-mysql-net`을 생성한 뒤 해당 네트워크 내부에 배치해준다. 🌐

---

## 🎯 마무리
이 프로젝트를 통해 MySQL과 db_backup서버, Spring Boot 애플리케이션을 효과적으로 컨테이너화하고, 데이터 백업 자동화까지 구현할 수 있었다! 🚀
