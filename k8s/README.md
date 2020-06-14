# Kubernetes (k8s)

쿠버네티스에 대해 스터디한 내용을 정리합니다.

## 자주쓰는 명령어

```bash
# 클러스터에 올라와 있는 모든 자원(pod, replica set, deployment 등)들의 목록을 보여줍니다.
>> kubectl get all
```

```bash
# yaml 파일에 정의된 대로 자원을 생성합니다.
>> kubectl create -f ${yaml 파일}
```

```bash
# 클러스터에 올라와 있는 모든 자원 삭제
>> kubectl delete all --all
```


## 롤링업데이트

* maxSurge
롤링 업데이트 시, 최대 몇개의 pod을 추가할 것인가.
예) 4개인 경우 25%이면 1개가 추가됨.

* maxUnavailable
롤링 업데이트 시, 동작하지 않는 pod의 갯수 설정
예) 4개인 경우 25%이면 1개로 설정됨. (4-1 = 3개가 운영됨)

* 업데이트 시키는 방법 2가지

```bash
# (방법1)
# "set image" 명령어로 deploy 대상 컨테이너의 이미지를 변경하면 롤링업데이트가 진행됩니다.
# 이때, 맨 뒤에 --record=true 옵션을 주면 revision history에 저장됩니다.
>> kubectl set image deploy http-go http-go=gasbugs/http-go:v2 --record=true

# history는 아래와 같이 명령어를 실행하면 볼 수 있습니다.
>> kubectl rollout history deploy http-go
```

```bash
# (방법2)
# "edit" 명령어를 실행하여 yaml 파일을 직접 변경시키면 롤릴업데이트가 진행욉니다.
>> kubectl edit deploy http-go --record=true
```

* 롤백
```bash
# 아래와 같이 명령어를 실행하면 이전버젼으로 롤백됩니다.
>> kubectl rollout undo deploy http-go

# 원하는 revision을 명시하면, 해당 revision으로 롤백합니다.
>> kubectl rollout undo deploy http-go --to-revision=1
```
