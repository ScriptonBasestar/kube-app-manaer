<!-- 
status: converted
new_file: /tasks/todo/fix-import-dependencies.md
converted_date: 2025-07-10
-->

# ❗ 의존성 Import 문제

## 문제 요약
분할된 테스트 파일에서 모듈 import 실패
- `ModuleNotFoundError: No module named 'sqlalchemy'`
- 개발 환경에 필요한 의존성 설치 필요

## 원인
- sbkube 패키지가 개발 모드로 설치되지 않음
- 테스트 의존성 미설치

## 해결 방법
```bash
# 개발 모드로 설치
uv pip install -e .

# 테스트 의존성 설치  
uv pip install -e ".[test]"

# 또는 Makefile 사용
make install-test
```

## 현재 상태
- 디렉토리 구조 생성 완료
- 파일 분할 및 이동 완료
- import 문제로 테스트 실행 검증 보류

## 다음 단계
1. 의존성 설치 후 테스트 재실행
2. import 경로 수정 (필요시)
3. 마커 추가 완료