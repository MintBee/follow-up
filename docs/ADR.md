# Architecture Decision Record

## 초기 ADR 논의 사항
 * AD-1: 상황(Situation) 데이터 저장 전략.
 * AD-2: 대량의 이벤트 목록에 대한 페이징(Paging) 방식 (occurredAt + id 커서 기반).
 * AD-3: 사용자당 생성 가능한 커스텀 주제 최대 개수 (제안: 20개).
 * AD-4: 이벤트 데이터 보강 및 중복 제거 로직 구현 방안.
 * AD-5: AI 토큰 비용 제어 방안 (사용자별/전체 한도 설정).
 * AD-6: 데이터 정제(Sanitization) 전략 세부 사항 (필터링할 패턴 목록).

