# 🔧 TODO: DeploymentTracker 세션 문제 해결

**source**: issues  
**priority**: 중간  
**estimated_time**: 1.5시간  

## 문제 재현
- [ ] DeploymentTracker 테스트 실행하여 세션 에러 확인
  ```bash
  pytest tests/unit/state/test_deployment_tracker.py::test_track_deployment -v -s
  ```

## 원인 확정 & 해결 전략
**원인**: `DeploymentDatabase.create_deployment()`가 반환한 객체가 detached 상태에서 `deployment.id` 접근 시도  
**전략**: Tracker를 mock 기반으로 변경하거나 Database와의 통합 방식 개선

## 작업 항목

### [ ] 현재 상황 정확한 파악
- **파일**: `sbkube/state/tracker.py:124`
- **에러 지점**: `self.active_deployments[deployment.id] = deployment`
- **호출 경로**: `track_deployment()` → `DeploymentDatabase.create_deployment()`

### [ ] 단기 해결: Mock 기반 테스트로 변경
- **파일**: `tests/unit/state/test_deployment_tracker.py`
- **방법**: DeploymentDatabase를 mock하여 세션 의존성 제거
  ```python
  @patch('sbkube.state.tracker.DeploymentDatabase')
  def test_track_deployment(self, mock_db_class):
      mock_db = mock_db_class.return_value
      mock_deployment = MagicMock()
      mock_deployment.id = 123
      mock_deployment.deployment_id = "test-deploy"
      mock_db.create_deployment.return_value = mock_deployment
      
      tracker = DeploymentTracker()
      result = tracker.track_deployment(sample_data)
      
      assert result is not None
      assert 123 in tracker.active_deployments
  ```

### [ ] 중기 해결: DeploymentDatabase 인터페이스 개선
- **파일**: `sbkube/state/database.py`
- **방법**: 필요한 정보만 반환하는 메서드 추가
  ```python
  def create_deployment_and_get_id(self, deployment_data) -> Tuple[int, str]:
      """배포 생성 후 필요한 ID들만 반환"""
      with self.get_session() as session:
          deployment = Deployment(...)
          session.add(deployment)
          session.flush()
          return deployment.id, deployment.deployment_id
  ```

### [ ] DeploymentTracker 로직 수정
- **파일**: `sbkube/state/tracker.py`
- **현재 문제 코드**:
  ```python
  deployment = self.db.create_deployment(data)
  self.active_deployments[deployment.id] = deployment  # 여기서 에러
  ```
- **개선 방안**:
  ```python
  # 방안 1: ID만 사용
  deployment_id, deploy_name = self.db.create_deployment_and_get_id(data)
  self.active_deployments[deployment_id] = {
      'deployment_id': deploy_name,
      'created_at': datetime.now(),
      'status': 'active'
  }
  
  # 방안 2: 세션 내에서 ID 미리 추출
  with self.db.get_session() as session:
      deployment = self.db._create_deployment_in_session(session, data)
      deployment_id = deployment.id
      self.active_deployments[deployment_id] = deployment
  ```

### [ ] 단위 테스트 분리
- **Database 테스트**: 순수 데이터베이스 로직만 테스트 (이미 통과)
- **Tracker 테스트**: mock 기반으로 비즈니스 로직만 테스트
- **통합 테스트**: 실제 Database와 Tracker 연동 테스트는 integration/에서

## 성공 기준
- [ ] `pytest tests/unit/state/test_deployment_tracker.py -v` 모든 테스트 통과
- [ ] Database 기능은 영향받지 않음 (6개 테스트 계속 통과)
- [ ] Tracker의 핵심 비즈니스 로직 검증 가능

## 회귀 테스트
- **파일**: `tests/integration/test_state_integration.py` (신규)
- **목적**: Database + Tracker 실제 통합 동작 검증
- **내용**: 실제 SQLite 사용하여 end-to-end 테스트

## 우선순위 조정 근거
- Database 테스트는 이미 완전히 작동 (core 기능 보장)
- Tracker는 Database 위의 추가 레이어 (비즈니스 로직)
- Mock 테스트로도 충분히 Tracker 로직 검증 가능
- 실제 통합은 integration 테스트에서 커버