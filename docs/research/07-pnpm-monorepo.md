---
format: report
target: A4
topic: pnpm workspace monorepo
---

# pnpm Workspace 모노레포 컨벤션

## 1. pnpm 핵심 차별점

pnpm은 npm/Yarn Classic의 **flat node_modules 호이스팅(flat hoisting)** 모델 대신, **content-addressable store** 와 **하드링크(hardlink) 및 심볼릭 링크(symlink)** 를 결합한 비평탄(non-flat) 구조로 설계된 패키지 매니저다. 동일한 의존성 파일은 디스크상의 단일 저장소에 한 번만 보관되고, 각 프로젝트의 `node_modules` 에는 그 파일들이 하드링크로 연결되므로 추가 디스크 공간을 거의 차지하지 않는다 ([pnpm.io/motivation](https://pnpm.io/motivation)). 의존성이 업데이트될 때 100개 파일 중 1개만 변했다면 store 에는 1개 파일만 추가되어 갱신 비용이 작다 ([pnpm.io/motivation](https://pnpm.io/motivation)).

속도 측면에서도 pnpm 은 의존성 해석(resolve) → 디렉터리 구조 계산 → 링킹(linking) 의 세 단계를 동시 진행해 npm 의 전통적 3단계 직렬 설치보다 빠르다 ([pnpm.io/motivation](https://pnpm.io/motivation)). 또한 루트 `node_modules` 에는 직접 의존성만 심볼릭 링크로 노출되고 전이 의존성은 `node_modules/.pnpm` 하위에 격리된 채 들어간다. 그래서 선언되지 않은 전이(transitive) 의존성에 우연히 접근하는 **유령 의존성(phantom dependency)** 문제를 구조적으로 차단한다 ([pnpm.io/motivation](https://pnpm.io/motivation)). 락파일은 `pnpm-lock.yaml` 단일 파일에 워크스페이스 전체의 정확한 의존 그래프가 기록되므로, 동일 커밋을 받은 어떤 머신에서도 비결정성 없이 같은 트리를 재현할 수 있다.

## 2. 워크스페이스 모델

pnpm 워크스페이스는 루트의 **`pnpm-workspace.yaml`** 파일로 선언한다. 이 파일에 `packages:` 키와 글롭(glob) 목록을 적으면 해당 디렉터리들이 단일 워크스페이스로 묶인다 ([pnpm.io/workspaces](https://pnpm.io/workspaces)).

```yaml
packages:
  - "packages/*"
  - "packages/adapters/*"
  - "server"
  - "ui"
  - "cli"
```

워크스페이스 내부 의존성은 **`workspace:` 프로토콜** 로 표기한다. 이 프로토콜이 쓰이면 pnpm 은 레지스트리가 아니라 로컬 워크스페이스 패키지에만 해석을 시도한다 ([pnpm.io/workspaces](https://pnpm.io/workspaces)).

```json
{
  "dependencies": {
    "foo": "workspace:*",
    "bar": "workspace:^",
    "zoo": "workspace:^1.5.0"
  }
}
```

`pnpm publish` 시점에 `workspace:*` 같은 표기는 실제 버전(예: `1.5.0`)으로 자동 치환되어 외부 사용자도 등록된 패키지처럼 사용할 수 있다 ([pnpm.io/workspaces](https://pnpm.io/workspaces)).

명령 실행 범위는 **`--filter`** 로 제어한다. 패키지 이름·경로·의존 그래프·변경 파일 등을 셀렉터로 쓸 수 있다 ([pnpm.io/filtering](https://pnpm.io/filtering)).

```bash
pnpm --filter "@scope/pkg" test           # 정확한 이름
pnpm --filter "@scope/*" build            # 패턴
pnpm --filter "foo..." test               # foo + 의존성 전체
pnpm --filter "...foo" test               # foo + foo 에 의존하는 것
pnpm --filter "./packages/**" lint        # 디렉터리 글롭
pnpm --filter "...[origin/master]" test   # 변경 파일 기준
pnpm --filter=!foo build                  # 제외
```

버전 일관성을 위해서는 두 가지 메커니즘이 있다. 첫째, **`overrides`** 는 의존 그래프 전체에서 특정 패키지(혹은 피어 의존성)의 버전을 강제한다. 최신 pnpm Settings 문서는 `pnpm-workspace.yaml` 에 두는 방식을 안내하지만 ([pnpm.io/settings#overrides](https://pnpm.io/settings#overrides)), pnpm 9 계열까지는 루트 `package.json` 의 `pnpm.overrides` 키에 두는 방식이 함께 통용되며, Paperclip 은 후자(`package.json` 의 `pnpm.overrides`) 를 사용한다.

```yaml
overrides:
  "foo": "^1.0.0"
  "qar@1>zoo": "2"
```

둘째, 비교적 새로 도입된 **catalogs** 는 워크스페이스 전반에서 자주 쓰는 의존성의 버전 범위를 한 곳에 정의해 각 `package.json` 이 `catalog:` 프로토콜로 참조하게 한다. 업그레이드는 yaml 한 군데만 수정하면 되고, `package.json` 들은 변경되지 않으므로 머지 충돌이 줄어든다 ([pnpm.io/catalogs](https://pnpm.io/catalogs)).

```yaml
catalog:
  react: ^18.3.1
  redux: ^5.0.1
```

```json
{ "dependencies": { "react": "catalog:", "redux": "catalog:" } }
```

## 3. 다른 모노레포 도구와의 비교

| 도구 | 역할 | 강점 | 약점 |
|---|---|---|---|
| pnpm workspace | 패키지 매니저 + 워크스페이스 링킹 | content-addressable store, 유령 의존성 차단, `workspace:` 프로토콜·필터·overrides·catalogs 내장 ([pnpm.io/motivation](https://pnpm.io/motivation), [pnpm.io/workspaces](https://pnpm.io/workspaces)) | 자체 태스크 그래프나 원격 캐시는 없음 — 빌드 오케스트레이션은 별도 도구 필요 |
| Turborepo | 빌드 캐시·태스크 그래프 | `turbo.json` 만으로 태스크 병렬화 + 원격 캐시, 패키지 매니저 무관(npm/yarn/pnpm 모두 지원) ([turborepo.dev/docs](https://turborepo.dev/docs)) | 의존성 해석·링킹은 하지 않음, pnpm 등 워크스페이스 위에 얹어 써야 함 |
| Nx | 풀스택 모노레포 도구 | 태스크 그래프·캐시·모듈 경계(boundary) 강제·플러그인·CI 자가복구 ([nx.dev/getting-started/intro](https://nx.dev/getting-started/intro)) | 도입 비용·러닝커브·자체 컨벤션이 강함 |
| Lerna | 레거시 (현재 Nx 산하) | 멀티 패키지 퍼블리싱 워크플로, "Powered by Nx" 로 캐시·태스크는 Nx 가 담당 ([lerna.js.org](https://lerna.js.org/)) | 단독 가치는 줄었고 실질적으로 Nx 의 일부 — 신규 프로젝트에서는 우선순위가 낮음 |

요약하면 pnpm workspace 는 **의존성 레이어** 를, Turborepo/Nx 는 **빌드·태스크 레이어** 를 담당한다. 작은 모노레포는 pnpm 워크스페이스만으로 충분하고, 빌드 시간이 병목이면 그 위에 Turborepo 를 얹거나 Nx 로 통합 관리하는 식의 조합이 일반적이다. npm workspaces 와 비교하면, npm 도 루트 `package.json` 의 `workspaces` 필드로 다중 패키지 링킹을 지원하지만 ([docs.npmjs.com — Workspaces](https://docs.npmjs.com/cli/v10/using-npm/workspaces)), `workspace:` 프로토콜·필터 셀렉터 문법·content-addressable store·catalogs 같은 기능은 pnpm 쪽이 더 풍부하다. 모노레포의 이점은 **원자적 리팩터링(atomic refactor)** 과 단일 락파일에 의한 버전 일관성이고, 비용은 빌드·CI 시간 증가, 워크스페이스 경계 무너질 때의 의존 그래프 폭주, 퍼블리싱 워크플로 복잡화다. 이 비용은 catalogs/overrides 로 버전 스큐(version skew)를 막고, Turborepo/Nx 로 영향 범위만 빌드/테스트하는 식으로 완화한다.

## 4. Paperclip과의 관계

Paperclip 의 루트 `pnpm-workspace.yaml` 은 `packages/*`, `packages/adapters/*`, `packages/plugins/*`, `packages/plugins/examples/*`, `server`, `ui`, `cli` 를 워크스페이스로 묶고, `!packages/plugins/sandbox-providers/**` 와 `!packages/plugins/examples/plugin-orchestration-smoke-example` 두 패턴으로 일부 디렉터리를 명시적으로 제외한다 (`pnpm-workspace.yaml:1-14`). 실제 `packages/` 디렉터리에는 `db`, `shared`, `adapter-utils`, `mcp-server`, `plugins`, `adapters/` 가 들어 있고, `adapters/` 아래에는 `acpx-local` · `claude-local` · `codex-local` · `cursor-cloud` · `cursor-local` · `gemini-local` · `openclaw-gateway` · `opencode-local` · `pi-local` 9 개 어댑터 패키지가 각각 별도로 존재한다. 즉 control plane(server), 프론트엔드(ui), DB 스키마(db), 공용 타입(shared), 외부 코딩 에이전트 어댑터(adapters/*), 어댑터 공통 유틸(adapter-utils), 사용자/내부 플러그인(plugins/*), MCP 서버 구현(mcp-server), 그리고 CLI 가 모두 동일한 락파일과 동일한 TypeScript 컴파일 그래프를 공유한다. 이런 구조 덕분에 개발자는 `pnpm --filter @paperclipai/server dev`, `pnpm --filter "./packages/adapters/*" build`, `pnpm --filter "...[origin/master]" test` 처럼 변경 영향 범위에 맞춰 dev/build/test 를 좁혀 실행할 수 있고 ([pnpm.io/filtering](https://pnpm.io/filtering)), 어댑터·플러그인 간 공통 의존성은 `workspace:*` 로 링크되어 server 의 인터페이스 변경이 모든 어댑터 패키지에 단일 PR 로 반영되는 원자적 리팩터링이 가능하다 ([pnpm.io/workspaces](https://pnpm.io/workspaces)). 한편 일부 sandbox-provider 플러그인은 `!packages/plugins/sandbox-providers/**` 로 워크스페이스에서 명시적으로 제외해, 루트 `pnpm-lock.yaml` 에 서드파티 의존성 변동이 번지는 것을 막는 패턴도 함께 쓰고 있다 — 같은 모노레포 안에 두면서도 락파일 영향 반경을 분리한 절충안이다.

## 5. 출처

- [pnpm — Motivation](https://pnpm.io/motivation)
- [pnpm — Workspaces](https://pnpm.io/workspaces)
- [pnpm — Filtering](https://pnpm.io/filtering)
- [pnpm — Catalogs](https://pnpm.io/catalogs)
- [pnpm — Settings: overrides](https://pnpm.io/settings#overrides)
- [npm Docs — Workspaces](https://docs.npmjs.com/cli/v10/using-npm/workspaces)
- [Turborepo Docs](https://turborepo.dev/docs)
- [Nx — Getting Started](https://nx.dev/getting-started/intro)
- [Lerna](https://lerna.js.org/)
- [Paperclip 레포 루트 `pnpm-workspace.yaml`](https://github.com/paperclipai/paperclip)
