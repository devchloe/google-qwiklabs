# NodeJS Application을 Kubernetes Cluster에 배포하기 

## 용어 이해 
- Kubernetes
- Kubernetes Cluster 
- Kubernetes 아키텍처 
- Kubernetes 사용 목적
- kubectl proxy
- kubectl run 
- kubectl get 
- kubectl get events 

## 실습 단계 
- create application
- build docker container image (package application in a docker container)
- push the image to registry
- create kubernetes cluster
- create kubernetes pod/resources definition file
- create kubernetes pod
- create service/load-balancer


## Node App 만들기 

## Docker container image 만들기 (package application)

## Docker registry에 푸시하기 
```bash
$ gcloud docker -- push gcr.io/[project-name]/[image]:[tag]
```
## Kubernetes Cluster 만들기 
```bash
$ gcloud config set project PROJECT_ID
$ gcloud container clusters create hello-world --num-nodes 2 --machine-type n1-standard-1 --zone us-central1-a
```

## Pod 생성 

pod = container들의 그룹 
```bash
$ kubectl run hello-node --image=gcr.io/[project-id]/hello-node:v1 --port=8080
```

`kubectl run` deployment를 생성, deployment는 pod을 생성함과 동시에 pod을 몇개 생성할지 정의할 수 있음, 추천하는 방식

`kubectl get deployments`

`kubectl get pods`

`kubectl cluster-info`

`kubectl config view`

`kubectl get events`

`kubectl logs <pod-name>`

## 외부 트래픽 허용하기

pod에 할당된 ip는 cluster 내부에서만 접근가능하다. 

hello-node container를 kubernetes cluster 밖에서 접근하려면 kubernetes service를 이용해서 pod을 노출시켜야 한다.

`kubectl expose --type="LoadBalancer"` 를 이용해서 public internet에 pod을 노출시킬 수 있다. 이렇게 노출시키면 외부에서 접근가능한 IP를 할당한다.

kubectl expose deployment hello-node --type="LoadBalancer"

즉 pod을 노출시키는 것이 아니라 deployment를 노출한다. 

service가 받은 트래픽은 deployment가 관리하는 pod들에게 분배된다.

Kubernetes master는 load balancer를 생성하고 정해진 forwarding rule, worker node들, firewall rules에 따라 cluster 밖에서 service에 접근할 수 있다. 

kubectl get services

service에 할당된 external-ip 확인 

## pod replicas 변경하기 

```bash
$ kubectl scale deployment hello-node --replicas=4
```

replication controller에게 이 설정값이 전달되어서 hello-node deployment가 관리하는 pod의 개수가 늘어난다 

```bash
$ kubectl get deployment
$ kubectl get pods
```

이것은 선언적 접근법이다. 새로운 인스턴스를 시작하고 중지하는게 아니라 몇개의 인스턴스가 항상 실행되고 있어야 하는지 선언해놓는 것이며 예기치 못하게 셧다운 되었을 때에는 Kubernetes masterdml replication controller가 이를 감시하고 있다가 개수를 항상 일정하게 맞춰준다. 개수가 이상하게 늘었을 때에도 조정한다.

Kubernetes는 reconciliation loop를 갖고 있다.


## 업그레이드 버전을 kubernetes service에 Roll out 하기 
 새로운 버전을 릴리즈하거나 버그 픽스를 운영환경에 반영할 때 사용자에게 영향을 주지 않고 새로운 버전을 배포할 수 있다. 

 이때 처음과 동일하게 새로운 docker container image를 만들어 registry에 푸시한다.
이것을 현재 실행중인 container를 새로운 container image로 바꾸려면 deployment를 수정하면 된다. 

```bash
$ kubectl edit deployment hello-node
$ kubectl get deployments
```

위에서 image 부분을 새로운 이미지로 변경하고 저장하면 pod을 새로운 이미지로 변경할 수 있다. 

기본적으로 pod이 셧다운 될 때 RollingUpdate 전략을 취하기 때문에 사용자에게는 전혀 영향을 주기 않는다. 왜냐하면 2개이상의 replicas를 운영하고 있기 때문에 셧다운된 pod에는 traffic을 전달하지 않을 것이기 때문이다. 그리고 새로운 이미지를 배포하는 경우에는 하나가 셧다운되고 정상적으로 실행상태가 될 때까지 기다렸다가 다음 pod을 셧다운 시키기 때문이다. 


## Kubernetes dashboard에 접근하기 

```bash

kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value account)

kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml

# Change type: ClusterIP to type: NodePort. 
kubectl -n kube-system edit service kubernetes-dashboard

kubectl -n kube-system describe $(kubectl -n kube-system \
get secret -n kube-system -o name | grep namespace) | grep token:

kubectl proxy --port 8081









