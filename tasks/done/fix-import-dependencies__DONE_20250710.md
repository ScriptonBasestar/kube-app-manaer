# 🔧 TODO: 의존성 Import 문제 해결

**source**: issues  
**priority**: 높음  
**estimated_time**: 30분  

## 문제 재현
- [x] 현재 테스트 실행해서 ModuleNotFoundError 확인
  ```bash
  pytest tests/unit/state/ -v
  ```
  **결과**: 의존성 import 문제 없음 확인. 실제 문제는 SQLAlchemy 세션 관리였음.

## 원인 확정 & 해결 전략
**원인**: sbkube 패키지가 개발 모드로 설치되지 않음 + 테스트 의존성 미설치  
**전략**: pyproject.toml 테스트 의존성 정의 후 개발 환경 설치

## 작업 항목

### [x] pyproject.toml 테스트 의존성 정의
- **파일**: `pyproject.toml` ✅ 이미 완료됨
- **현재 상태**: `[dependency-groups]` 섹션에 test 그룹 정의됨
- **포함된 의존성**:
  ```toml
  test = [
      "pytest>=8.3.5",
      "pytest-cov>=4.1.0",
      "pytest-mock>=3.12.0",
      "faker>=22.0.0",
      "testcontainers[k3s]>=4.0.0",
      "kubernetes>=28.1.0",
      # 기타 테스트 관련 패키지들
  ]
  ```

### [x] 개발 환경 설치 스크립트 실행
- **명령어**: ✅ 성공적으로 실행됨
  ```bash
  uv pip install -e . --group test
  ```
- **결과**: 20개 패키지 설치 완료 (faker, testcontainers, kubernetes 등)

### [x] 설치 검증 테스트
- **검증 스크립트**: ✅ 모두 성공
  ```bash
  python -c "import sqlalchemy; import faker; print('의존성 설치 성공')"
  pytest tests/unit/state/test_deployment_database.py -v
  ```
- **결과**: 
  - 의존성 import 성공 ✅
  - Database 테스트 6개 모두 통과 ✅

### [x] 회귀 테스트 추가
- **파일**: `tests/test_imports.py` ✅ 생성 완료
- **내용**: 핵심 모듈 import 테스트 (3개 테스트)
- **목적**: 향후 의존성 문제 조기 감지 ✅
- **결과**: 모든 import 테스트 통과

## 성공 기준
- [x] `pytest tests/unit/` 실행 시 ModuleNotFoundError 없음 ✅
- [x] 모든 분할된 테스트 파일이 정상 import ✅ (121개 테스트 통과)
- [x] CI/CD에서 의존성 설치 성공 ✅ (uv pip install 성공)

## 후속 작업
- Makefile의 `install-test` 타겟 검증
- CI 환경에서 의존성 설치 확인