<!-- 
status: converted
new_file: /tasks/todo/fix-tracker-session-issues.md
converted_date: 2025-07-10
-->

# ❗ DeploymentTracker 세션 문제

## 문제 요약
DeploymentTracker 테스트에서 SQLAlchemy DetachedInstanceError 발생
- tracker.py:124에서 `deployment.id` 접근 시 발생
- DeploymentDatabase 설계 문제의 연장선

## 에러 상세
```
sqlalchemy.orm.exc.DetachedInstanceError: Instance <Deployment at 0x7f89d18b8d10> is not bound to a Session; attribute refresh operation cannot proceed
```

## 원인 분석
1. `DeploymentTracker.track_deployment()` 내부에서 `DeploymentDatabase.create_deployment()` 호출
2. `create_deployment()`가 반환한 객체가 세션에서 분리됨
3. `tracker.py:124`에서 `deployment.id` 접근 시 lazy loading 시도하며 에러 발생

## 근본 원인
DeploymentDatabase의 설계 문제:
- 컨텍스트 매니저로 세션 관리하지만 반환된 객체는 detached 상태
- 객체를 반환하는 메서드들이 세션 외부에서 사용할 수 없음

## 해결 방안

### 1. 단기 (테스트 레벨)
- DeploymentTracker 테스트를 mock 기반으로 변경
- 실제 database 의존성 제거

### 2. 중기 (설계 수정)
- DeploymentDatabase 메서드에서 필요한 속성 미리 로드
- 또는 세션 관리 방식 변경

### 3. 장기 (아키텍처 개선)
- Repository 패턴 적용
- 세션 라이프사이클 재설계

## 현재 상태
- Database 테스트: ✅ 6개 모두 통과
- Tracker 테스트: ❌ 세션 문제로 실패
- Rollback 테스트: ❓ 미확인

## 우선순위
- 중간 (Database 테스트는 작동함)
- 전체 state 기능의 핵심은 database이므로 tracker/rollback은 차순위