---
title: "Model Context Protocol: MCP 핵심 정리"
writer: chanho
date: 2025-04-29 14:10:00 +0900
categories: [AI, LargeLanguageModel]
tags: [ai, llm, mcp]
---

> 요즘 이곳 저곳에서 MCP 관련된 질문을 많이 받고 있다. 이에 간략하게 해당 내용을 정리하고자 한다. 해당 게시글을 Anthropic의 Model Context Protocol의 공식 문서를 참조 해서 작성하였다.

## MCP란?

> MCP(Model Context Protocol)를 간략하게 정의하면 AI 모델이 파일-시스템, DB, SaaS API 같은 외부 자원과 표준화된 방식으로 ‘실시간 양방향’ 통신을 할 수 있게 해 주는 오픈 프로토콜 로서, MCP를 통해 LLM이 다양한 외부 데이터 소스와 도구에 쉽게 연결할 수 있다.


💡`Protocol: 프로토콜은 서로 다른 시스템끼리 데이터를 주고받을 때 따라야 하는 규칙과 절차를 정의한 약속 입니다. 쉽게 말해, 컴퓨터들끼리 소통하기 위한 공통 언어`

- Anthropic에서는 MCP를 USB-C 포트처럼 “한 번 꽂으면 어떤 기기든 연결되는” AI용 통합 커넥터로 비유 하기도 하였다.

<img width="100%" alt="image" src="https://github.com/user-attachments/assets/0a9c42ad-18ad-4646-b230-467d2e065160">

<sub><i>출처:  https://norahsakal.com/blog/mcp-vs-api-model-context-protocol-explained/</i></sub>

MCP는 에이전트와 복잡한 워크플로우 구축을 쉽게 할 수 있고 다양한 LLM을 유연하게 전환할 수 있도록 한다. 또한 데이터 보안과 인프라 내 통합을 쉽게 관리할 수 있도록 할 수 있다.

### 주요 특징

1. MCP는 LLM과 외부 데이터 및 도구를 연결하는 방식을 표준화 한다.
2. USB-C처럼 하나의 규격으로 다양한 자원에 연결할 수 있게 한다.
3. 오픈소스로 공개되어 누구나 자유롭게 사용 및 개선할 수 있다.

### MCP가 주목 받게 된 계기

MCP는 Anthropic에서 2024년 11월에 오픈소스로 공개하였다. 당시에는 초기 개념을 제시하였지만, 생각보다 많은 흥행을 이루지는 못하고 있었다. 이때, 2025년 2월에 AI IDE인 “Cursor”에서 MCP를 지원하면서 커뮤니티들을 중심으로 MCP에 대한 관심이 크게 증가하게 되었으며, 2025년 3월 27일 OpenAI 제품군에 MCP 채택을 발표하며 사실상 업계 표준 후보로 격상하게 되었다.

### 기존 방식과 차이점

| 구분 | 전통적 API/플러그인 | **MCP** |
| --- | --- | --- |
| 통합 단위 | 서비스마다 별도 API | **단일 표준**으로 툴·데이터 일괄 연결 |
| 통신 방향 | 요청-응답 **단방향** | **지속형 양방향** 스트림 (멀티턴 대화에 최적) |
| 도구 발견 | 하드코딩 필요 | 런타임에 **동적 디스커버리** |
| 확장성 | 통합마다 코드 증가 | “서버 하나 추가 = 기능 한 줄 추가” |
| 사용 난이도 | 개발자 중심 | 자연어 지시 + 간단 설정으로도 가능 |

> 관점 차이
>
> - **API/플러그인** : “모델이 **정의된 함수**를 한 번 호출”
> - **MCP** : “모델이 **살아있는 세션**에서 데이터를 읽고, 쓰고, 명령까지 실행”

## MCP의 통신 구조

> MCP는 `JSON-RPC 2.0` 메시지 포맷을 사용하여 통신한다. Host, Client, Server 간의 통신을 설정한다.


<img width="100%" alt="image" src="https://github.com/user-attachments/assets/cfbe6e48-c735-4d14-b978-88a8bfaadfaa">

<sub><i>출처: https://modelcontextprotocol.io/introduction</i></sub>

