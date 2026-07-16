Make(구 Integromat)의 Slack 앱 기준으로, DM과 채널 메시지 모두 **"Send a Message"(메시지 생성)** 모듈 하나로 처리합니다. Channel type 설정만 다르게 하면 됩니다.

**0. 사전 준비 — Slack 연결 만들기**

1. 시나리오에 Slack 모듈 추가 → Create a connection 클릭
2. 연결 타입 선택: Slack (User) 또는 Slack (Bot)
    - Bot 연결로 채널에 보내려면 봇이 반드시 해당 채널 멤버여야 함 (아니면 "channel not found" 에러)
3. Slack 워크스페이스 인증/권한 승인 후 저장

**1. Slack 채널에 메시지 보내기**

1. "Send a Message" 모듈 추가
2. Channel id or name 입력 방식 선택 (직접 입력 또는 목록에서 선택)
3. 채널 지정: 이름(`#general`) 또는 채널 ID(`C0123ABC12`)
4. Channel type → Public channel 또는 Private channel 선택
5. Text 필드에 보낼 메시지 내용 입력 (필요시 Blocks로 리치 메시지 구성 가능)
6. (선택) Thread message id 입력하면 스레드 답글로 전송
7. 저장 후 실행/테스트

**2. Slack DM(다이렉트 메시지) 보내기**

1. 동일하게 "Send a Message" 모듈 추가
2. Channel type → Direct message 선택 (다수에게 보낼 땐 Direct message to multiple people)
3. Id type 선택: Channel ID / User ID / Username 중 하나로 수신자 지정
4. 해당 필드에 수신자 값 입력 (예: 사용자 이메일로 먼저 "Search for a User" 모듈로 User ID를 조회해서 매핑하는 방식이 가장 안전)
5. Text에 메시지 내용 입력
6. 저장 후 실행/테스트

**팁**

- User 필드에 어떤 포맷이 들어가는지 헷갈리면: 매핑을 끄고 드롭다운에서 사용자를 직접 선택 → 다시 매핑을 켜면 그 필드가 요구하는 정확한 포맷(보통 User ID)이 표시됩니다.
- 이메일로 사용자를 찾으려면 "Search for a User" 모듈(email 입력) → 결과의 User ID를 "Send a Message"의 Direct message 필드에 매핑.
- Bot 연결 시 icon emoji, icon url, user name 필드로 발신자 표시를 커스터마이징할 수 있습니다 (User 연결에는 적용 안 됨).

Sources:

- [Slack modules - Make Apps Documentation](https://apps.make.com/slack-modules)
- [Slack - Make Apps Documentation](https://www.make.com/en/help/app/slack)
- [Slack - Create a Message to send direct message (Make Community)](https://community.make.com/t/slack-create-a-message-to-send-direct-message/55624)