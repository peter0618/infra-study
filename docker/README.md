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
