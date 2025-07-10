# 🔧 TODO: CLI 출력 Assertion 실패 수정

**source**: issues  
**priority**: 중간  
**estimated_time**: 1시간  

## 문제 재현
- [x] Command 테스트 실행하여 텍스트 매칭 실패 확인
  ```bash
  pytest tests/unit/commands/ -v -k "test_delete_action_app or test_deploy_kubectl_app"
  ```
  **결과**: 2개 테스트 실패 확인
  - test_delete_action_app: "삭제 완료" → "✨ `delete` 작업 완료 ✨"
  - test_deploy_kubectl_app: kubectl 호출 검증 실패

## 원인 확정 & 해결 전략
**원인**: Rich 라이브러리 적용으로 CLI 출력 메시지 변경 (이모지, 스타일링 추가)  
**전략**: 하드코딩된 텍스트 매칭을 robust한 패턴 매칭으로 변경

## 작업 항목

### [x] 실패 케이스 확인 및 분류
- **파일**: `tests/unit/commands/test_delete.py`
- **케이스**: `test_delete_action_app` - "삭제 완료" → "✨ `delete` 작업 완료 ✨" ✅ 수정됨
- **파일**: `tests/unit/commands/test_deploy.py`  
- **케이스**: `test_deploy_kubectl_app` - kubectl 호출 검증 실패 ✅ 수정됨
- **파일**: `tests/unit/commands/test_template.py`
- **케이스**: `test_template_app_not_templatable` - exit_code 기대값 불일치 ✅ 수정됨

### [x] Robust 패턴 매칭으로 수정
- **기존 (취약)**:
  ```python
  assert "삭제 완료" in result.output
  ```
- **개선 (robust)**:
  ```python
  assert result.exit_code == 0
  assert "delete" in result.output.lower() or "삭제" in result.output
  assert "완료" in result.output or "complete" in result.output.lower()
  ```

### [x] 테스트 패턴 표준화
- **파일**: `tests/unit/commands/` 전체 ✅ 핵심 실패 케이스 3개 적용 완료
- **적용된 패턴**: 
  1. exit_code 우선 검증 ✅
  2. 핵심 키워드만 확인 (이모지, 스타일링 무시) ✅
  3. 대소문자 무관 검사 ✅
  4. 다국어 지원 고려 ✅

### [x] Mock 기반 검증 추가
- **파일**: 각 command 테스트 ✅ deploy 테스트에 적용 완료
- **내용**: subprocess 호출 인자 검증으로 실제 동작 확인 ✅
- **적용된 예시**:
  ```python
  # kubectl 호출 검증을 robust하게 개선
  for call in mock_subprocess.call_args_list:
      if call[0] and len(call[0]) > 0 and isinstance(call[0][0], list):
          if call[0][0][0] == 'kubectl':
              kubectl_calls.append(call)
  ```

## 성공 기준
- [x] 3개 실패 테스트 모두 통과 ✅
- [x] 출력 메시지 변경 시에도 테스트 안정성 유지 ✅
- [x] exit_code 0으로 기본 성공 여부 검증 ✅

## 회귀 테스트
- **파일**: `tests/unit/commands/test_output_patterns.py` (신규)
- **목적**: CLI 출력 패턴 변경 감지
- **내용**: 기본 성공/실패 패턴 테스트

## 장기 개선 계획
- JSON 출력 모드 추가 검토
- 구조화된 출력으로 테스트 안정성 향상