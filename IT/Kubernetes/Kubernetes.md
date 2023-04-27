# Kubernetes
## Kubernetes에 대한 일반적인 설명
쿠버는 계속 사용량이 증가하고 있다.
쿠버네티스의 목표 : 응용프로그램을 컨테이너 형식으로 자동화 시켜 호스팅하는 것.
일반적으로 쿠버네티스는 컨테이너선에 비유하며, Master node, Worker Node로 구분한다.
Master Node는 ETCD라는 Key-value 데이터 저장소를 사용해 쿠버네티스의 동작 정보를 저장한다.
Kube-scheduler는 컨테이너를 관리하는 영역으로, 노드의 용량 혹은 다은 정책이나 제약조건등 리소스 요구사항 등을 관리하며, 컨테이너를 배포한다.
kube-apiserver는 모든 쿠버네티스의 주요 관리 요소를 관리하는 역할을 한다.
worker node에는 컨테이너 런타임 엔진이 설치되어야 하며, 대표적으로는 docker가 있다.
kubelet은 각 노드의 동작을 관장하는 핵심 프로세스이다. kube-apiserver는 각 노드의 kubelet으로 부터 노드의 정보를 가져와 관리한다.
kube-proxy 서비스는 작업 노드에 필요한 규칙을 만들고, 각각의 컨테이너를 연결하는 역할을 한다.

노드|구성요소
-|-
Master node|ETCD,kube-apiserver, Kube Controller Manager, kube-scheduler
Worker Nodes|kubelet,kube-proxy,container runtime engine

## Docker와 Containerd의 차이
### Docker
docker는 conatiner를 매우 쉽게 조작할 수 있어 거의 표준으로 자리잡았다.
Kubernetes는 초기버전에서는 Docker만을 컨테이너 런타임으로 사용하였기 때문에, 굉장히 밀접한 관계를 가지고 있었다.
하지만 rkt과 같은 다른 컨테이너 런타임을 사용하고자 하는 수요가 증가함에 따라 쿠버네티스는 CRI(Container Runtime Interface)를 소개하게 되었다.
쿠버네티스의 CRI는 OCI(Open Container Initiative) 표준을 준수하는 모든 컨테이너 런타임을 지원할 수 있게 해주었다.
OCI는 imagespec과 runtimespec으로 이루어져 있으며, imagespec은 이미지를 어떻게 빌드하는지에 대한 기준을 정의한다.
docker는 CRI를 지원하지 않기 때문에(CRI의 개념이 등장하기 훨씬 이전부터 Docker가 존재했기 때문) 쿠버네티스는 dockershim을 도입하였다.
dockershim은 CRI 밖에서 docker를 계속 지원하도록 하는 임시 방편이었다.
docker 내부에는 CLI, API, BUILD, VOLUMES, AUTH, SECURITY, containerd와 같은 많은 구성요소들이 있는데, 이중 containerd는 CRI와 호환이 가능하다.
dockershim은 v1.24버전부터 삭제되었고, docker는 공식적으로 쿠버네티스 컨테이너 런타임에서 제외되었나, docker에서 만들어진 image는 OCI를 따르고 있기 때문에 여전히 사용이 가능하다.

containerd는 현재 docker에서 독립된 프로젝트이며, CNCF의 회원자격을 가지고 있다.
이제는 docker를 설치하지 않아도 컨테이너를 containerd를 사용하여 컨테이너를 실행할 수 있는 것이다.
하지만 docker가 설치되지 않은 환경에서 containerd만을 사용하여 컨테이너의 실행이 가능할까?
그렇지 않다. docker CLI를 통한 실행처럼 이를 실행시켜줄 수 있는 툴이 필요하다.
containerd를 설치하면 ctr이라는 툴이 함께 설치되기는 하지만, 사용자 친화적이지 않고, 기능도 제한적이다.
ctr에 대한 더 나은 대안은 nerdctl, podman 등으로, docker와 아주 유사하며, 대부분의 명령어가 호환된다.
crictl이라는 툴도 있으며, 이 툴은 여러가지 런타임에 대한 지원을 한다. 하지만 디버깅툴에 가까워 컨테이너 제작에는 이상적이지는 않다.
crictl은 pod도 직접적으로 확인이 가능하다. 하지만 kubelet과 정보를 주고받지 않기 때문에 crictl을 사용해 pod를 만들경우 kubelet이 자동으로 pod를 삭제한다는 문제가 있다.
이제 대부분의 경우에는 crictl을 사용하는 것을 추천한다고 한다.

