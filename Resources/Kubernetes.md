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

## 1st Subject