# 🔧 TODO: SQLAlchemy 세션 문제 해결

**source**: issues  
**priority**: 높음  
**estimated_time**: 2시간  

## 문제 재현
- [x] State 테스트 실행하여 DetachedInstanceError 확인
  ```bash
  pytest tests/unit/state/test_deployment_database.py -v
  pytest tests/unit/state/test_deployment_tracker.py -v
  ```
  **결과**: 
  - Database 테스트: 6개 모두 통과 ✅
  - Tracker 테스트: 4개 실패 (DetachedInstanceError at tracker.py:124) ❌

## 원인 확정 & 해결 전략
**원인**: 컨텍스트 매니저로 세션 관리하지만 반환 객체가 detached 상태  
**전략**: 세션 내에서 모든 속성 접근 완료 또는 eager loading 적용

## 작업 항목

### [ ] 문제 정확한 재현 및 분석
- **재현 스크립트**: `debug/session-issue.py`
- **내용**: 최소한의 코드로 DetachedInstanceError 재현
- **분석**: 정확히 어느 지점에서 세션 분리되는지 확인

### [ ] DeploymentDatabase 설계 수정
- **파일**: `sbkube/state/database.py`
- **방법 1**: 컨텍스트 매니저 내에서 필요 속성 미리 로드
  ```python
  def create_deployment(self, deployment_data):
      with self.get_session() as session:
          deployment = Deployment(...)
          session.add(deployment)
          session.flush()
          
          # 필요한 속성 미리 접근하여 로드
          _ = deployment.id
          _ = deployment.deployment_id
          return deployment
  ```

### [ ] 테스트 방식 개선
- **파일**: `tests/unit/state/test_deployment_database.py`
- **방법 2**: 세션 컨텍스트 내에서 모든 검증 완료
  ```python
  def test_create_deployment(self, deployment_db, sample_deployment_data):
      with deployment_db.get_session() as session:
          deployment = Deployment(...)
          session.add(deployment)
          session.flush()
          
          # 세션 내에서 모든 assertion 수행
          assert deployment.deployment_id == "expected"
          assert deployment.id is not None
  ```

### [ ] DeploymentTracker 세션 관리 수정
- **파일**: `sbkube/state/tracker.py:124`
- **문제**: `deployment.id` 접근 시 lazy loading 시도
- **해결**: 
  ```python
  # 현재 (문제)
  deployment = self.db.create_deployment(data)
  self.active_deployments[deployment.id] = deployment
  
  # 수정 (해결)
  deployment = self.db.create_deployment(data)
  deployment_id = deployment.id  # 미리 접근
  self.active_deployments[deployment_id] = deployment
  ```

### [ ] Repository 패턴 적용 검토
- **파일**: `sbkube/state/repository.py` (신규)
- **목적**: 세션 라이프사이클을 더 명확하게 관리
- **패턴**:
  ```python
  class DeploymentRepository:
      def create(self, data) -> dict:  # 객체 대신 dict 반환
          with self.session() as session:
              deployment = Deployment(...)
              session.add(deployment)
              session.commit()
              return {
                  'id': deployment.id,
                  'deployment_id': deployment.deployment_id,
                  # 필요한 속성들
              }
  ```

## 성공 기준
- [ ] Database 테스트 6개 모두 통과 유지
- [ ] Tracker 테스트에서 DetachedInstanceError 해결
- [ ] Rollback 테스트도 정상 동작
- [ ] 세션 관리 패턴이 일관성 있게 적용

## 회귀 테스트
- **파일**: `tests/unit/state/test_session_management.py` (신규)
- **목적**: 세션 라이프사이클 및 객체 상태 테스트
- **내용**:
  ```python
  def test_session_lifecycle():
      """세션 생성, 사용, 정리가 올바르게 동작하는지 테스트"""
      
  def test_object_detachment():
      """객체가 detached 되는 시점 및 처리 테스트"""
  ```

## 장기 개선 계획
- 전체 state 모듈의 아키텍처 리뷰
- 세션 관리 모범 사례 문서화
- 타 ORM 라이브러리 검토 (필요시)