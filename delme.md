## 자동화 도구 비교 구현: 신규 문의 이메일 자동 분류·배정 (Make vs Zapier)

작성자: 서인석 / 제출일: 2026-07-17

### 1. 사용한 도구

- Make, Zapier
	- 전부 시험용 버전을 사용함.
- Slack
- Google Sheet

### 2. 구현 과정 요약

동일한 워크플로우("신규 문의 메일 수신 → 긴급 키워드 여부로 분기 → Google Sheets 이력 기록 + Slack 알림")를 Make와 Zapier에 각각 구현했다. Make에서는 Router 모듈 하나로 두 분기를 한 시나리오 안에 표현했고, Zapier는 Filter가 단일 조건만 지원해 긴급용/일반용 Zap을 각각 만들어 동일한 Trigger를 공유하도록 구성했다. 두 도구 모두 Gmail 커넥터로 새 메일을 감지했고, 분기 결과에 따라 Google Sheets에 이력을 남긴 뒤 Slack으로 담당 채널에 알림을 보냈다.

#### 2.1 Make 구현 

- 구현 절차
	- ![](<working/make-t1/1-steps>)
- Model Diagram
	- ![](<working/make-t1/D1-model diagram.jpg>)
- Sample Mail Capture (Keyword 포함)
	- ![](<D2-gmail1.jpg>)
- Sample Mail Capture (Keyword 없음)
	- ![](<D3-gmail2.jpg>)
- Slack 결과 화면 (Keyword 포함)
	- ![](<D4-slack-channel1.jpg>)
- Slack 결과 화면 (Keyword 없음)
	- ![](<D5-slack-channel2.jpg>)

#### 2.2 Zapier 구현

- 구현절차
	- [[working/zapier-t1/1-steps]]
- Model 정의 
	- Model 2개 작성 
		- ![](<D1-ZAP workflow 정의 목록.jpg>)
	- Model - With Keyword / Without Keyword
		- ![](<D2-ZAP flow 정의 1.jpg>)
		- ![](<D3-ZAP flow 정의 2.jpg>)
- 입력 - Google Mail 
	- 메일 내용은 Make와 동일한 것을 사용함.
	- ![](<D4-gmail 목록.jpg>)
- Zap 실행 History
	- ![](<D5-zap 실행 기록.jpg>)
- 실행 결과 - Google Sheet
	- ![](<D6-google sheet 내용.jpg>)
- 실행 결과 - Slack
	- ![](<D7-slack channel 기록.jpg>)
### 3. Make/Zapier 비교

| 비교 항목       | Make                                 | Zapier                                 |
| ----------- | ------------------------------------ | -------------------------------------- |
| UI/UX       | 노드(시각적 캔버스) 기반, 흐름 한눈에 파악 용이         | 리스트(단계별) 기반, 직관적이나 전체 흐름 파악은 상대적으로 어려움 |
| 설정 난이도      | Router/모듈 개념 학습에 초기 진입장벽 있음          | Trigger-Action 순서만 알면 바로 시작 가능         |
| 연동 서비스 범위   | Gmail, Sheets, Slack 등 주요 서비스 폭넓게 지원 | 동일 서비스 지원하나 세부 트리거 옵션 구성이 다름           |
| 조건 분기 방식    | Router 하나로 다중 분기 표현 가능               | Filter는 단일 조건이라 분기마다 별도 Zap 필요         |
| 무료 플랜 범위    | 월 1,000 Ops                          | 월 100 Tasks                            |
| 실행 로그 확인 방식 | 시나리오 History에서 단계별 입출력 데이터 상세 확인     | Task History에서 Zap별 실행 결과 확인           |

### 4. 도구별 장단점

- Make는 하나의 시나리오 안에서 복잡한 분기와 다중 액션을 시각적으로 관리할 수 있어 조건이 많은 워크플로우에 유리하지만, 처음 접할 때 모듈/라우터 개념을 익히는 데 시간이 걸린다. 
- Zapier는 진입장벽이 낮고 각 단계가 단순 리스트로 표현되어 빠르게 결과물을 만들 수 있지만, 분기가 늘어날수록 Zap을 여러 개로 쪼개야 해서 관리 포인트가 늘어난다.

### 5. 적합 상황 의견

- 조건 분기가 2개 이상이거나 하나의 흐름 안에서 여러 갈래를 한눈에 관리해야 하는 업무라면 Make가 더 적합하다고 판단했다. 
- 반대로 분기 없이 단순한 Trigger-Action 자동화를 빠르게 만들어야 하는 상황, 혹은 노코드 도구를 처음 접하는 팀원이 유지보수까지 맡아야 하는 경우라면 Zapier의 직관적인 리스트형 UI가 더 유리하다.
- 시스템 유지 보수 측면에서는 Make가 사용자 친화적인 인터페이스를 가지고 있어 유리하다.

---

## 자유 주제 자동화: 부정 리뷰 실시간 감지 및 AI 요약 알림

### 1. 반복 업무 정의

고객 리뷰가 Google Sheets에 새로 쌓일 때마다 사람이 직접 읽고 부정적인 내용인지 판단해 요약한 뒤 담당자에게 전달하던 작업을, 별점 기준 자동 분기와 AI 요약으로 대체한다.

### 2. 도구 선정 및 이유

- Make를 선정했다. 
- 선정 이유
	 1. Router로 별점 조건 분기를 한 시나리오 안에서 처리할 수 있음.
	 2. OpenAI/Anthropic 모듈이 기본 제공되어 별도 API 연동 설정 없이 AI 요약 Action을 추가할 수 있음.
	 3. Error Handler 라우트를 붙여 실패 알림·대체 저장까지 하나의 시나리오로 완결 가능.

### 3. 워크플로우 흐름 설명

1. Trigger: Google Sheets – 새 리뷰 행 추가 감지
2. Router: 별점 3점 이하(부정) / 4점 이상(긍정) 분기
3. Action 1(부정 경로): Claude/ChatGPT 모듈 – 리뷰 3줄 요약 + 감정 키워드 추출
4. Action 2(부정 경로): Slack – 요약 내용과 함께 담당자 멘션 긴급 알림
5. Action(긍정 경로): Slack 아카이브 채널에 단순 기록
6. Error Handler: 시나리오 실행 실패 시 Gmail로 오류 알림 발송, Sheets 기록 실패 시 임시 백업 시트에 대체 저장

### 4. 구현 화면
- Model Diagram
	- ![](<working/make-free/D1-model diagram.jpg>)

### 5. 실행 결과

- 구현 절차
	- ![](<working/make-free/1-steps>)

- 입력 (Google Sheet)
	- ![](<D2-google sheet.jpg>)

- Slack Bad Review
	- ![](<D3-channel bad review.jpg>)

- Slack Good Review
	- ![](<D4-channel good review.jpg>)

### 6. 보너스 반영 사항

AI 연동 Action(리뷰 요약·감정 키워드 추출)과 실패 알림·대체 저장 로직을 모두 하나의 Make 시나리오 안에 구현해 보너스 1, 2를 함께 충족했다.