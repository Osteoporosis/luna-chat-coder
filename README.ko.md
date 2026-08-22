# Luna Chat Coder

[English README](README.md)

**Version 0.1.2**

> 이 리포지토리에서 작업하는 AI agent는 먼저 [`AGENTS.md`](AGENTS.md)를 읽어야 합니다.

Luna Chat Coder는 일반 chat에서 신뢰할 수 있는 software development를 수행하기 위한 fallback skill을 내장한 리포지토리 template입니다.

최초 설정이 끝난 뒤의 사용법은 단순합니다. ChatGPT에 리포지토리와 개발 작업만 알려주면 됩니다. Luna는 리포지토리에서 자동으로 발견되고, 정상적인 작업은 chat 샌드박스에서 수행하며, 그 경로가 부족할 때만 GitHub Actions를 사용합니다.

## 빠른 시작

이 리포지토리가 문서화하는 ChatGPT Web 경로에서는 다음과 같이 설정합니다.

1. 이 리포지토리에서 **Use this template → Create a new repository**를 선택합니다.
2. ChatGPT에서 <https://chatgpt.com/plugins>의 Plugins Directory를 열고 **GitHub Plugin**을 설치/연결합니다.
3. GitHub에서 <https://github.com/apps/chatgpt-codex-connector>의 **ChatGPT Codex Connector**를 설치하고 새 리포지토리에 대한 access를 부여합니다. 이미 선택된 리포지토리만 허용하도록 설치되어 있다면 새 리포지토리를 access 목록에 추가합니다.
4. 일반 ChatGPT 대화에서 리포지토리 URL과 원하는 개발 작업을 알려줍니다. 사용자가 Luna Chat Coder를 별도로 언급할 필요는 없습니다. 리포지토리의 `AGENTS.md`가 모델을 내장 skill로 안내합니다.

이 template으로 만든 리포지토리에는 Luna가 이미 들어 있으므로 Luna를 별도로 설치할 필요는 없습니다.

1–3단계는 이 리포지토리가 검증하는 ChatGPT Web workflow에서 사용자가 한 번 수행하는 사용자/관리자 설정입니다. 리포지토리 생성, Plugin 연결, GitHub App 설치, 리포지토리 authorization이 끝난 뒤에 chat이 해당 리포지토리를 작업할 수 있다고 기대해야 합니다. Organization 정책에 따라 관리자 승인이 필요할 수 있습니다.