- `MCP Host`: LLM 애플리케이션(IDE, Claude Desktop등)으로, MCP 클라이언트를 통해 서버에 연결한다.
- `MCP Client`: 호스트 내에서 서버와의 통신을 담당하는 구성 요소로 Host 안에서 서버와 1:1 연결을 유지하는 프로토콜 클라이언트이다.
- `MCP Server`: 특정 기능이나 데이터를 제공하는 서비스로 MCP를 통해 클라이언트와 통신한다.
- `Local Data Sources`: MCP 서버가 액세스 할 수 있는 사용자 컴퓨터의 파일, 데이터베이스 및 서비스
- `Remote Services`: MCP 서버가 연결할 수 있는 인터넷을 통해 사용 가능한 외부 시스템

### Key Details

1. Stateful Connections: 상태 유지 연결

: 상태 유지 연결은 클라이언트와 서버 간의 연결이 지속되어, 이전의 상호 작용 정보를 바탕으로 연속적인 통신이 가능함을 의미한다. 이러한 방식은 복잡한 작업 흐름이나 사용자 세션을 관리하는 데 유리하다.

1. Server and client capability negotiation: 기능 협상

: MCP에서는 클라이언트와 서버가 서로의 기능을 협상하여 사용할 수 있는 기능을 결정한다. 예를 들어, 서버가 특정 도구나 리소스를 제공할 수 있는지, 클라이언트가 이를 지원하는지 등을 확인하여 최적의 통신 환경을 구성한다.

1. JSON-RPC 2.0 message format

: JSON-RPC는 가볍고 운송 방식에 독립적으로 HTTP, Socket, Process 내 등 어디서나 사용이 가능하다. JSON 형식을 사용하는 원격 프로시저 호출(Remote Procedure Call, RPC) 프로토콜이다. 주요 개념은 “메서드 이름”과 “파라미터”를 JSON 오브젝트로 보내고, “결과” 또는 “에러”를 JSON 오브젝트로 돌려받는다는 것이다.

`👉 기본 메시지 구조`

1. 요청 객체 (Request Object)

```json
{
  "jsonrpc": "2.0",           // 버전
  "method": "methodName",     // 호출할 메서드 이름 (필수)
  "params": {...} or [...],   // 메서드에 전달할 인자 (선택)
  "id": 1                     // 요청 식별자 (필수, 응답과 매칭됨)
}

```

`❗ 요청에서 id를 생략하면 "Notification"이 되고 Notification은 결과를 기대하지 않는 요청이다.`

1. 응답 객체 (Response Object)

```json
{
  "jsonrpc": "2.0",
  "result": {...},            // 성공 결과 (성공 시 필수)
  "id": 1                     // 요청 시 보낸 id 그대로
}

```

또는 에러가 발생했다면,

```json
{
  "jsonrpc": "2.0",
  "error": {
    "code": -32601,            // 에러 코드
    "message": "Method not found", // 에러 메시지
    "data": {...}              // 추가적인 에러 정보 (선택)
  },
  "id": 1
}

```

`❗ result와 error는 동시에 존재하면 안 되고 둘 중 하나만 있어야 한다.`

1. 에러 객체 (Error Object)

| 항목 | 설명 |
| --- | --- |
| `code` | 에러 종류를 나타내는 정수형 코드 |
| `message` | 에러를 간단히 설명하는 짧은 문자열 |
| `data` | (선택) 추가적인 디버깅 정보나 설명 |

**주요 에러 코드 예시**

| 코드 | 의미 |
| --- | --- |
| -32700 | Parse error: JSON 파싱 실패 |
| -32600 | Invalid Request: 요청 형식 오류 |
| -32601 | Method not found: 존재하지 않는 메서드 호출 |
| -32602 | Invalid params: 파라미터 오류 |
| -32603 | Internal error: 서버 내부 오류 |
1. 배치 요청(Batch Requests): 한 번에 여러 요청을 배열(Array) 로 묶어 보낼 수 있다.

```json
[
  {"jsonrpc": "2.0", "method": "sum", "params": [1,2,4], "id": "1"},
  {"jsonrpc": "2.0", "method": "subtract", "params": [42,23], "id": "2"},
  {"jsonrpc": "2.0", "method": "get_data", "id": "3"}
]

```

- 서버는 각각의 요청에 대한 응답 배열(Array) 을 반환한다.
- 요청과 응답은 id로 매칭한다.
- Notification(응답이 없는 요청)은 응답 배열에 포함되지 않는다.

