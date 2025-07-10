<!-- 
status: converted
new_file: /tasks/todo/fix-state-session-issues.md
converted_date: 2025-07-10
-->

# ❗ SQLAlchemy 세션 문제 - State 테스트

## 문제 요약
분할된 state 테스트에서 SQLAlchemy DetachedInstanceError 발생
- `Instance <Deployment> is not bound to a Session`
- 데이터베이스 세션 관리 문제

## 에러 상세
```
sqlalchemy.orm.exc.DetachedInstanceError: Instance <Deployment at 0x7efe11f42ba0> is not bound to a Session; attribute refresh operation cannot proceed
```

## 원인 분석
1. 테스트 fixture에서 생성된 객체가 세션에서 분리됨
2. 테스트 분할 과정에서 세션 관리 코드 누락 가능성
3. 원본 테스트에서 사용하던 세션 컨텍스트가 새 파일에서 누락

## 해결 방안
1. **즉시 수정**: conftest.py에서 세션 관리 fixture 추가
2. **코드 검토**: 원본 test_deployment_state.py의 세션 관리 방식 확인
3. **대안**: 테스트 메서드 내에서 session.add() 및 session.commit() 추가

## 임시 우회
현재 단계에서는 state 테스트 제외하고 다른 리팩토링 테스트 검증 진행

## 상태
- 상태: 해결 필요
- 우선순위: 높음 (core 기능)
- 영향: state 관련 15개 테스트 실행 불가