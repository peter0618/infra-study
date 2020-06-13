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
