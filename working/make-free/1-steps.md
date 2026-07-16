Make로 "부정 리뷰 실시간 감지 및 AI 요약 알림"(제안 3)을 구현하는 시나리오입니다. Router 분기 + AI Action(보너스1) + Error Handler(보너스2)까지 한 시나리오 안에 담습니다.

**0단계 — 사전 준비** Make에서 Google Sheets, Slack, OpenAI(ChatGPT) 또는 Anthropic(Claude), Gmail 네 개의 커넥션을 미리 인증해둡니다. 리뷰가 쌓이는 시트에는 최소 "별점", "리뷰내용", "작성일시" 컬럼이 있어야 합니다.

**1단계 — Trigger: Google Sheets "Watch Rows"** Google Sheets 앱의 "Watch Rows" 모듈을 첫 블록으로 추가합니다. 스프레드시트/시트 탭 지정, Table contains headers 체크, 처음 실행 시 Limit은 1~2로 낮게 잡아 테스트합니다. 폴링 주기는 스케줄에서 설정(예: 15분)하고, 테스트는 Run once로 수동 실행합니다.

**2단계 — Router 추가** Watch Rows 뒤에 Router를 연결해 2개 경로를 만듭니다.

**3단계 — 경로 1 필터: 부정 리뷰** 경로 1 연결선에 Filter 설정: 별점(Rating) 컬럼 `Less than or equal to` 3. Watch Rows의 별점 필드를 대상으로 매핑합니다.

**4단계 — 경로 2 필터: 긍정 리뷰** 경로 2 필터: 별점 `Greater than or equal to` 4. (별점 체계가 5점 만점이 아니라면 기준값만 조정합니다.)

**5단계 — 경로1 Action 1: AI 요약 모듈** 경로 1에 OpenAI "Create a Chat Completion"(또는 Anthropic "Create a Message") 모듈을 연결합니다. 프롬프트 예시: "다음 고객 리뷰를 3줄로 요약하고, 부정적인 감정 키워드를 2~3개 뽑아줘: {{리뷰내용}}". Model은 비용이 낮은 모델(gpt-4o-mini 등)을 선택해 무료/저비용 범위 내에서 운영합니다.

**6단계 — 경로1 Action 2: Slack 긴급 알림** AI 모듈 뒤에 Slack "Create a Message"를 연결합니다. Destination Type은 User(DM) 또는 담당 채널로 지정하고, 메시지 본문에 AI 모듈의 출력값(요약, 키워드)과 원본 리뷰 링크·작성일시를 함께 삽입해 `[부정 리뷰 감지] {{AI요약}} - 키워드: {{키워드}}` 형태로 구성합니다.

**7단계 — 경로2 Action: Slack 아카이브 기록** 경로 2에는 AI 모듈 없이 바로 Slack "Create a Message"를 연결해, 별도의 "리뷰 아카이브" 채널에 간단히 기록만 남깁니다(멘션 없이 로그 성격).

**8단계 — Error Handler(보너스2) 추가** 핵심 모듈(예: 6단계 Slack 알림, 또는 AI 모듈)을 우클릭 → "Add error handler"를 선택합니다. 에러 핸들러 경로 안에 두 가지를 연결합니다.

- Gmail "Send an Email" 모듈: 담당자에게 "시나리오 실행 실패 - [모듈명] 오류" 알림 발송.
- Google Sheets "Add a Row" 모듈: 원래 리뷰 데이터를 "임시 백업 시트"에 대체 저장(Directive는 Resume로 설정해 이후 흐름이 중단되지 않도록 처리).

**9단계 — 테스트** 시트에 별점 2점짜리 테스트 리뷰 1건, 별점 5점짜리 테스트 리뷰 1건을 입력해 Run once로 실행하고, 두 경로가 각각 실행되는지, AI 요약이 정상 생성되는지, Slack에 도착하는지 확인합니다. 이어서 AI 모듈의 API 키를 일시적으로 잘못된 값으로 바꾸는 등 의도적으로 오류를 발생시켜 Error Handler가 Gmail 알림과 백업 시트 저장까지 정상 동작하는지도 확인합니다(확인 후 키는 원상복구).

**10단계 — 활성화** 모든 테스트가 정상이면 시나리오 좌상단 스케줄 토글을 켜서 Trigger 발생 시 자동 실행되도록 전환합니다.

이 로그 화면들(경로별 실행, AI 요약 결과, 오류 알림 도착)을 캡처해두면 보고서의 "실행 결과"와 "보너스 반영 사항" 증빙 자료로 그대로 쓸 수 있습니다.