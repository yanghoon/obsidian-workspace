#helm

# kubectl

## kubeconfig 관리

kubectl은 클러스터 접속 정보(클러스터 주소, 사용자 인증, 네임스페이스 등)를 포함하는 kubeconfig 파일을 참조하여 k8s 클러스터와 통신합니다.

kubeconfig 설정 방법 :

1. 기본적으로 kubectl은 사용자의 홈 디렉터리 내의 `$HOME/.kube/config` 파일을 참조합니다.
2. `KUBECONFIG` 환경 변수를 설정하면 기본 위치 대신 해당 경로상의 kubeconfig 파일을 사용합니다.
	- `export KUBECONFIG=/path/to/kubeconfig` (Linux/macOS)
	- Windows에서는 `set KUBECONFIG=C:\path\to\kubeconfig`로 설정 가능
3. `--kubeconfig` 플래그를 사용하면 환경 변수 및 기본 위치보다 우선해 지정한 파일을 사용합니다.

설정 우선순위:

  1. `--kubeconfig` 플래그(명령어 실행 시 지정)
  2. `KUBECONFIG` 환경 변수
  3. 기본 경로 `$HOME/.kube/config`

다중 kubeconfig 병합:

1. `KUBECONFIG`에 여러 파일 경로를 콜론(:) 또는 세미콜론(Windows)으로 연결해 둘 이상 파일을 사용할 수 있으며, 이 경우 파일들이 병합되어 컨텍스트, 클러스터 등이 처리됩니다.
   여러 kubeconfig 파일 경로를 `KUBECONFIG` 환경변수에 연결하여 멀티 클러스터 접근 설정 가능(예: `export KUBECONFIG=~/.kube/config:~/.kube/config2`)
2. `kubectl config view --merge --flatten > merged_config` 명령어로 여러 kubeconfig 파일을 하나로 병합 가능
   병합한 kubeconfig 파일을 `KUBECONFIG` 환경변수에 설정해 사용하거나 기본 config 파일로 교체 가능

예제:

```bash
# KUBECONFIG에 여러 config 지정 (Linux/macOS)
export KUBECONFIG=~/.kube/config:~/.kube/config2
```

```bash
# 윈도우 PowerShell
set KUBECONFIG=C:\Users\user\.kube\config;C:\Users\user\.kube\config2
```

```bash
# 여러 config 병합 후 새 파일로 생성
KUBECONFIG=~/.kube/config:~/.kube/config2 kubectl config view --merge --flatten > ~/.kube/merged_config

# 병합 config 사용 설정
export KUBECONFIG=~/.kube/merged_config

# 컨텍스트 전환
kubectl config use-context <context-name>
```

# Helm

# k3d

## k3d 로컬 클러스터 만들기

k3d는 Docker 기반으로 작동하는 경량 Kubernetes 클러스터 생성 도구로, k3s를 실행하여 로컬 개발 및 테스트 환경을 쉽게 구성할 수 있습니다.

k3d를 사용해서 로컬에 Kubernetes 클러스터를 만들 때, yaml 설정 파일로 노드 수, 역할, 자원 제한을 지정할 수 있습니다.

아래는 이러한 설정을 위한 k3d 클러스터 생성용 yaml 예시입니다.

- Node 구성
	- Master Node 1개
	- Worker Node 2개
- 각 노드는 1 core, 2GB 메모리 제한

```yaml
apiVersion: k3d.io/v1alpha4
kind: Simple
metadata:
  name: mycluster
servers: 1   # Master 노드 1개
agents: 2    # worker 노드 2개
options:
  server:
    k3s:
      extraArgs:
	    # 필요한 경우 불필요한 기본 컴포넌트 비활성화 optional
        - --no-deploy=servicelb,traefik
nodes:
  - role: server
    image: rancher/k3s:v1.27.4-k3s1
    name: k3d-mycluster-server-0
    # Master 노드 스케줄링 차단: taint 설정으로 직접 적용 필요 (아래 참고)
  - role: agent
    image: rancher/k3s:v1.27.4-k3s1
    name: k3d-mycluster-agent-0
    cpu: 1          # 1 core 제한
    memory: 2gb     # 2GB 메모리 제한
  - role: agent
    image: rancher/k3s:v1.27.4-k3s1
    name: k3d-mycluster-agent-1
    cpu: 1
    memory: 2gb
```

Master 노드 스케줄링 차단:

- k3d에서 직접 마스터 노드에 taint를 yaml에서 지정하는 기능은 지원하지 않아서, 클러스터 생성 후 kubectl로 taint를 설정해야 합니다.

```bash
kubectl taint nodes k3d-mycluster-server-0 \
	node-role.kubernetes.io/master=:NoSchedule
```

클러스터 생성:

```bash
k3d cluster create --config cluster.yaml
```

이렇게 하면 요청하신 조건에 맞춘 k3d 로컬 클러스터가 만들어집니다.