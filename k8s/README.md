# Kubernetes (k8s)

쿠버네티스에 대해 스터디한 내용을 정리합니다.
(실습은 주로 GCP에서 진행했습니다.)

## 자주쓰는 명령어

```bash
# 클러스터에 올라와 있는 모든 자원(pod, replica set, deployment 등)들의 목록을 보여줍니다.
>> kubectl get all
```

```bash
# yaml 파일에 정의된 대로 자원을 생성합니다.
>> kubectl create -f ${yaml 파일경로}
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

## 네임스페이스

```bash
# namespace를 생성하는 yaml 파일 내용을 보여줍니다.
>> kubectl create ns peter --dry-run=client -o yaml

# namespace를 생성하는 yaml 파일을 생성합니다.
>> kubectl create ns peter --dry-run=client -o yaml > peter-ns.yaml

# 'peter' 네임스페이스에 nginx deployment를 생성합니다.
>> kubectl create deploy nginx --image nginx -n peter

# 'peter' 네임스페이스에 생성된 자원을 보고싶으면 아래와 같이 명령어를 실행합니다.
>> kubectl get all -n peter

# 'peter' 네임스페이스를 삭제합니다. => 'peter' 네임스페이스에 있었던 모든 자원이 삭제됩니다.
>> kubectl delete ns peter

# dns가 어느 네임스페이스에 속해 있는지 알아내기!
>> kubectl get pod --all-namespaces | grep dns
```

## 서비스

#### 1) sessionAffinity
* ClusterIP : 내부 자원끼리 서로 통신하기 위한 IP
* sessionAffinity : 사용자가 서비스를 통해 특정 pod에 접속했을 때, 그 session을 유지해줍니다. (로그인 서비스 등을 제공하기 위해서 이 설정을 활용할 수 있습니다.)
```bash
# 서비스의 sessionAffinity 테스트를 위해 yaml 파일을 하나 생성합니다.
>> kubectl create deploy http-go --image=gasbugs/http-go --dry-run=client -o yaml > http-go-deploy.yaml
```

```bash
# 파일을 편집하여 Service 부분을 아래와 같이 추가합니다.

apiVersion: v1
kind: Service
metadata:
  name: http-go-svc
spec:
  selector:
    app: http-go
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: http-go
  name: http-go
spec:
  replicas: 1
  selector:
    matchLabels:
      app: http-go
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: http-go
    spec:
      containers:
      - image: gasbugs/http-go
        name: http-go
        resources: {}
status: {}
```

```bash
# 서비스와 Deployment가 정의된 yaml 파일로 서비스를 띄웁니다.
>> kubectl create -f http-go-deploy.yaml

# 서비스의 sessionAffinity를 ClientIP로 수정하면 로드벨런싱 되지 않고 하나의 pod에 계속 접속됩니다.
>> kubectl edit svc http-go-svc

# 간단하게 pod 하나 띄워서 접속 테스트를 해볼 수 있습니다.
>> kubectl run -it --rm --image=busybox bash
/# wget -O- -q 10.36.15.109:80
=> sessionAffinity를 ClientIP로 설정했을 때는 단일 pod에 붙고, None으로 설정 한 경우에는 여러 pod으로 로드벨런싱 됨을 확인할 수 있습니다.
```

#### 2) NodePort

위 예제에서 만든 서비스에 부가적으로 NodePort 서비스를 만들면 외부에서 접속할 수 있습니다.
아래와 같이 NodePort 타입의 서비스 yaml 파일을 작성합니다. (nodePort로 설정할 수 있는 값 범위 : 30000~32767)

```bash
apiVersion: v1
kind: Service
metadata:
  name: http-go-np
spec:
  type: NodePort
  selector:
    app: http-go
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30001
```
http-go-np.yaml

```bash
# 위에서 작성한 서비스를 띄웁니다.
>> kubectl create -f http-go-np.yaml

# 외부 접속을 위해서는 방화벽을 열어주어야 합니다. (여기서는 30001번 포트로 실습해봅니다.)
>> gcloud compute firewall-rules create http-go-svc-rule --allow=tcp:30001

# 현재 프로젝트에서 사용되고 있는 방화벽 정책 목록은 다음 명령어로 확인할 수 있습니다.
>> gcloud compute firewall-rules list

# node의 EXTERNAL-IP를 알아내기 위해 아래 명령어를 입력합니다.
>> kubectl get node -o wide

# 위에서 알아낸 EXTERNAL-IP로 아래와 같이 curl로 어플리케이션에 접근할 수 있습니다.
>> curl {EXTERNAL-IP}:30001
```
