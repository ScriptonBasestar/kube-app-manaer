# TODO 리스트

## 테스트 케이스 수정

[x] prepare 테스트에서 sources.yaml 파일 누락 문제 해결  
- 관련 ISSUE: prepare 테스트 실패 - sources.yaml 파일 누락  
- 위치: `tests/test_prepare.py`

[x] 빌드 테스트에서 지원되지 않는 타입 사용 문제 수정  
- 관련 ISSUE: 지원되지 않는 앱 타입으로 인한 빌드 테스트 실패  
- 위치: `tests/test_build.py`

## 앱 타입 지원 확장

[x] config_model에 copy-app, install-yaml 타입 추가  
- 관련 ISSUE: 지원되지 않는 앱 타입으로 인한 빌드 테스트 실패  
- 위치: `sbkube/models/config_model.py`

[x] build 명령어에 copy-app 타입 처리 로직 구현  
- 관련 ISSUE: 지원되지 않는 앱 타입으로 인한 빌드 테스트 실패  
- 위치: `sbkube/commands/build.py`

[x] build 명령어에 install-yaml 타입 처리 로직 구현  
- 관련 ISSUE: 지원되지 않는 앱 타입으로 인한 빌드 테스트 실패  
- 위치: `sbkube/commands/build.py`

## 예제 설정 수정

[x] devops 예제에서 copy-app 타입을 지원되는 타입으로 변경  
- 관련 ISSUE: 지원되지 않는 앱 타입으로 인한 빌드 테스트 실패  
- 위치: `examples/k3scode/devops/config.yaml`

## 배포 기능 구현

[x] deploy 명령어에 install-yaml 타입 처리 로직 구현  
- 관련 ISSUE: 지원되지 않는 앱 타입으로 인한 빌드 테스트 실패  
- 위치: `sbkube/commands/deploy.py`

[x] template 명령어에 install-yaml 타입 처리 로직 구현  
- 관련 ISSUE: 지원되지 않는 앱 타입으로 인한 빌드 테스트 실패  
- 위치: `sbkube/commands/template.py`

## 테스트 환경 개선

[ ] 테스트에서 임시 sources.yaml 파일 생성 로직 추가  
- 관련 ISSUE: prepare 테스트 실패 - sources.yaml 파일 누락  
- 위치: `tests/conftest.py` 또는 개별 테스트 파일

[ ] 빌드 테스트에서 올바른 앱 타입 사용하도록 수정  
- 관련 ISSUE: 지원되지 않는 앱 타입으로 인한 빌드 테스트 실패  
- 위치: `tests/test_build.py`