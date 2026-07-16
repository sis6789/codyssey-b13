Zapier로 "신규 문의 이메일 자동 분류·배정"(제안 1)을 구현하는 시나리오입니다. Zapier의 기본 Filter는 하나의 조건 경로만 통과시키므로, Make의 Router 하나로 표현했던 구조를 Zap 2개(긴급용 / 일반용)로 나눠 구현합니다. (유료 Paths 기능을 쓰면 한 Zap 안에서 분기 가능하지만, 무료 플랜 기준으로는 아래 방식을 권장합니다.)

**0단계 — 앱 연결 준비** Zapier에 Gmail, Google Sheets, Slack 세 앱을 각각 연결(로그인 인증)해둡니다.

---

## Zap 1: 긴급 문의 경로

**1단계 — Trigger: Gmail "New Email Matching Search"** Trigger 앱으로 Gmail 선택, 이벤트는 "New Email Matching Search"를 씁니다. Search string에 `subject:(긴급 OR 장애 OR 환불) is:unread`처럼 검색 조건을 넣어 Gmail 자체 검색 문법으로 1차 필터링합니다. 계정 연결 후 "Test trigger"로 샘플 메일을 하나 불러옵니다.

**2단계 — Filter by Zapier** Trigger 뒤에 "Filter" 단계를 추가합니다. Only continue if: Subject(Text) `Contains` 긴급 — 여러 키워드를 OR로 걸고 싶으면 Filter 조건 그룹을 추가(Or)해 장애, 환불도 각각 검사하도록 설정합니다. (1단계 검색어로 이미 걸러졌다면 이 Filter는 이중 안전장치 역할입니다.)

**3단계 — Action: Google Sheets "Create Spreadsheet Row"** 스프레드시트/워크시트를 지정하고, 컬럼에 Gmail 트리거의 동적 필드(수신일시, 발신자, 제목, 본문 일부)를 매핑합니다. "긴급여부" 컬럼에는 고정값으로 "긴급"을 입력합니다.

**4단계 — Action: Slack "Send Direct Message"** Slack 액션에서 "Send Direct Message" 이벤트를 선택합니다. To 필드에 담당자를 지정(고정 담당자면 드롭다운 선택, 발신 부서에 따라 달라지면 별도 Lookup Table 액션으로 이메일→Slack 사용자 매핑 후 변수 연결). Message 필드에 Gmail 변수를 넣어 `[긴급] {{제목}} - {{발신자}}` 형태로 구성합니다.

**5단계 — 테스트 및 켜기** 각 단계 옆 "Test" 버튼으로 개별 실행 확인 후, Zap 우측 상단 토글로 Zap을 켭니다(On).

---

## Zap 2: 일반 문의 경로

**6단계 — Zap 복제** Zap 1을 "Copy"로 복제하면 Trigger/구조를 그대로 재사용할 수 있어 편리합니다.

**7단계 — Trigger 검색어 및 Filter 조건 반전** Trigger의 Search string을 `-subject:(긴급 OR 장애 OR 환불) is:unread`처럼 제외 조건으로 바꾸고, Filter 단계도 Subject `Does not contain` 긴급/장애/환불(모두 불포함)으로 반대로 설정합니다. Zap 1과 겹치지 않게 하는 것이 핵심입니다.

**8단계 — Action 수정** Google Sheets Create Row는 그대로 두되 "긴급여부" 고정값만 "일반"으로 변경합니다. Slack 액션은 "Send Direct Message" 대신 "Send Channel Message"로 바꿔 일반 문의 채널에 조용히 기록되도록 합니다.

**9단계 — 테스트 및 켜기** 동일하게 각 단계 테스트 후 Zap을 켭니다.

---

**10단계 — 분기별 실행 확인** 긴급 키워드 포함 메일 1건, 미포함 메일 1건을 실제 발송해 Zap 1과 Zap 2가 각각 정확히 한 번씩 실행되는지 Zap History(Task History)에서 확인합니다. 이 로그 화면을 캡처해두면 보고서의 "분기별 1회 이상 실행 확인" 요건 증빙 자료로 바로 쓸 수 있습니다.