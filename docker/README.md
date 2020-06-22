# Docker

도커에 대해 학습한 내용을 정리합니다.

## 자주 쓰는 명령어

```bash
# 현재 PC에서 관리하고 있는 모든 컨테이너들의 목록을 출력합니다.
# 작동이 멈춘 컨테이너도 출력됩니다.
>> docker ps -a

# 해당 컨테이너의 bash shell을 실행합니다.
>> docker exec -it ${컨테이너 ID | 컨테이너 별명} /bin/bash

# local storage에 pull된 이미지 목록을 출력합니다.
>> docker images

# 해당 이미지를 컨테이너로 띄웁니다. (-d : 백그라운드, -p : 포트 지정)
>> docker run -d -p [호스트포트:컨테이너포트] [이미지명]
>> ex) docker run -d -p 1234:6379 redis
```

## docker-compose
컨테이너 조합 및 복잡한 설정이 추가되면 명령어만으로 원하는 설정을 하기 어렵습니다.
이 떄, docker-compose 를 이용하여 가독성과 편리성을 높일 수 있습니다.

```bash
version: "3.7" # 파일 규격 버전
services: # 이 항목 밑에 실행하려는 컨테이너 들을 정의
  db: # 서비스 명
    image: mysql:5.7 # 사용할 이미지
    restart: always
    container_name: mysql-test # 컨테이너 이름 설정
    ports:
      - "3307:3306" # 접근 포트 설정 (컨테이너 외부:컨테이너 내부)
    environment: # -e 옵션
      MYSQL_DATABASE: test-db
      MYSQL_ROOT_PASSWORD: password  # MYSQL 패스워드 설정 옵션

    command: # 명령어 실행
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
    volumes:
      - /Users/Shared/data/mysql-test:/var/lib/mysql # -v 옵션 (다렉토리 마운트 설정)
```
docker-compose.yaml
위와 같이 docker-compose.yaml을 작성합니다.
```bash
# docker-compose.yaml 파일에 설정된 대로 컨테이너를 생성합니다.
>> docker-compose up
```
