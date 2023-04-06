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
Kubernetes는 클러스터의 모든 변경사항을 ETCD에 저장한다.
클러스터를 어떻게 구성하느냐에 따라 ETCD도 다양하게 배포된다.
메뉴얼 설치의 경우 직접 ETCD를 다운로드 받아 설치해주어야 하며, kubeadm을 사용할 경우 자동 배포된다.
만약 쿠버네티스를 HA로 구성할 경우 ETCD도 해당 Master Node의 수 만큼 자동으로 배포된다.