### 실제로 어디에 활용할 수 있는가?

1. 개인 생산성: 노션·구글 드라이브 MCP 서버 연결 → AI가 리서치, 초안 작성, 문서 저장을 한 번에.
2. 코딩 워크플로우: IDE ⇄ Git / 패키지매니저 / 테스트러너를 MCP로 묶어, 에이전트가 코드 작성-커밋-빌드까지 자동.
3. 기업 데이터 허브: CRM·데이터웨어하우스·슬랙을 AI 에이전트가 실시간 조회·갱신 → 영업 리포트 자동화.
4. 멀티모달 제작 파이프라인: Blender·Figma·FFmpeg MCP 서버로 디자인 → 렌더 → 편집 작업을 일괄 지시.

## MCP Core architecture

<img width="100%" alt="image" src="https://github.com/user-attachments/assets/6953a1bc-813c-4776-97c9-aaf34a727de6">

<sub><i>출처: https://modelcontextprotocol.io/docs/concepts/architecture</i></sub>

### Protocol layer

> 메시지 프레이밍, 요청/응답 연결, 고수준 통신 패턴을 관리한다.


```python
class Session(BaseSession[RequestT, NotificationT, ResultT]):
    async def send_request(
        self,
        request: RequestT,
        result_type: type[Result]
    ) -> Result:
        """Send request and wait for response. Raises McpError if response contains error."""
        # Request handling implementation

    async def send_notification(
        self,
        notification: NotificationT
    ) -> None:
        """Send one-way notification that doesn't expect response."""
        # Notification handling implementation

    async def _received_request(
        self,
        responder: RequestResponder[ReceiveRequestT, ResultT]
    ) -> None:
        """Handle incoming request from other side."""
        # Request handling implementation

    async def _received_notification(
        self,
        notification: ReceiveNotificationT
    ) -> None:
        """Handle incoming notification from other side."""
        # Notification handling implementation
```

### Transport layer

> 실제 데이터 전송을 담당하는 층이다.

- `stdio`: 표준 입력/ 출력을 사용하며 로컬 프로세스에 적합하다.
- `HTTP + SSE`: 서버에서 클라이언트는 SSE를 사용하고 클라이언트에서 서버는 HTTP POST를 사용해서 전송할 수 있다.

`💡 모든 통신은 JSON-RPC 2.0 포맷을 사용한다.`

### Message Types

- Request: 요청
- Result: 응답
- Error: 요청 처리 실패 시 반환
- Notification: 응답을 기대하지 않는 단 방향 메시지

```tsx
interface Request { method: string; params?: { ... }; }
interface Result { [key: string]: unknown; }
interface Error { code: number; message: string; data?: unknown; }
interface Notification { method: string; params?: { ... }; }
```

### Connection Lifecycle

1. `Initialization`: 클라이언트가 초기화 요청(`initialize`)을 보내면 서버가 초기화 응답(`initialize response`)을 보내고 클라이언트가 초기화 완료 알림(`initialized notification`)을 보냄
2. `Message exchange`: 요청/응답(Request-Response) 또는 알림(Notifications) 방식으로 자유롭게 통신
3. `Termination`: 어느 한 쪽이 `close()` 호출, 연결 종료 또는 에러 발생 시 종료

### 구현 예시

```python
import asyncio
import mcp.types as types
from mcp.server import Server
from mcp.server.stdio import stdio_server

app = Server("example-server")

@app.list_resources()
async def list_resources() -> list[types.Resource]:
    return [
        types.Resource(
            uri="example://resource",
            name="Example Resource"
        )
    ]

async def main():
    async with stdio_server() as streams:
        await app.run(
            streams[0],
            streams[1],
            app.create_initialization_options()
        )

if __name__ == "__main__":
    asyncio.run(main())
```

## Resources

> MCP 리소스는 서버가 LLM에 다양한 데이터(텍스트, 바이너리)를 제공할 수 있도록 표준화된 방식으로 제공하는 핵심 구조이다.


💡 리소스는 기본적으로 애플리케이션이 제어를 하며 모델이 직접 제어하도록 하려면 Tools 사용을 권장한다.

### Resource URIs

> 리소스는 아래 형식을 따르는 URI를 사용하여 식별되며, 프로토콜 및 경로 구조는 MCP 서버 구현에 의해 정의된다.


