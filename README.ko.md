# Luna Chat Coder

[English README](README.md)

**Version 0.1.4**

평소 쓰는 ChatGPT 웹 채팅으로 실제 GitHub 리포지토리 작업을 해보세요. 별도의 로컬 코딩 에이전트를 띄우거나, 터널을 열거나, 내 컴퓨터를 채팅에 연결할 필요가 없습니다.

ChatGPT에는 이미 코드를 실행할 수 있는 샌드박스가 있습니다. 다만 네트워크 접근이 제한될 수 있어서 소스를 가져오고, 의존성을 준비하고, 큰 변경을 안정적으로 게시하는 단계에서 작업이 쉽게 막힙니다. Luna는 이 내장 샌드박스에서 개발 작업을 이어가고, 부족한 부분만 연결된 GitHub를 통해 보완하도록 모델에게 알려줍니다.

## 무엇이 좋아지나요?

- **내장 샌드박스를 제대로 활용합니다.** 편집, 빌드, 테스트, 디버깅은 가능한 한 채팅 안의 샌드박스에서 이어갑니다.
- **환경 문제에 덜 막힙니다.** 평소 경로로 어떤 단계를 안정적으로 끝낼 수 없을 때는 필요한 부분만 GitHub를 이용해 처리하고, 전체 작업을 다른 환경으로 옮기지 않습니다.
- **중간에 끊겨도 다시 이어가기 쉽습니다.** Chat이나 sandbox가 사라져도 대화 내용을 바탕으로 코드를 다시 만드는 대신 정확한 GitHub 상태에서 복구합니다.
- **게시 결과를 확인합니다.** 어떤 상태에서 작업했는지와 실제로 게시된 결과를 확인한 뒤 완료를 보고합니다.

목표는 단순합니다. 새 인프라를 운영하는 대신, 채팅에 리포지토리와 개발 작업만 알려주는 것입니다.

## 빠른 시작

이 리포지토리가 문서화하는 ChatGPT Web 환경에서는 다음과 같이 설정합니다.

1. **Use this template → Create a new repository**를 선택합니다.
2. ChatGPT의 <https://chatgpt.com/plugins>에서 **GitHub Plugin**을 설치하고 연결합니다.
3. GitHub에서 <https://github.com/apps/chatgpt-codex-connector>의 **ChatGPT Codex Connector**를 설치하고 새 리포지토리에 접근 권한을 부여합니다. 이미 일부 리포지토리만 허용하도록 설치했다면 새 리포지토리를 그 목록에 추가합니다.
4. 일반 ChatGPT 대화에서 리포지토리 URL과 원하는 개발 작업을 보냅니다. 예를 들어 변경을 구현하고 pull request를 열어 달라고 요청할 수 있습니다.

평소 사용법은 여기까지입니다. 이 template으로 만든 리포지토리에는 Luna가 이미 들어 있고, Luna 이름을 따로 언급하거나 내부 우회 절차를 직접 운영할 필요가 없어야 합니다.

Organization 정책에 따라 Plugin이나 GitHub App 사용에 관리자 승인이 필요할 수 있습니다.

## 어떻게 동작하나요?

Luna는 리포지토리 자체의 지침과 요구사항을 읽고, 작업해야 할 정확한 소스를 복구한 뒤, 평소 편집과 테스트는 chat sandbox에서 진행합니다.

샌드박스의 직접 접근만으로 부족한 단계는 먼저 연결된 GitHub 경로를 이용합니다. 그 경로로도 해당 단계를 안정적으로 끝내기 어렵다면 제한된 GitHub Actions 실행으로 처리한 뒤, 가능하면 다시 샌드박스에서 작업을 이어갑니다. GitHub Actions는 기본 개발환경으로 사용하지 않습니다.

## 기존 리포지토리에 추가

다음 skill directory 전체를 복사합니다.

```text
.agents/skills/luna-chat-coder/
```

그 다음 [`AGENTS.md`](AGENTS.md)의 짧은 Luna entry-point instruction을 대상 리포지토리의 기존 agent instruction에 합칩니다. 프로젝트 자체의 engineering guidance는 그대로 두십시오. Luna는 그것을 대체하지 않고 주변에서 작업을 이어가기 쉽게 돕습니다.

이 리포지토리가 문서화하는 ChatGPT Web 환경에서는 작업을 요청하기 전에 GitHub Plugin을 연결하고 ChatGPT Codex Connector에 해당 리포지토리 접근 권한을 부여합니다.

## 문서

일반 작업에서 쓰는 동작은 [`SKILL.md`](.agents/skills/luna-chat-coder/SKILL.md)에 정의되어 있습니다. 필요할 때 사용하는 세부 절차는 [`actions-missions.md`](.agents/skills/luna-chat-coder/references/actions-missions.md)와 [`recovery.md`](.agents/skills/luna-chat-coder/references/recovery.md)에 있습니다. [`design-rationale.md`](.agents/skills/luna-chat-coder/references/design-rationale.md)는 Luna 자체를 변경할 때 참고하는 maintainer memory이며, 일반 skill 동작은 이 문서를 읽지 않아도 완결됩니다.

Luna는 Agent Skills 구조를 따릅니다. 이 리포지토리가 문서화하고 검증하는 환경은 ChatGPT Web이며, 다른 host도 동등한 sandbox와 GitHub 기능을 실제로 제공한다면 같은 skill을 사용할 수 있습니다.

## 라이선스

MIT. [`LICENSE`](LICENSE)를 참고하십시오.
