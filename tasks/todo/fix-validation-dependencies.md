# 🔧 TODO: 테스트 의존성 누락 문제 해결

**source**: issues  
**priority**: 높음  
**estimated_time**: 45분  

## 문제 재현
- [ ] Performance 테스트 실행하여 faker 에러 확인
  ```bash
  pytest tests/performance/ -v
  ```
- [ ] Integration 테스트에서 testcontainers 스킵 확인
  ```bash
  pytest tests/integration/ -v
  ```

## 원인 확정 & 해결 전략
**원인**: pyproject.toml에 테스트 선택적 의존성 미정의  
**전략**: 완전한 테스트 의존성 그룹 정의 및 환경별 분리

## 작업 항목

### [ ] pyproject.toml 의존성 완전 정의
- **파일**: `pyproject.toml`
- **섹션**: `[project.optional-dependencies]`
- **내용**:
  ```toml
  [project.optional-dependencies]
  test = [
      "pytest>=8.0",
      "pytest-cov",
      "pytest-mock",
      "pytest-asyncio",
      "pytest-benchmark",
      "sqlalchemy>=1.4.0",
      "faker>=1.0.0",
      "testcontainers>=3.0.0",
      "kubernetes>=18.0.0"
  ]
  
  test-minimal = [
      "pytest>=8.0", 
      "pytest-cov",
      "pytest-mock",
      "sqlalchemy>=1.4.0"
  ]
  ```

### [ ] Makefile 타겟 수정
- **파일**: `Makefile`
- **기존**: `install-test` 타겟이 존재하지 않는 [test] 참조
- **수정**:
  ```makefile
  install-test:
  	uv pip install -e ".[test]"
  
  install-test-minimal:
  	uv pip install -e ".[test-minimal]"
  ```

### [ ] CI/CD 환경별 의존성 설치 확인
- **파일**: `.github/workflows/test.yml`
- **검증**: 현재 `uv pip install -e ".[test]"` 명령어 동작 확인
- **필요시 수정**: 의존성 설치 스텝 업데이트

### [ ] 환경 분리 테스트
- **최소 환경**: Core 기능만 (faker, testcontainers 제외)
  ```bash
  uv pip install -e ".[test-minimal]"
  pytest tests/unit/ tests/integration/ --ignore=tests/performance/ -m "not requires_k8s"
  ```
- **전체 환경**: 모든 의존성 포함
  ```bash
  uv pip install -e ".[test]"
  pytest tests/ -v
  ```

## 성공 기준
- [ ] `make install-test` 명령어 정상 동작
- [ ] Performance 테스트에서 faker import 성공
- [ ] Integration 테스트에서 testcontainers 사용 가능 (선택적)
- [ ] CI에서 의존성 설치 성공

## 회귀 테스트
- **스크립트**: `scripts/test-dependencies.sh` (신규)
- **내용**: 환경별 의존성 설치 및 테스트 실행 검증
- **실행**:
  ```bash
  # 최소 의존성 테스트
  ./scripts/test-dependencies.sh minimal
  
  # 전체 의존성 테스트  
  ./scripts/test-dependencies.sh full
  ```

## 개발자 가이드 업데이트
- **파일**: `README.md`, `tests/README.md`
- **내용**: 환경별 의존성 설치 가이드 추가