```
[protocol]://[host]/[path]
```

### Resource Discovery

> 리소스 탐색 방법에는 서버가 ‘resources/list’ 엔드포인트를 통해 리소스 목록을 제공하는 Direct resoures 방식과 URI Template 기반으로 동적 리소스를 제공하는 Resource Templates 방식이 있다.


```tsx
{
  uri: string;           // Unique identifier for the resource
  name: string;          // Human-readable name
  description?: string;  // Optional description
  mimeType?: string;     // Optional MIME type
}
```

### Reading resources

> 클라이언트가 `resources/read` 요청으로 리소스를 읽으며, 서버는 다음 형태로 응답한다. (디렉토리처럼 여러 파일을 반환할 수도 있다.)


```tsx
{
  contents: [
    {
      uri: string;
      mimeType?: string;
      text?: string;   // 텍스트 리소스
      blob?: string;   // 바이너리 리소스 (base64)
    }
  ]
}

```

## Resource updates

- **리스트 변경 알림:** 서버가 `notifications/resources/list_changed` 로 목록 변경을 알림
- **콘텐츠 변경 구독:** 클라이언트가 `resources/subscribe` → 서버가 업데이트 시 `resources/updated` 알림

### 리소스 지원 구현 예시

```python
app = Server("example-server")

@app.list_resources()
async def list_resources():
    return [types.Resource(uri="file:///logs/app.log", name="Application Logs", mimeType="text/plain")]

@app.read_resource()
async def read_resource(uri):
    if str(uri) == "file:///logs/app.log":
        return await read_log_file()
    raise ValueError("Resource not found")

# 서버 실행
async with stdio_server() as streams:
    await app.run(streams[0], streams[1], app.create_initialization_options())

```

## Prompt

> 프롬프트는 MCP 서버가 재사용 가능한 프롬프트 템플릿과 워크플로우를 정의하여 클라이언트와 LLM이 쉽게 사용할 수 있도록 해주는 기능이다. 즉, 개발자가 미리 정의해 놓은 프롬프트 템플릿들의 Hub로 서버는 여러 프롬프트를 등록해두고, 클라이언트는 필요할 때 선택하거나 인자만 채워서 사용할 수 있다. 즉, 개발자 입장에서는 “이런 식으로 질문하면 좋다.”하는 표준 프롬프트 세트를 미리 만들어 제공하고 사용자 입장에서는 복잡하게 고민할 필요 없이, 미리 준비된 프롬프트를 골라서 간단한 인자만 입력하거나 바로 클릭해서 편하게 사용할 수 있게 된다.


### Prompt Structure

```tsx
{
  name: string;              // Unique identifier for the prompt
  description?: string;      // Human-readable description
  arguments?: [              // Optional list of arguments
    {
      name: string;          // Argument identifier
      description?: string;  // Argument description
      required?: boolean;    // Whether argument is required
    }
  ]
}
```

### 요청

- 클라이언트는 `prompts/list` 엔드포인트를 호출하여 사용 가능한 프롬프트 목록을 조회할 수 있다.
- 특정 프롬프트를 사용할 때는 `prompts/get` 요청을 보낸다.

### Dynamic Prompts

프롬프트는 리소스를 삽입하거나 다단계 워크플로우를 가질 수 있다.

👉 리소스 포함 예시

- 로그 파일과 코드 파일을 함께 분석하는 프롬프트
- `resource` 타입으로 리소스 삽입

```tsx
{
  "name": "analyze-project",
  "description": "Analyze project logs and code",
  "arguments": [
    {
      "name": "timeframe",
      "description": "Time period to analyze logs",
      "required": true
    },
    {
      "name": "fileUri",
      "description": "URI of code file to review",
      "required": true
    }
  ]
}
```

👉 다중 단계 워크플로우 예시

- 에러 상황을 디버깅하는 대화 플로우
- 사용자 입력 → 추가 질문 → 응답 반복