기존 리포지토리에 추가하려면 [기존 리포지토리에 추가](#기존-리포지토리에-추가)를 참고하십시오.

## Luna가 자동으로 하는 일

리포지토리 작업을 시작하면 모델은 조용히 다음을 수행해야 합니다.

1. `AGENTS.md`와 내장 `SKILL.md`를 읽습니다.
2. 정확한 리포지토리/PR/commit state를 식별하고 남아 있는 샌드박스 작업을 확인합니다.
3. edit 전에 정확한 target commit 또는 PR-head source 전체를 샌드박스 working tree로 materialize하고 base identity를 검증합니다.
4. 무엇을 설치하거나 가져오기 전에 샌드박스에 이미 있는 capability를 조사합니다.
5. 일반적인 edit/build/test/debug 작업은 샌드박스 작업 컨테이너에서 수행합니다.
6. supply, exact transport 또는 실제 degraded execution에 도움이 될 때만 bounded Actions mission을 사용합니다.
7. 리포지토리와 task가 요구하는 check로 실제 동작을 검증합니다.
8. 검증된 정확한 변경을 가장 단순하고 신뢰할 수 있는 GitHub 경로로 게시합니다.
9. 실제로 수행한 것만 보고하고, degraded remote execution이 필요했다면 그 사실을 알립니다.

정상적인 task에서는 사용자가 Luna를 조작하거나 내부 checklist를 지켜볼 필요가 없어야 합니다.

## Actions mission이 유용한 경우

GitHub Actions는 기본 개발환경이 아니라 bounded remote mechanism입니다.

대표적인 mission role 세 가지는 다음과 같습니다.

- **Supply** — 샌드박스가 engineering 작업 자체는 할 수 있지만 필요한 dependency/package set, 런타임, SDK, compiler, executable/application distribution, installer, native library, generated input 같은 외부 input을 구할 수 없는 경우.
- **Exact transport** — direct Git access가 없거나 비효율적일 때 정확한 repository source를 샌드박스로 가져오거나, 검증된 변경을 GitHub로 되돌릴 때 반복적인 content write보다 byte-preserving archive/patch/bundle이 더 안전하거나 효율적인 경우.
- **Degraded remote execution** — usage, duration, resource, execution hard limit 때문에 샌드박스 자체를 사용할 수 없거나 task를 지속할 수 없는 경우.

Transport는 양방향이며 최후의 벌칙 같은 경로가 아니라 하나의 선택지입니다. 작은 의도적 text edit은 direct file operation이 적합할 수 있지만, exact source나 검증된 변경이 이미 존재하고 재구성이 의미 있는 fidelity/partial-update risk를 추가한다면 가능하면 byte-preserving archive, checksummed artifact payload, Git object, patch, bundle을 선호합니다. Luna는 관찰된 payload에 대해 정확성과 신뢰성을 유지하면서 overhead가 가장 낮은 경로를 선택합니다.

성공으로 보고된 경우를 포함해 모든 mission result는 의존하기 전에 예상 output contract와 맞는지 확인해야 합니다. Green run 자체만으로 의도한 artifact, commit, ref, checksum 또는 source identity가 정확하다고 증명되지는 않습니다. 실패한 경우에는 source 수정이나 retry 전에 반환된 error, 실패한 step, log, partial result를 추가로 진단하며, 근거 없는 반복 실행은 명시적으로 피합니다.

상세한 mission 규칙은 [`actions-missions.md`](.agents/skills/luna-chat-coder/references/actions-missions.md)에 있습니다.

## 왜 필요한가

Chat 기반 development에는 이미 유용한 실행환경이 있습니다. Luna의 목적은 그것을 최대한 활용하면서도 영구적인 개발자 workstation인 것처럼 가정하지 않는 것입니다.

- 샌드박스는 reset되거나 사라질 수 있습니다.
- network, storage, resource, duration, usage limit 때문에 필요한 작업이 막힐 수 있습니다.
- 리포지토리가 처음부터 없는 tool 또는 외부 input을 요구할 수 있습니다.
- conversation text는 intent를 보존하기에는 좋지만 정확한 source byte의 저장소로는 부적합합니다.
- GitHub Actions는 workflow, startup, artifact, cleanup overhead가 있는 remote metered execution입니다.

따라서 정책은 **sandbox first; supply/transport에는 bounded remote help를 쓰되 engineering execution 자체는 샌드박스가 지속할 수 없을 때만 remote로 옮긴다**입니다. 새로운 것을 가져오기 전에 이미 있는 capability부터 조사합니다.

Engineering 방법은 Luna가 아니라 리포지토리가 정의합니다. 단지 실행하기 쉽다는 이유로 database, test framework, 런타임 또는 대체 기술을 Luna가 임의로 선택하지 않습니다. 리포지토리가 선언한 요구사항을 가능한 한 충실하게 실행할 수 있도록 돕습니다.

사용자의 컴퓨터는 의도적으로 workflow 밖에 있습니다. 일반적인 리포지토리 개발을 위해 사용자 host에 직접 접근하거나 host isolation을 약화시킬 필요가 없어야 합니다.

## 정확한 게시와 복구

GitHub는 정확한 리포지토리 state의 영속적인 source입니다. 복구 우선순위는 다음과 같습니다.

```text
commit / PR head
    > immutable Git 또는 Actions artifact
    > 살아남은 sandbox working tree
    > conversation reconstruction
```

Source acquisition과 publication에서는 실제 payload와 관찰된 integration 신뢰성에 따라 connected repository operation, native Git object operation, archive/checksummed-artifact/patch/bundle transport 가운데 선택합니다. 상당한 변경은 예상 base SHA에 묶어야 합니다. Base가 이동했다면 새 state를 복구하고 의도적으로 rebase/merge하거나 payload를 다시 만듭니다.

Practical한 byte-preserving transport로 기존 byte를 운반할 수 있다면 exact source tree나 검증된 multi-file/binary payload를 prose 또는 model-authored complete-file/blob content에서 다시 만들지 않습니다.

Temporary mission state는 task-owned이고 bounded해야 하지만 cleanup은 recovery를 고려해야 합니다. 실패한 run, branch, artifact, log가 debugging, review, handoff, recovery 가치가 있는 동안은 보존합니다. Conversation context가 유실되었다면 낯선 remote object를 삭제하기 전에 durable GitHub evidence에서 ownership과 terminal state를 복원합니다.

상세 규칙은 [`recovery.md`](.agents/skills/luna-chat-coder/references/recovery.md)와 [`actions-missions.md`](.agents/skills/luna-chat-coder/references/actions-missions.md)를 참고하십시오.

## 설계 모델 (선택 사항)

Luna를 사용하는 데 이 모델을 알 필요는 없지만, 설계를 설명하면 다음과 같습니다.

```text
Chat
    intent와 interaction

Sandbox work container
    기본 disposable development workstation

GitHub
    정확하고 영속적인 repository state

GitHub Actions mission
    정상 경로가 부족할 때 사용하는 bounded remote capability, transport 또는 execution
```

Actions mission은 live remote terminal보다 무인 심해/우주 탐사선에 가깝습니다. 정확한 source identity, input, 목적, return contract를 주고 독립적으로 실행한 뒤 종료 후 durable result를 확인합니다.

## ChatGPT Web 경로에 GitHub 연결이 두 개인 이유

이 workflow에는 서로 다른 두 layer가 필요합니다.

1. **GitHub Plugin** — ChatGPT 쪽 workflow/tool capability입니다.
2. **ChatGPT Codex Connector GitHub App** — target 리포지토리에 access 권한을 부여받는 GitHub 쪽 installation입니다.

둘 다 target 리포지토리에 대해 사용할 수 있고 authorization되어 있어야 하며 서로 대체할 수 없습니다. Actions/workflow/log/artifact access는 실제 Actions mission이 필요할 때에만 추가로 필요합니다.

UI 이름은 바뀔 수 있습니다. 중요한 것은 capability와 authorization boundary입니다.

## 리포지토리 발견과 구조

Template은 discovery, runtime policy, operational detail, maintainer memory를 의도적으로 분리합니다.

```text
AGENTS.md
    -> 작은 repository entry point
    -> chat-based development 전에 skill을 읽도록 모델을 안내

.agents/skills/luna-chat-coder/
  SKILL.md
      -> canonical machine policy
  references/
    actions-missions.md
      -> mission, failure diagnosis, transport, lifecycle, cleanup 규칙
    recovery.md
      -> sandbox/chat/context loss 또는 source ambiguity 이후 복구 규칙
    design-rationale.md
      -> Luna 자체를 수정할 때만 읽는 maintainer design memory
```

의도한 패턴은 **discover early, activate late**입니다. Luna를 load했다고 Actions를 실행해야 하는 것은 아닙니다. 정상 경로에서는 skill이 거의 보이지 않아야 합니다.

`AGENTS.md`는 discovery accelerator이지 hard runtime dependency가 아닙니다. Downstream 프로젝트는 최상위 README나 AGENTS 내용을 자기 프로젝트에 맞게 교체할 수 있습니다. Skill directory는 그대로 보존하고, 가능하면 Luna의 짧은 entry-point instruction을 프로젝트 자체의 `AGENTS.md`에 merge하는 것을 권장합니다. Repo-local skill discovery를 지원하는 host라면 이 pointer 없이도 skill을 발견할 수 있지만, Luna는 모든 host가 그렇게 동작한다고 가정하지 않습니다.

Maintainer rationale을 skill directory 안에 둔 이유는 template 복제, top-level file 교체, 리포지토리 migration, 공개 전 history reset 이후에도 설계 맥락이 함께 살아남게 하기 위해서입니다. 일반 development task에서는 이 문서를 읽을 필요가 없습니다.

## 호환성 범위

내장 skill은 Agent Skills 구조를 따르며 core policy를 host-neutral하게 유지합니다. Agent Skills는 open cross-platform format입니다: <https://agentskills.io/>.

이 리포지토리가 완전히 문서화하는 integration path는 **ChatGPT Web + GitHub Plugin + ChatGPT Codex Connector**입니다. 다른 Agent Skills host도 실제로 동등한 code-execution 및 GitHub capability를 제공한다면 같은 sandbox-first, durable-state, exact-transport, bounded-mission policy를 사용할 수 있습니다.

Format compatibility 자체는 완전한 operational support를 보장하지 않습니다. Host는 자신에게 없는 repository write, Actions, log, artifact, credential access가 있다고 가정해서는 안 됩니다.

## 기존 리포지토리에 추가

다음 complete skill directory를 복사합니다.

```text
.agents/skills/luna-chat-coder/
```

그 다음 이 리포지토리의 `AGENTS.md`에 있는 짧은 Luna entry-point instruction을 target 리포지토리의 기존 agent instruction에 추가합니다. 프로젝트 자체의 engineering guidance는 유지하십시오. Luna는 그 guidance를 둘러싼 continuity/fallback layer이지 대체재가 아닙니다.

검증된 ChatGPT Web 경로에서는 chat에 작업을 요청하기 전에 GitHub Plugin을 연결하고 ChatGPT Codex Connector에 해당 리포지토리 access를 부여합니다.

## 범위

Luna Chat Coder가 다루는 범위는 다음과 같습니다.

- chat-based development를 위한 early repository-policy discovery
- sandbox 또는 conversation context loss 이후 exact recovery
- sandbox-first execution과 capability inventory
- 리포지토리가 요구하는 누락 input의 faithful acquisition
- 더 나은 byte-preserving path일 때 양방향 exact source/change transport
- bounded GitHub Actions mission과 failure diagnosis
- 샌드박스 자체를 사용할 수 없거나 required execution을 지속할 수 없을 때 degraded remote execution
- evidence-based completion reporting
- temporary mission-owned remote state의 recovery-aware cleanup

Protocol은 작게 유지합니다. constrained chat-based repository development에서 반복되는 실패를 실제로 예방하는 규칙만 추가합니다.