## ETCD
ETCD는 신뢰할 수 있는 분산 키-벨류 저장소이다.
신속하고 안전하며 간단하다.
### Install
1. Download Binaries
   crul -L https://github.com/etcd-io/etcd/releases/download/v3.4.24/etcd-v3.4.24-linux-amd64.tar.gz -o etcd-v3.4.24-linux-amd64.tar.gz
2. Extract
   tar -zxvf etcd-v3.4.24-linux-amd64.tar.gz
3. Run ETCD Service
   ./etcd

service를 시작하면 ETCD는 2379 Port로 서비스 리슨이 시작된다.
etcdctl을 통해서 ETCD를 사용할 수 있다.
```bash
./etcdctl set key1 value1 ## API 버전 2점대
./etcdctl put key1 value1 ## API 버전 3점대
```
검색에는 get을 사용한다.
```bash
./etcdctl get key1
```
ETCD는 버전에 따라 명령어가 상이함으로 버전에 주의하여야 한다.
v2.0 부터는 초당 1만건의 쓰기 기능을 지원하였으며, v3.0부터는 많은 기능개선이 되었기 때문이다.
주의할 것은 v2.0과 v3.0 사이의 차이가 크다는 것으로, etcdctl을 실행할 때 다음과 같이 버전의 확인이 반드시 필요하다.
```bash
./etcdctl --version
> etcdctl version: 3.4.24
> API version: 2
```
여기서 API version이 중요한데, 2로 되어있는 경우 v2.0을, 3.n으로 되어있는 경우 v3.n을 API버전으로 인식한다는 것이다.
기본적으로 etcdctl의 API version은 2로 설정되어 있으며, 만약 3버전대를 사용하고 싶다면 다음과 같이 설정해야 한다.
```bash
ETCDCTL_API=3 ./etcdctl version
## 또는
export ETCDCTL_API=3
./etcdctl version
```

### K8S에서의 ETCD의 역할
Kubernetes는 클러스터의 모든 변경사항을 ETCD에 저장한다. (API server가 저장)
클러스터를 어떻게 구성하느냐에 따라 ETCD도 다양하게 배포된다.
메뉴얼 설치의 경우 직접 ETCD를 다운로드 받아 설치해주어야 하며, kubeadm을 사용할 경우 자동 배포된다.
만약 쿠버네티스를 HA로 구성할 경우 ETCD도 해당 Master Node의 수 만큼 자동으로 배포된다.
기본적으로 쿠버네티스가 ETCD에 접속하고자 한다면 2379 포트를 쿠버네티스 API server가 접근할 수 있게 열어두어야 한다.

### Kube-apiserver
kube-apiserver는 사용자로부터 직접적으로 커맨드를 입력받고, 그 정보를 바탕으로 모든 작업을 진행하는 컨트롤타워이다.
kube-apiserver는 master node에 존재하며, etcd, scheduler, kubelet등을 관장한다.
kube-apiserver가 담당하는 부분
1. Authenticate User
2. Validate Request
3. Retrieve data
4. Update ETCD
5. Scheduler
6. Kubelet

kubectl get pods -n kube-system
cat /etc/kuberntes/manifests/kube-apiserver.yaml

### Kube-Controller-Manager
master node 안에 존재하는 controller-manager는 클러스터의 전체적인 상태를 모니터링하고, 이벤트가 발생하면 그에 상응하는 조치를 취하는 역할을 한다.
Kube-Contorller-Manager를 설치하면, 나머지 다른 Controller들도 함께 설치된다.