```tsx
{
  "messages": [
    {
      "role": "user",
      "content": {
        "type": "text",
        "text": "Analyze these system logs and the code file for any issues:"
      }
    },
    {
      "role": "user",
      "content": {
        "type": "resource",
        "resource": {
          "uri": "logs://recent?timeframe=1h",
          "text": "[2024-03-14 15:32:11] ERROR: Connection timeout in network.py:127\n[2024-03-14 15:32:15] WARN: Retrying connection (attempt 2/3)\n[2024-03-14 15:32:20] ERROR: Max retries exceeded",
          "mimeType": "text/plain"
        }
      }
    },
    {
      "role": "user",
      "content": {
        "type": "resource",
        "resource": {
          "uri": "file:///path/to/code.py",
          "text": "def connect_to_service(timeout=30):\n    retries = 3\n    for attempt in range(retries):\n        try:\n            return establish_connection(timeout)\n        except TimeoutError:\n            if attempt == retries - 1:\n                raise\n            time.sleep(5)\n\ndef establish_connection(timeout):\n    # Connection implementation\n    pass",
          "mimeType": "text/x-python"
        }
      }
    }
  ]
}
```

### 프롬프트 구현 예시

```python
from mcp.server import Server
import mcp.types as types

# Define available prompts
PROMPTS = {
    "git-commit": types.Prompt(
        name="git-commit",
        description="Generate a Git commit message",
        arguments=[
            types.PromptArgument(
                name="changes",
                description="Git diff or description of changes",
                required=True
            )
        ],
    ),
    "explain-code": types.Prompt(
        name="explain-code",
        description="Explain how code works",
        arguments=[
            types.PromptArgument(
                name="code",
                description="Code to explain",
                required=True
            ),
            types.PromptArgument(
                name="language",
                description="Programming language",
                required=False
            )
        ],
    )
}

# Initialize server
app = Server("example-prompts-server")

@app.list_prompts()
async def list_prompts() -> list[types.Prompt]:
    return list(PROMPTS.values())

@app.get_prompt()
async def get_prompt(
    name: str, arguments: dict[str, str] | None = None
) -> types.GetPromptResult:
    if name not in PROMPTS:
        raise ValueError(f"Prompt not found: {name}")

    if name == "git-commit":
        changes = arguments.get("changes") if arguments else ""
        return types.GetPromptResult(
            messages=[
                types.PromptMessage(
                    role="user",
                    content=types.TextContent(
                        type="text",
                        text=f"Generate a concise but descriptive commit message "
                        f"for these changes:\n\n{changes}"
                    )
                )
            ]
        )

    if name == "explain-code":
        code = arguments.get("code") if arguments else ""
        language = arguments.get("language", "Unknown") if arguments else "Unknown"
        return types.GetPromptResult(
            messages=[
                types.PromptMessage(
                    role="user",
                    content=types.TextContent(
                        type="text",
                        text=f"Explain how this {language} code works:\n\n{code}"
                    )
                )
            ]
        )

    raise ValueError("Prompt implementation not found")
```

## Tools

> 서버를 통해 LLM이 실제 행동을 수행할 수 있도록 지원한다. LLM이 Tools를 통해 외부 시스템과 상호작용 하거나, 계산을 수행하거나 작업을 수행할 수 있다.


`💡Tools는 모델 제어형(model-controlled)으로 설계되었다. 즉, 서버에서 클라이언트로 Tools를 노출하고, LLM이 이를 자동으로 호출할 수 있도록 의도되어 있다.`

- 탐색(Discovery): 클라이언트가 `tools/list` 엔드포인트를 통해 사용 가능한 도구를 조회
- 호출(Invocation): `tools/call` 엔드포인트를 통해 도구를 호출하고 결과를 반환
- 유연성(Flexibility): 간단한 계산부터 복잡한 API 연동까지 가능

### Tool definition structure

```tsx
{
  name: string;          // Unique identifier for the tool
  description?: string;  // Human-readable description
  inputSchema: {         // JSON Schema for the tool's parameters
    type: "object",
    properties: { ... }  // Tool-specific parameters
  },
  annotations?: {        // Optional hints about tool behavior
    title?: string;      // Human-readable title for the tool
    readOnlyHint?: boolean;    // If true, the tool does not modify its environment
    destructiveHint?: boolean; // If true, the tool may perform destructive updates
    idempotentHint?: boolean;  // If true, repeated calls with same args have no additional effect
    openWorldHint?: boolean;   // If true, tool interacts with external entities
  }
}
```

### Implementing tools

