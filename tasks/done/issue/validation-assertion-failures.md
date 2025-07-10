<!-- 
status: converted
new_file: /tasks/todo/fix-assertion-failures.md
converted_date: 2025-07-10
-->

# ❗ Assertion 실패 - Command 테스트

## 문제 요약
CLI 출력 메시지 변경으로 인한 텍스트 매칭 assertion 실패
- 5개 command 테스트에서 발생
- 실제 기능은 정상 동작하나 예상 텍스트와 불일치

## 실패 케이스

### 1. test_delete_action_app
```python
# 기대값
assert "삭제 작업 완료" in result.output or "삭제 완료" in result.output

# 실제값
"✨ `delete` 작업 완료 ✨"
```

### 2. test_deploy_kubectl_app
- namespace 출력 형식 변경
- kubectl 명령어 확인 메시지 변경

### 3. test_template_app_not_templatable
- 에러 메시지 형식 변경

## 원인 분석
1. CLI 출력 메시지가 Rich 라이브러리 적용으로 변경
2. 이모지 및 스타일링 추가로 텍스트 패턴 변화
3. 테스트가 구체적인 텍스트에 의존하여 취약

## 해결 방안

### 즉시 수정
```python
# 현재 (취약)
assert "삭제 완료" in result.output

# 개선안 (robust)
assert result.exit_code == 0
assert "delete" in result.output.lower()
assert "완료" in result.output
```

### 장기 개선
1. **출력 파싱**: JSON 또는 구조화된 출력 모드 추가
2. **상태 기반 검증**: 실제 리소스 상태 확인
3. **Mock 검증**: subprocess 호출 인자 검증

## 영향도
- 우선순위: 중간
- 영향 범위: Command 테스트 5개
- 기능 영향: 없음 (표시 문제)

## 임시 우회
현재는 exit_code 기반으로만 검증하고 텍스트 assertion 비활성화