Controller-Manager의 종류에는 여러가지가 있다. (아래는 몇몇 예시이다)
1. Node-Controller : Kube-API-server를 통해 Node의 상태를 확인하고, Node안의 프로그램들이 정상적으로 동작할 수 있게 유지하는 역할을 한다. apiserver를 통해 5초마다 node의 상태를 점검한다. node가 하나 사용불가 상태가 되면, node-controller는 해당 node를 약 40초간 모니터링하고 난 이후에도 상태가 사용불가이면 unreachable 처리를 한다. 그리고 5분 후에는 해당 node를 클러스터에서 제외시키고, 해당 node가 가지고 있던 pod들을 사용가능한 node로 옮긴다. (Node Monitor Period = 5s / Node Monitor Grace Period = 40s / Pod Eviction Timeout = 5m)
2. Replication-Controller : replicaset의 상태를 모니터링하는 역할을 하며, set안에 포함된 pod의 갯수를 항상 동일하게 유지하는 역할을 한다.
이외
- Deployment-Controller
- Namespace-Controller
- Endpoint-Controller
- CronJob
- Job-Controller
- PV-Protection-Controller
- Service-Account-Controller
- Stateful-Set
- Replicaset
- PV-Binder-Controller

### Kube-Scheduler
Kube-Scheduler는 어떤 Pod가 어떤 Node에 배치될지만을 결정하는 역할을 한다. Pod를 만드는 것은 Kubelet이 담당한다.
각각의 Pod는 요구하는 리소스가 서로 다르기 때문에 Kube-Scheduler는 각각의 Pod와 Node의 정보를 보고, 각 Pod가 배치될 최적의 Node를 찾는다.
먼저 Node를 필터링하고, Node들에 대한 Rank를 산정한 이후 Pod를 배포한다.(Rank를 산정하는 방식은 여러가지가 있으며, 직접 만들수도 있다.)

### Kubelet
Kubelet은 모든 노드의 활동을 관리하는 관리주체이다. scheduler에 의해 worker node에 pod를 만들라는 요청을 받으면, 해당 node의 kubelet은 pod를 생성한다.
Kubelet은 worker node에서 일어나는 일들을 모니터링하며 apiserver에 현황을 보고하는 역할을 수행한다.
kubeadm을 사용해서 설치를 진행하더라도, kubelet은 자동으로 설치되지 않기 때문에 각각의 worker node에 수동으로 kubelet을 설치해주어야 한다.

### Kube-proxy
Kubernetes 클러스터 안에서는 모든 Pod는 다른 Pod에 연결될 수 있다.
이는 Pod networking 솔루션을 클러스터에 배포함으로써 가능해지며, POD Network는 kubernetes cluster 내부의 모든 node와 pod에 걸친 가상네트워크이다.
이러한 Pod network는 메모리 상에만 존재하는 service라는 object를 사용해 구현되는데, 모든 cluster에 걸쳐 배포될 수는 없다. 그리고 이를 보완하는 것이 kube-porxy이다.
kube-proxy는 각각의 node에 배포되어 service간의 통신을 중계한다.
kube-proxy의 중계방식은 iptables 규칙을 사용하는 것이 있다.
kube-proxy는 각각의 node 마다 데몬셋으로 배포된다.

### Pod
Pod는 kuberntes가 만들 수 있는 가장 작은 object이다. 대부분의 경우에 pod는 하나의 container를 가지고있다.
pod안에는 container가 여러개 존재할 수 있다는 의미이다. 이때문에 pod를 사용하면 container를 관리하는 측면에서 여러 container를 번들처럼 관리할 수 있다는 장점이 있다.
하지만 여러개의 container를 하나의 pod에 배치하는 것은 극히 드문 일이다.
pod를 만들 이미지는 온라인의 repository나 private repository에서 끌어올 수 있다.
kubectl run [podname] --image [image name] 으로 pod를 만들때 사용할 image를 지정할 수 있다.
YAML을 사용해서도 pod를 만들 수 있다.

