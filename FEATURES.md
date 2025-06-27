# sbkube CLI 주요 기능

sbkube는 Kubernetes 애플리케이션의 배포 및 관리를 자동화하는 CLI 도구입니다. Helm 차트, YAML 매니페스트, Git 저장소 등을 활용하여 애플리케이션의 생명주기를 효율적으로 관리할 수 있도록 돕습니다.

## 1. 핵심 명령어

- **`prepare`**: 애플리케이션 배포에 필요한 외부 소스(Helm 저장소, Git 저장소, Helm 차트 등)를 로컬 환경에 준비합니다.
- **`build`**: 준비된 소스를 기반으로 Helm 차트 등을 빌드하여 배포 가능한 형태로 만듭니다.
- **`template`**: 빌드된 Helm 차트를 YAML 매니페스트로 렌더링합니다. 실제 클러스터에 적용하기 전에 YAML 내용을 확인할 수 있습니다.
- **`deploy`**: Helm 차트, YAML 파일, 또는 사용자 정의 스크립트(exec)를 Kubernetes 클러스터에 적용하여 애플리케이션을 배포합니다.
- **`upgrade`**: 이미 배포된 Helm 릴리스를 새로운 버전으로 업그레이드하거나, 존재하지 않을 경우 새로 설치합니다.
- **`delete`**: 배포된 Helm 릴리스, Kubernetes 리소스(YAML), 또는 사용자 정의 스크립트(uninstall action)를 사용하여 애플리케이션을 클러스터에서 삭제합니다.
- **`validate`**: 설정 파일(config.yaml/toml, sources.yaml/toml)의 유효성을 JSON 스키마 및 Pydantic 데이터 모델을 기반으로 검증합니다.
- **`version`**: sbkube CLI의 현재 버전을 표시합니다.

## 2. 주요 기능 및 개선 사항

### 2.1. 안정적인 명령어 실행 및 에러 처리
- `subprocess.run` 호출을 `sbkube.utils.common.run_command` 유틸리티 함수로 중앙화하여 명령어 실행의 일관성을 확보하고, 타임아웃 처리 및 에러 로깅을 개선했습니다.

### 2.2. 유연한 애플리케이션 정의 및 관리
- `AppInfoScheme`의 `type` 정의를 간소화하고 명확히 하여, `exec`, `install-helm`, `install-action`, `install-kustomize`, `pull-helm`, `pull-helm-oci`, `pull-git`, `pull-http` 등 핵심 배포 및 소스 관리 타입에 집중합니다.
- `install-action` 타입 앱에 대한 삭제 로직을 구현하여, `specs.uninstall.script`에 정의된 스크립트를 통해 애플리케이션을 깔끔하게 제거할 수 있습니다.

### 2.3. Kubernetes OCI (Open Container Initiative) 지원 강화
- `AppPullHelmOciSpec` 모델에 `registry_url` 필드를 추가하여 OCI 레지스트리에서 Helm 차트를 풀링하는 기능을 명확히 지원합니다.

### 2.4. 전역 설정 및 옵션 처리 일관성
- `--namespace`와 같은 전역 CLI 옵션 및 `config.yaml` 내의 전역 설정 처리 로직을 중앙화하여, 각 명령어에서 설정 우선순위를 일관되게 적용하고 코드 중복을 줄였습니다.

### 2.5. 개발 및 유지보수 효율성 증대
- 중복되거나 역할이 모호했던 `sbkube.utils.cli_check.py` 내의 `print_helm_connection_help` 함수를 제거하고, `validate_model.py` 스크립트를 `validate` CLI 명령어로 통합하여 코드 베이스를 간결하게 유지합니다.

## 3. 기타
- `sbkube` 명령어를 인수 없이 실행하면 현재 Kubernetes 설정 정보를 상세하게 보여주어, 클러스터 연결 상태를 쉽게 파악할 수 있습니다.
