<!-- 
status: converted
new_file: /tasks/todo/fix-validation-dependencies.md
converted_date: 2025-07-10
-->

# ❗ 테스트 의존성 누락 문제

## 문제 요약
테스트 실행에 필요한 의존성 패키지들이 누락되어 일부 테스트 실행 불가

## 누락된 의존성

### 1. faker
- **영향**: performance 테스트 실행 불가
- **에러**: `ModuleNotFoundError: No module named 'faker'`
- **위치**: `tests/performance/conftest.py:19`

### 2. testcontainers
- **영향**: integration 테스트 10개 스킵
- **메시지**: "testcontainers not available, skipping K8s integration tests"
- **위치**: integration 테스트 전반

## 원인 분석
1. `pyproject.toml`에 테스트 의존성이 선택적 의존성으로 정의되지 않음
2. 현재 `make install-test`가 존재하지 않는 `[test]` extra를 참조
3. 개발 환경과 CI 환경 간 의존성 불일치

## 해결 방안

### 1. pyproject.toml 수정
```toml
[project.optional-dependencies]
test = [
    "pytest>=8.0",
    "pytest-cov",
    "pytest-mock", 
    "faker>=1.0.0",
    "testcontainers>=3.0.0",
    "pytest-asyncio",
    "pytest-benchmark"
]
```

### 2. Makefile 수정
```makefile
install-test:
    uv pip install -e ".[test]"
```

### 3. 환경 분리
- **최소 테스트**: core 기능만 (faker, testcontainers 제외)
- **전체 테스트**: 모든 의존성 포함

## 임시 우회

### Performance 테스트 비활성화
```bash
pytest tests/unit/ tests/integration/ --ignore=tests/performance/
```

### Integration 테스트 마커 활용
```bash
pytest -m "not requires_k8s"
```

## 영향도
- **우선순위**: 높음 (CI/CD 영향)
- **영향 범위**: 15개 테스트 + 전체 test 스크립트
- **해결 복잡도**: 낮음 (설정 변경)

## 후속 작업
1. pyproject.toml 의존성 정의
2. CI 환경에서 의존성 설치 검증
3. 개발자 환경 설정 가이드 업데이트