```yaml
# YAML파일을 통한 pod의 생성에서 root level의 요소는 다음과 같다
# 1. apiVersion : 만들고자 하는 object를 만들기 위해 사용하는 api의 버전을 말한다. 만들고자 하는 object가 어떤것인지에 따라 버전이 달라진다.
# 2. kind : 만들고자 하는 object가 무엇인지 지정하는 부분.
# 3. metadata : Dictionary 형태로 작성되는 모든 메타데이터를 지정하는 영역이다. 메타 아레의 요소들 앞에 붙는 공백의 갯수는 상관이 없지만, 들여쓰기는 반드시 맞춰주어야 한다.
# 4. spec : 생성하려는 object에 따라 그 특성이 서로 다르기 때문에 object마다 spec영역에서 사용 가능한 속성들이 다르다. spec영역도 Dictionary 형식으로 작성한다.

#sample.yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:                  # labels에서는 key: value 형태로 라벨을 붙일 수 있는데, 이는 pod들을 필터링하는 용도로 사용할 수 있다.
     app: myapp            # 다른 속성들과는 다르게 label은 app, type 등 원해는대로 붙여 필터링이 가능하다.
spec:

```
다 작성된 YAML파일을 가지고 pod를 생성하기 위해서는 아래의 명령어를 사용할 수 있다.
```bash
kubectl create -f sample.yaml
```

### Replication Controller
Replication Controller는 Replica Set으로 대체되었다.
ReplicationController를 생성하기 위해 yaml파일을 작성한다면 다음과 같이 작성할 수 있다.

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:
  template:    ##템플릿의 아래쪽으로 기존의 pod의 항목들을 위치시킬 수 있다. 템플릿은 결국 어떤 pod를 생성할지를 지정하는 부분이다.
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx
  replicas: 3   ## replica는 템플릿을 기준으로 만들 pod가 몇개인지 설정하는 부분이다.

```
### Replica Set
Replica Set은 Replication Controller의 대체제로, 거의 비슷한 형태를 띄고있다.
하지만 apiVersion은 apps/v1으로 조금 다르다.
또한 ReplicaSet은 selector 부분에 추가로 설정이 필요하다.
ReplicaSet의 가장 큰 차이점은 기존에 생성된 Pod도 함께 모니터링 할 수 있다는 것이다. 만약 selector를 통해 지정한 이미 만들어진 Pod가 종료된다면 ReplicaSet이 설정에 맞춰 새로 만들어주는 것이다.


```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
  spec:
    containers:
    - name: nginx-container
      image: nginx
  replica: 3
  selector:
    matchLabels:
      type: front-end
```

### Service
Pod그룹간의 통신을 중계하는 오브젝트이다.
서비스에는 여러가지 타입이 있다.
- NodePort : 클러스터의 노드 안의 포트에 접근할 수 있게 해주는 서비스. NodePort는 30000 ~ 32767까지의 범위에서만 지정 가능하다. 클러스터 전체에 걸쳐 설정될 수 있다.
- ClusterIP : 클러스터 내부에 가상 IP를 생성하여 다른 서비스와 소통할 수 있게 해주는 서비스. Kubernetes의 기본 서비스타입은 ClusterIP이다.
- LoadBalancer : 부하분산용 서비스

#### 01. NodePort
NodePort의 경우에는 신경써야 할 Port가 총 3개이다. NodePort와 Port, TargetPort가 그것으로, NodePort는 워커노드의 Port이며, Port는 서비스의 Port이다. TargetPort는 Pod의 Port이다.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
    - targetPort: 80 ## 제공하지 않으면 service의 port와 동일하다고 간주
      port: 80
      nodePort: 30008 ## 제공하지 않으면 30000 ~ 32767 범위에서 자동으로 할당
  selector: ## Pod의 Label을 그대로 입력해주면 서비스가 해당 Label에 해당하는 Pod를 end-point로 삼게끔 생성된다.
    app: myapp
    type: front-end
```
#### 02. ClusterIP
Pod의 IP는 Pod가 생성되고 사라질때마다 유동적으로 할당되기 때문에 ClusterIP를 통하여 Pod에 접근하는 IP를 고정시킬 수 있다.
ClusterIP 또한 selector를 통해 Pod와 맵핑될 수 있다.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  type: ClusterIP
  ports:
    - targetPort: 80
      port: 80
  selector:
    app: myapp
    type: front-end