```tsx
app = Server("example-server")

@app.list_tools()
async def list_tools() -> list[types.Tool]:
    return [
        types.Tool(
            name="calculate_sum",
            description="Add two numbers together",
            inputSchema={
                "type": "object",
                "properties": {
                    "a": {"type": "number"},
                    "b": {"type": "number"}
                },
                "required": ["a", "b"]
            }
        )
    ]

@app.call_tool()
async def call_tool(
    name: str,
    arguments: dict
) -> list[types.TextContent | types.ImageContent | types.EmbeddedResource]:
    if name == "calculate_sum":
        a = arguments["a"]
        b = arguments["b"]
        result = a + b
        return [types.TextContent(type="text", text=str(result))]
    raise ValueError(f"Tool not found: {name}")
```

## Sampling

> 서버가 LLM으로부터 응답 생성을 요청할 수 있도록 지원한다. 이 기능은 에이전트 행동을 가능하게 하면서도 보안과 개인정보 보호를 유지한다.


### 샘플링 작동 방식

1. 서버가 `sampling/createMessage` 요청을 클라이언트에 보냄
2. 클라이언트가 요청을 검토 및 수정 가능
3. 클라이언트가 LLM에 샘플링 요청
4. 클라이언트가 생성된 응답 검토
5. 클라이언트가 결과를 서버에 반환

이런 사람 개입(human-in-the-loop) 구조는 사용자가 LLM이 무엇을 보고 생성하는지에 대해 통제권을 유지할 수 있게 한다.

### Message format

```tsx
{
  messages: [
    {
      role: "user" | "assistant",
      content: {
        type: "text" | "image",
        text?: string,
        data?: string,   // base64 인코딩 (이미지의 경우)
        mimeType?: string
      }
    }
  ],
  modelPreferences?: {
    hints?: [{ name?: string }],
    costPriority?: number,
    speedPriority?: number,
    intelligencePriority?: number
  },
  systemPrompt?: string,
  includeContext?: "none" | "thisServer" | "allServers",
  temperature?: number,
  maxTokens: number,
  stopSequences?: string[],
  metadata?: Record<string, unknown>
}

```

### 요청 파라미터 상세

- Messages
    - 대화 이력을 `messages` 배열에 담아 보냅니다.
        - `role`: "user" 또는 "assistant"
        - `content`: 텍스트 또는 이미지 (base64 인코딩)
- Model Preferences
    - 모델 선택 선호도를 제시할 수 있다.
    - `hints`: 모델 이름 제안 (ex. "claude-3", "sonnet")
    - 우선순위(0~1):
        - `costPriority`: 비용 절감 중요도
        - `speedPriority`: 응답 속도 중요도
        - `intelligencePriority`: 모델 성능 중요도
- System Prompt
    - 시스템 프롬프트를 지정할 수 있습니다.
    - 단, 클라이언트가 수정하거나 무시할 수 있습니다.
- Context inclusion
    - `"none"`: 추가 컨텍스트 없음
    - `"thisServer"`: 요청 서버의 컨텍스트만 포함
    - `"allServers"`: 모든 MCP 서버의 컨텍스트 포함
- Sampling parameters
    - `temperature`: 무작위성 조정 (0~1)
    - `maxTokens`: 최대 토큰 수
    - `stopSequences`: 생성 중단 조건 문자열 배열
    - `metadata`: 기타 제공자별 추가 파라미터

### 응답 포맷

```tsx
{
  model: string,  // Name of the model used
  stopReason?: "endTurn" | "stopSequence" | "maxTokens" | string,
  role: "user" | "assistant",
  content: {
    type: "text" | "image",
    text?: string,
    data?: string,
    mimeType?: string
  }
}
```

### 요청 예시

```tsx
{
  "method": "sampling/createMessage",
  "params": {
    "messages": [
      {
        "role": "user",
        "content": {
          "type": "text",
          "text": "What files are in the current directory?"
        }
      }
    ],
    "systemPrompt": "You are a helpful file system assistant.",
    "includeContext": "thisServer",
    "maxTokens": 100
  }
}
```

## Roots

> Roots는 MCP에서 서버가 동작할 수 있는 범위를 정의하는 개념이다. 클라이언트가 서버에게 어떤 리소스가 관련 있는지와 어디에 위치하는지 알려주는 방법을 제공한다. (루트는 주로 파일 시스템 경로에 사용되지만, HTTP URL처럼 유효한 URL이라면 무엇이든 될 수 있다.)
> 

