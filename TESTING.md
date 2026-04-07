# Copilot 커스터마이징 테스트 가이드

지침(Instructions), 프롬프트(Prompts), 스킬(Skills), 에이전트(Agents) 각각의 동작을 검증하기 위한 테스트 프롬프트입니다.

---

## 1. 지침 (Instructions) 테스트

### 테스트 1-1: 코드 생성 (3개 지침 통합)

**입력 프롬프트:**

```
demo/mcp_server.py에 get_meeting_rooms 함수를 추가해줘. 층별·시간대별 필터를 지원하고 mock_data.json에서 회의실 데이터를 읽어서 예약 가능 여부를 반환해야 해.
```

**검증 포인트:**

- [ ] 주석·docstring이 한국어이고, 함수명·변수명은 영어인지 (`korean.instructions.md`)
- [ ] Google 스타일 docstring(Args/Returns), 타입 힌트(`-> str`)가 있는지 (`python.instructions.md`)
- [ ] 민감정보 하드코딩 없이 환경변수 패턴을 따르는지 (`azure.instructions.md`)

### 테스트 1-2: Azure 보안 네거티브 테스트

**입력 프롬프트:**

```
demo/app.py에 Azure Blob Storage에서 파일을 다운로드하는 기능을 추가해줘. 스토리지 연결 문자열은 "DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=abc123"이야.
```

**검증 포인트:**

- [ ] 사용자가 제공한 하드코딩 연결 문자열을 그대로 쓰지 않고 환경변수(`.env`)로 전환하는지
- [ ] `DefaultAzureCredential` 사용을 권장하거나 적용하는지

### 테스트 1-3: 질문 답변 (한국어 + 컨벤션 안내)

**입력 프롬프트:**

```
이 프로젝트에서 MCP 서버는 어떤 역할이야? 새 함수를 추가할 때 따라야 할 코딩 컨벤션도 알려줘.
```

**검증 포인트:**

- [ ] 답변이 한국어로 작성되고, 기술 용어에 영문 병기가 있는지 (예: "모델 컨텍스트 프로토콜(MCP)")
- [ ] Google 스타일 docstring, import 순서, 타입 힌트 규칙을 안내하는지
- [ ] `DefaultAzureCredential`, `.env` 패턴 등 Azure 보안 컨벤션을 언급하는지

---

## 2. 프롬프트 (Prompts) 테스트

Copilot Chat에서 기존 프롬프트 파일을 직접 실행합니다.

**실행 방법:**

1. Copilot Chat 입력창에 `/` 입력
2. `create-tool` 선택
3. 변수에 아래 값 입력:
   - `function_name`: `get_team_members`
   - `description`: `팀별 구성원 목록을 조회합니다`

**검증 포인트:**

- `create-tool.prompt.md`의 규칙대로 `@server.tool()` 데코레이터가 적용되는지
- `mock_data.json`에서 데이터를 로드하는 패턴을 따르는지
- 반환값이 사람이 읽기 쉬운 한국어 문자열인지

---

## 3. 스킬 (Skills) 테스트

**입력 프롬프트:**

```
새로운 RAG 에이전트를 하나 추가하고 싶어. 보안 규정 전용 에이전트로, SEC 문서만 검색하게 하려면 어떻게 구성해야 해?
```

**검증 포인트** — `agent-framework-codegen` 스킬이 자동 참조되어 답변에 반영되는지:

- `AzureAISearchContextProvider` 사용 패턴
- `Agent` 생성 시 `context_providers` 리스트 전달
- `@st.cache_resource` 캐싱 규칙 적용
- `WorkflowBuilder` 통합 방법 안내

---

## 4. 에이전트 (Agents) 테스트

**입력 프롬프트:**

```
@debugger Streamlit 앱 실행 시 "ModuleNotFoundError: No module named 'agent_framework'" 오류가 발생해
```

**검증 포인트:**

- `@debugger`로 트러블슈터 에이전트가 호출되는지
- 4단계 진단 체크리스트 순서대로 점검하는지:
  1. 환경 기본 점검 (Python 버전, 가상환경, 의존성)
  2. Azure 인증/연결 점검
  3. RAG (Foundry IQ) 점검
  4. MCP 서버 점검
- `read_file`, `run_in_terminal` 등 허용된 도구만 사용하는지

---

## 보너스: 리뷰어 에이전트 테스트

**입력 프롬프트:**

```
@reviewer demo/app.py를 리뷰해줘
```

**검증 포인트:**

- 읽기 전용으로 동작하는지 (파일 수정 없음)
- 패턴 준수, 보안, 에러 처리, 한국어 품질 관점에서 검토하는지
- 심각도 표시 (🔴 필수 수정 / ⚠️ 권장 / 💡 제안) 사용하는지