```

#### 03. LoadBalancer
LoadBalancer는 여러 Pod에 연결하여 클라이언트의 요청을 연결된 여러 Pod들에 고르게 분산해 주는 역할을 한다.
쿠버네티스에서는 Cloud 프로바이더의 자체 Loadbalancer를 사용할 수 있도록 지원하고 있다. (GCP, AWS, Asure 에서는 확실하게 지원된다.)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  ports:
    - targetPort: 80
      port: 80
```

### Deployment
Pod에 대한 배포, 이미지 관리, 롤링 업데이트, 롤백, 일시정지, 재시작 등을 할 수 있게 해주는 오브젝트이다.
각각의 Pod를 배포하는 것이 아닌 Deployment를 통한 통합 배포가 가능하다.
```yaml
apiVersion: apps/v1
kind: Deployment  ## Deployment는 replicaset과 YAML파일이 거의 같다.
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
  spec:
    containers:
    - name: nginx-container
      image: nginx
  replica: 3
  selector:
    matchLabels:
      type: front-end
```

### Namespaces
Namespace는 Kubernetes 오브젝트들의 격리를 제공한다. 클러스터 내부의 오브젝트들을 감싸는 큰 단위의 격리체계로 생각하면 된다.
또한 리소스에 대한 제한도 가능하다. 각각의 Namespace는 리소스의 할당이 가능하며, 할당된 이상의 리소스는 사용하지 않는다.
Namespace는 지정하지 않은 경우에는 기본적으로 생성되는 Default를 사용한다.
kube-system Namespace 안에는 Kubernetes의 동작에 사용되는 필수 오브젝트들이 격리되어있다.
kube-public에는 클러스터 공통으로 사용하는 오브젝트들이 격리되어 있다.

```yaml
## namespce-dev.yaml 예시
apiVersion: v1
kind: Namespace
metadata:
  name: dev

## namespace의 설정 예시(Pod)
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  namespace: dev  ## metadata의 하위에 namespace를 지정할 수 있다.
  labels:
    app: myapp
    type: front-end
spec:
  containers:
  - name: nginx-container
    image: nginx
```

Namespace에서 사용하는 리소스를 제한하려면 Resource Quota를 생성해야한다.
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
sepc:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```
### DNS
[servicename].[namespace].svc.cluster.local

### 명령형(Imperative) VS 선언형(Declarative)
인프라를 코드로 관리하는 것에는 여러가지 접근방법이 있다.
명령형은 어떻게 움직이라고 단계별 명령을 진행하는 것이고, 선언형은 YAML과 같은 파일에 정의해 둔 대로 명령을 수행하는 것을 말한다.
kubectl을 사용하는 것은 명령형으로, 여러가지 명령을 커맨드로 수행할 수 있지만 제한적이다.
YAML과 같은 파일에 지정하는 것은 선언형으로, Git과 같은곳에 저장하여 관리할 수 있으며, 변경하거나 검토하기에도 용이하다.
시험등과 같은 상황에서는 명령형으로 오브젝트를 생성하는 것이 더 시간절약에 도움이 된다.


## Kubectl 명령어
```bash
#pod 생성
kubectl run [podname] --image=[imagename]
#pod 확인
kubectl descirbe pod [podname] -n [namespace]
kubectl get pods -n [namespace]
kubectl get pods --all-namespaces
kubectl get pods -A
kubectl get pods -n [namespace] -o wide
#pod 삭제
kubectl delete pod [podname] -n [namespace] -o wide
# yaml 생성
kubectl run [podname] --image=[imagename] --dry-run=client -o yaml > [~~~.yaml]
  >>> kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml
# yaml을 통한 오브젝트 생성
kubectl create -f [~~~.yaml]
kubectl apply -f [~~~.yaml]
kubectl apply -f [~~~/dir] # 디렉토리 내부에 모든 파일들을 한번에 읽고 필요시 object 생성까지 진행
kubectl replace -f [~~~.yaml]
kubectl scale --replicas=[n] -f [~~~.yaml]
kubectl scale --replicas=[n] replicaset [replicasetname]
# 오브젝트 수정
kubectl edit [object] [objectname]
# 오브젝트 확인
kubectl get all
# kubectl에 대한 변수값 설정
kubectl config set-context $(kubectl config current-context) --namespace=[namespace]
```