```tsx
file:///home/user/projects/myapp
https://api.example.com/v1
```

### Roots를 사용하는 이유

Roots는 다음과 같은 중요한 역할을 한다.

1. **가이드 제공**: 서버에 관련 리소스 및 위치를 알려준다.
2. **명확성 확보**: 워크스페이스에 속하는 리소스를 명확히 구분한다.
3. **조직화 지원**: 여러 개의 루트를 통해 다양한 리소스를 동시에 다룰 수 있다.

### Roots 작동 방식

클라이언트가 루트를 지원하는 경우

1. 연결 시 `roots` 기능을 선언한다.
2. 서버에 **추천하는 roots 목록**을 제공한다.
3. (지원하는 경우) 루트가 변경될 때 서버에 알린다.

서버는 다음을 권장

- 클라이언트가 제공한 roots를 존중
- roots URI를 기준으로 리소스를 찾아 접근
- root 경계 내에서 작업을 우선 처리

`💡루트는 **강제사항은 아니며** 정보 제공용이다.`

### 주요 활용 사례

- 프로젝트 디렉토리 지정
- Git 저장소 위치 설정
- 외부 API 엔드포인트 관리
- 설정 파일 위치 지정
- 리소스 접근 범위 제한

---

## 마무리

> Model Context Protocol의 정의를 정리하기 위해 공식문서를 읽으며 느낀 생각과 추가적인 조사, 그리고 정리한 내용을 공유하고자 한다.


### 1. MCP는 특별한 것이 아니라, 필요한 규격화다

MCP를 처음 접했을 때 들었던 인상은 다음과 같다.

"지금까지 각자 알아서 코드단에서 만들어 쓰던 것들을 하나의 규칙으로 정리한 거 아닌가?"

문서를 읽으면서 이 생각은 확신으로 바뀌었다.

> 기존에는 LLM을 활용할 때 서비스나 프로젝트마다 제각기 “파일이나 API 같은 리소스를 어떻게 제공할지”, “외부 시스템과 상호작용하는 Tool을 어떻게 호출할지”, “공통으로 쓰이는 프롬프트 템플릿을 어떻게 관리할지”, ”서버와 모델 간의 통신 포맷을 어떻게 만들지” 에 대해서 모두 각자 알아서 자유롭게 설계했다.  MCP는 이 흐트러진 생태계를 위해 "서로 다르지만 연결 가능하도록 최소한의 규칙"을 정의한 것이었다. 마치 모든 디지털 기기가 각자 다른 충전 포트를 사용하던 혼란스러운 시대를 끝내고, USB-C 하나로 통합한 것처럼, 정리하면, MCP는 혁신이 아니라 필연적 규격화라는 결론을 내렸다.


### 2. 기존 에이전트의 Tool 호출 구조는 MCP로 대체될 수 있다

내가 특히 흥미롭게 본 부분은 에이전트와 Tool 호출의 변화였다.

기존 구조(LangGraph 예시):

```
(LangGraph 에이전트) → (Tool 함수 직접 호출)
```

MCP를 도입하면 구조는 이렇게 바뀔것 같다.

```
(LangGraph 에이전트) → (MCP Client) → (MCP Server) → (Tool 실행)
```

> 즉, 기존에는 에이전트가 코드 안에서 Tool을 직접 불렀다면, MCP 환경에서는 Tool 자체가 별도의 MCP 서버로 독립하고, 에이전트는 MCP Client를 통해 Tool 서버에 요청을 보내는 방식으로 바뀐다. 따라서, MCP를 제대로 적용하려면 기존 Tool들을 MCP Server로 서버화해야 하고 에이전트 쪽은 MCP Client를 통해 표준화된 방법으로 Tool을 호출해야 한다. 특히, 여러 종류의 Tool 서버(MCP 서버)를 동시에 연결하는 것도 가능해져서 아주 유연하고 강력한 "멀티 Tool 에이전트" 환경을 만들 수 있다는 가능성도 느꼈다.


결론적으로,

- MCP는 기존 LLM 활용을 규격화하고 표준화하려는 흐름이다.
- Tool 호출 구조를 재설계해야 하지만, 그만큼 확장성과 유연성이 커진다.
- 앞으로 MCP 기반 생태계 확장과 Tool/Resource 마켓화 흐름이 자연스럽게 일어날 것이다.
