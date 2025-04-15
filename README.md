![image info](images/A2A_banner.png)

**_불투명한 에이전트 애플리케이션 간의 통신과 상호운용성을 가능하게 하는 오픈 프로토콜_**

<!-- TOC -->

- [Agent2Agent 프로토콜 A2A](#agent2agent-프로토콜-a2a)
    - [시작하기](#시작하기)
    - [기여하기](#기여하기)
    - [향후 계획](#향후-계획)
    - [소개](#소개)

<!-- /TOC -->

기업의 AI 도입에서 가장 큰 과제 중 하나는 서로 다른 프레임워크와 벤더에서 구축된 에이전트들이 함께 작동하도록 하는 것입니다. 이를 위해 우리는 *Agent2Agent (A2A) 프로토콜*을 만들었습니다. 이는 서로 다른 생태계의 에이전트들이 서로 통신할 수 있도록 돕는 협업 방식입니다. Google은 이 오픈 프로토콜 이니셔티브를 주도하고 있으며, 이 프로토콜이 **다양한 프레임워크나 벤더에 구애받지 않고 에이전트들에게 공통 언어를 제공함으로써 다중 에이전트 통신을 지원하는 데 필수적**이라고 믿기 때문입니다.

*A2A*를 통해 에이전트들은 서로의 기능을 보여주고 사용자와의 상호작용 방식(텍스트, 양식, 또는 양방향 오디오/비디오)을 협상할 수 있으며, 이 모든 것이 보안을 유지하면서 이루어집니다.

### **A2A 작동 방식 보기**

다른 에이전트 프레임워크 간의 원활한 통신을 A2A가 어떻게 가능하게 하는지 [이 데모 영상](https://storage.googleapis.com/gweb-developer-goog-blog-assets/original_videos/A2A_demo_v4.mp4)을 통해 확인하세요.

### 개념적 개요

Agent2Agent (A2A) 프로토콜은 독립적인 AI 에이전트 간의 통신을 용이하게 합니다. 다음은 핵심 개념들입니다:

*   **에이전트 카드:** 에이전트의 기능, 기술, 엔드포인트 URL, 인증 요구사항을 설명하는 공개 메타데이터 파일 (일반적으로 `/.well-known/agent.json`에 위치). 클라이언트는 이를 발견(discovery)에 사용합니다.
*   **A2A 서버:** A2A 프로토콜 메서드([JSON 명세](/specification)에 정의됨)를 구현하는 HTTP 엔드포인트를 노출하는 에이전트. 요청을 받고 작업 실행을 관리합니다.
*   **A2A 클라이언트:** A2A 서비스를 사용하는 애플리케이션이나 다른 에이전트. A2A 서버의 URL로 요청(예: `tasks/send`)을 보냅니다.
*   **작업:** 작업의 중심 단위. 클라이언트는 메시지(`tasks/send` 또는 `tasks/sendSubscribe`)를 보내 작업을 시작합니다. 작업은 고유 ID를 가지며 상태(`submitted`, `working`, `input-required`, `completed`, `failed`, `canceled`)를 거쳐 진행됩니다.
*   **메시지:** 클라이언트(`role: "user"`)와 에이전트(`role: "agent"`) 간의 통신 턴을 나타냅니다. 메시지는 `Parts`를 포함합니다.
*   **파트:** `Message` 또는 `Artifact` 내의 기본 콘텐츠 단위. `TextPart`, `FilePart`(인라인 바이트 또는 URI 포함), 또는 `DataPart`(구조화된 JSON, 예: 양식)일 수 있습니다.
*   **아티팩트:** 작업 중 에이전트가 생성한 출력물(예: 생성된 파일, 최종 구조화된 데이터)을 나타냅니다. 아티팩트도 `Parts`를 포함합니다.
*   **스트리밍:** 장기 실행 작업의 경우, `streaming` 기능을 지원하는 서버는 `tasks/sendSubscribe`를 사용할 수 있습니다. 클라이언트는 Server-Sent Events (SSE)를 통해 `TaskStatusUpdateEvent` 또는 `TaskArtifactUpdateEvent` 메시지를 받아 실시간 진행 상황을 확인할 수 있습니다.
*   **푸시 알림:** `pushNotifications`를 지원하는 서버는 클라이언트가 제공한 웹훅 URL로 작업 업데이트를 사전에 보낼 수 있으며, 이는 `tasks/pushNotification/set`을 통해 구성됩니다.

**일반적인 흐름:**

1.  **발견:** 클라이언트가 서버의 well-known URL에서 에이전트 카드를 가져옵니다.
2.  **시작:** 클라이언트가 초기 사용자 메시지와 고유 작업 ID를 포함하는 `tasks/send` 또는 `tasks/sendSubscribe` 요청을 보냅니다.
3.  **처리:**
    *   **(스트리밍):** 서버가 작업이 진행됨에 따라 SSE 이벤트(상태 업데이트, 아티팩트)를 보냅니다.
    *   **(비스트리밍):** 서버가 작업을 동기적으로 처리하고 응답에서 최종 `Task` 객체를 반환합니다.
4.  **상호작용 (선택적):** 작업이 `input-required` 상태에 들어가면, 클라이언트는 동일한 작업 ID를 사용하여 `tasks/send` 또는 `tasks/sendSubscribe`를 통해 후속 메시지를 보냅니다.
5.  **완료:** 작업은 결국 종료 상태(`completed`, `failed`, `canceled`)에 도달합니다.

### **시작하기**

* 📚 [기술 문서](https://google.github.io/A2A/#/documentation)를 읽고 기능을 이해하세요
* 📝 프로토콜 구조의 [JSON 명세](/specification)를 검토하세요
* 🎬 A2A 작동 방식을 보려면 [샘플](/samples)을 사용하세요
    * 샘플 A2A 클라이언트/서버 ([Python](/samples/python/common), [JS](/samples/js/src))
    * [다중 에이전트 웹 앱](/demo/README.md)
    * CLI ([Python](/samples/python/hosts/cli/README.md), [JS](/samples/js/README.md))
* 🤖 A2A를 에이전트 프레임워크에 적용하는 방법을 보려면 [샘플 에이전트](/samples/python/agents/README.md)를 사용하세요
    * [에이전트 개발 키트 (ADK)](/samples/python/agents/google_adk/README.md)
    * [CrewAI](/samples/python/agents/crewai/README.md)
    * [LangGraph](/samples/python/agents/langgraph/README.md)
    * [Genkit](/samples/js/src/agents/README.md)
* 📑 프로토콜 세부 사항을 이해하기 위해 주요 주제를 검토하세요
    * [A2A와 MCP](https://google.github.io/A2A/#/topics/a2a_and_mcp.md)
    * [에이전트 발견](https://google.github.io/A2A/#/topics/agent_discovery.md)
    * [엔터프라이즈 준비](https://google.github.io/A2A/#/topics/enterprise_ready.md)
    * [푸시 알림](https://google.github.io/A2A/#/topics/push_notifications.md)

### **기여하기**

기여를 환영합니다! 시작하려면 [기여 가이드](CONTRIBUTING.md)를 참조하세요.\
질문이 있으신가요? [GitHub 토론](https://github.com/google/A2A/discussions/)에서 커뮤니티에 참여하세요.\
프로토콜 개선에 대한 피드백을 제공하려면 [GitHub 이슈](https://github.com/google/A2A/issues)를 이용하세요.\
비공개 피드백을 보내고 싶으신가요? 이 [Google 양식](https://docs.google.com/forms/d/e/1FAIpQLScS23OMSKnVFmYeqS2dP7dxY3eTyT7lmtGLUa8OJZfP4RTijQ/viewform)을 사용하세요

### **향후 계획**

향후 계획에는 프로토콜 자체의 개선과 샘플의 향상이 포함됩니다:

**프로토콜 개선:**

*   **에이전트 발견:**
    *   `AgentCard` 내에 인증 스키마와 선택적 자격 증명을 직접 포함하는 것을 공식화합니다.
*   **에이전트 협업:**
    *   지원되지 않거나 예상치 못한 기술을 동적으로 확인하기 위한 `QuerySkill()` 메서드를 조사합니다.
*   **작업 수명주기 & UX:**
    *   작업 내에서 동적 UX 협상 지원 (예: 에이전트가 대화 중에 오디오/비디오 추가).
*   **클라이언트 메서드 & 전송:**
    *   작업 관리 이상의 클라이언트 시작 메서드 지원 확장을 탐구합니다.
    *   스트리밍 신뢰성과 푸시 알림 메커니즘 개선.

**샘플 & 문서 개선:**

*   "Hello World" 예제 단순화.
*   다양한 프레임워크와 통합되거나 특정 A2A 기능을 보여주는 추가 에이전트 예제 포함.
*   공통 클라이언트/서버 라이브러리에 대한 더 포괄적인 문서 제공.
*   JSON 스키마에서 사람이 읽을 수 있는 HTML 문서 생성.

### **소개**

A2A 프로토콜은 Google LLC가 운영하는 오픈 소스 프로젝트로, [라이선스](LICENSE)에 따라 전체 커뮤니티의 기여를 환영합니다.
