---
format: report
target: A4
topic: React + Vite + shadcn/ui
---

# React · Vite · shadcn/ui 제어 보드 UI 스택

## 1. Vite 개요

**Vite** 는 네이티브 ES 모듈(ESM) 기반 dev server 와 Rollup(현재 Rolldown 으로 통합 중) 기반 production build 를 결합한 차세대 프런트엔드 툴체인이다. 전통적인 번들러 dev server 가 "브라우저에 무엇이라도 보여주기 전에 애플리케이션 전체를 미리 처리(process your entire application upfront)" 해야 했던 것과 달리, Vite 는 의존성만 한 번 사전 번들링(pre-bundle)하고 소스 코드는 브라우저가 요청할 때마다 ESM 으로 즉시 변환해 내려보낸다. 그 결과 "dev server 시작이 애플리케이션 크기에 무관하게 거의 즉시(nearly instant, regardless of application size)" 이뤄진다 ([vite.dev/guide/why](https://vite.dev/guide/why.html)).

**HMR(Hot Module Replacement)** 은 ESM 채널 위에서 동작한다. "파일을 편집하면 Vite 는 네이티브 ESM 위로 HMR 을 사용해 해당 모듈만 갱신하고, 풀 페이지 리로드나 리빌드 대기를 하지 않는다(update just that module in the browser, without a full page reload or waiting for a rebuild)" ([vite.dev/guide/why](https://vite.dev/guide/why.html)). 프로덕션 빌드는 여전히 번들이 필요하다 — 중첩 import 의 네트워크 라운드트립을 줄이기 위해서다 — 그래서 Rollup(또는 Rust 기반 후속 Rolldown)이 빌드 단계에 쓰인다 ([vite.dev/guide/why](https://vite.dev/guide/why.html)).

**Vite middleware mode** 는 Vite 를 독립 dev server 가 아니라 Express·Koa 같은 기존 Node.js 서버의 미들웨어로 끼워 넣는 모드다. `server.middlewareMode: true` 로 활성화하면 Vite 가 standalone HTTP 서버를 띄우지 않고 부모 서버의 라우터 체인에 합류한다 ([vite.dev/config/server-options](https://vite.dev/config/server-options)). `ViteDevServer` 의 `httpServer` 는 "middleware mode 에서는 null 이 되며(Will be null in middleware mode)" WebSocket 프록시를 위해 부모 HTTP 서버 인스턴스를 넘겨주는 것이 권장된다 ([vite.dev/guide/api-javascript](https://vite.dev/guide/api-javascript)).

## 2. shadcn/ui 의 분배 모델

**shadcn/ui** 는 npm 의존성으로 추가하는 컴포넌트 라이브러리가 아니다. 공식 소개는 단호하다 — "이것은 컴포넌트 라이브러리가 아니다. 당신의 컴포넌트 라이브러리를 짓는 방식이다(This is not a component library. It is how you build your component library)" ([ui.shadcn.com/docs](https://ui.shadcn.com/docs)). 사용자는 CLI 로 컴포넌트의 **소스 코드 자체** 를 자기 레포에 복사해 가며, 이후 그 코드는 자기 코드처럼 자유롭게 수정한다.

다섯 가지 핵심 원칙이 이 모델을 떠받친다 — **Open Code**(컴포넌트가 블랙박스가 아닌 가독 가능한 소스로 존재), **Composition**(모든 컴포넌트가 일관된 인터페이스 공유), **Distribution**(플랫 파일 스키마와 CLI 로 프로젝트 간 공유), **Beautiful Defaults**(Tailwind 기반의 기본 스타일링), **AI-Ready**(LLM 이 읽고 개선 가능한 오픈 코드) ([ui.shadcn.com/docs](https://ui.shadcn.com/docs)). 기술 스택은 접근성 있는 무스타일 프리미티브인 **Radix UI** 위에 **Tailwind CSS** 로 스타일을 입힌 형태다 ([ui.shadcn.com/docs](https://ui.shadcn.com/docs)).

**registry** 개념은 이 분배 모델을 일반화한다. "shadcn CLI 로 자체 코드 레지스트리를 운영해 커스텀 컴포넌트·훅·페이지·설정·룰·기타 파일을 어떤 프로젝트에든 배포할 수 있다(distribute your custom components, hooks, pages, config, rules and other files to any project)" ([ui.shadcn.com/docs/registry](https://ui.shadcn.com/docs/registry)). `registry.json` 과 `registry-item.json` 스키마가 메타데이터를 정의하며, 레지스트리는 React 에 한정되지 않고 어떤 프레임워크에도 작동한다 ([ui.shadcn.com/docs/registry](https://ui.shadcn.com/docs/registry)).

## 3. TanStack Query 와 서버 상태

**TanStack Query**(구 React Query)는 자신을 "웹 애플리케이션을 위한 빠진(missing) 데이터-페칭 라이브러리" 로 규정하며 "fetching, caching, synchronizing, updating server state" 를 담당한다 ([tanstack.com/query](https://tanstack.com/query/latest/docs/framework/react/overview)). 핵심 통찰은 **서버 상태(server state)** 가 클라이언트 상태와 본질적으로 다르다는 점이다 — 원격에 영속되고, 비동기 API 가 필요하며, 소유권이 공유되고, 모르는 사이에 낡을 수 있다. 그래서 전통적 상태관리 라이브러리는 "비동기·서버 상태 처리에 그다지 강하지 않다(not so great at working with async or server state)" ([tanstack.com/query](https://tanstack.com/query/latest/docs/framework/react/overview)).

기본 동작은 신선도 추적과 자동 재요청에 맞춰져 있다. 캐시된 데이터는 기본적으로 즉시 stale 로 간주되며, 세 시점에 자동 refetch 가 일어난다 — 새로운 query 인스턴스가 mount 될 때, **윈도우 포커스 복귀 시(refetchOnWindowFocus)**, 네트워크 재접속 시(refetchOnReconnect) ([tanstack.com/query 기본값 가이드](https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults)). `staleTime` 으로 신선 윈도우를 늘리거나, `refetchInterval` 로 폴링을 추가하거나, `queryClient.invalidateQueries()` 로 mutation 직후 명시적 무효화를 거는 식으로 갱신 정책을 세분화할 수 있다 ([tanstack.com/query 기본값 가이드](https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults)).

자주 갱신되는 agent dashboard — 작업 큐, 실행 중인 task 의 진행률, 코멘트 스레드 — 처럼 서버 측 상태가 끊임없이 변하는 UI 에 이 모델이 적합한 이유다. 각 화면은 `useQuery` 로 자기 데이터의 키만 선언하면 되고, 캐시 무효화·중복 제거·백그라운드 재요청은 라이브러리가 처리한다. 그 결과 컴포넌트 코드에서 `useState` + `useEffect` + 직접 fetch 로 짠 수십 줄의 보일러플레이트가 사라진다 — 공식 문서가 표현하듯 "복잡하고 오해되기 쉬운 많은 줄(many lines of complicated and misunderstood code)" 을 최소한의 query 로직으로 바꿔 준다 ([tanstack.com/query](https://tanstack.com/query/latest/docs/framework/react/overview)).

## 4. Single-origin dev (Express + Vite middleware)

전형적인 SPA 개발 셋업은 두 origin 을 띄운다 — Vite dev server 가 5173 포트에, API 서버가 다른 포트에. 그러면 브라우저는 cross-origin 요청을 보내야 하므로 **CORS** 설정이 필요하고, 쿠키·세션 인증은 SameSite·credentials 옵션이 얽히면서 디버깅이 까다로워진다.

**Vite middleware mode** 를 쓰면 이 문제가 사라진다. Express 가 API 라우트(`/api/*`)와 Vite 미들웨어를 같은 인스턴스에서 처리하므로, 브라우저 입장에서는 SPA 와 API 가 모두 단일 origin (`localhost:3100`) 에 있다 ([vite.dev/config/server-options](https://vite.dev/config/server-options)). CORS preflight 가 사라지고, 쿠키 기반 인증이 추가 설정 없이 동작하며, dev 와 production 사이에 프록시 룰을 따로 유지할 필요가 없다 ([vite.dev/guide/api-javascript](https://vite.dev/guide/api-javascript)). 즉, 프로덕션 배포물 — Express 가 빌드된 정적 자산을 서빙하는 형태 — 와 dev-time 의 요청 경로가 사실상 동일해져, **dev 와 production 의 동형성(isomorphism)** 이 확보된다.

같은 패턴은 SSR 시나리오에서 잘 알려져 있다. Vite 공식 가이드는 SSR dev server 셋업 예제로 `createViteServer({ server: { middlewareMode: true } })` 를 Express 에 `app.use(vite.middlewares)` 로 끼워 넣는 코드를 보여 준다 ([vite.dev/guide/api-javascript](https://vite.dev/guide/api-javascript)). 제어 보드처럼 SSR 까지는 필요 없고 SPA 만 띄우면 되는 경우에도, 같은 미들웨어 합류 패턴으로 동일 origin 효과를 얻을 수 있다.

## 5. Paperclip 과의 관계

Paperclip 의 `ui/` 폴더는 이 스택을 그대로 구현한다 — `ui/src/pages/` 에 라우트 단위 화면, `ui/src/components/` 에 shadcn/ui 기반 UI 컴포넌트, `ui/src/api/` 에 `/api` fetch 래퍼 (`ui/src/api/client.ts:22-25` 의 `credentials: "include"` 포함) 가 들어 있다. TanStack Query 훅과 `useQuery` / `useMutation` 호출은 `ui/src/context/` (예: `CompanyContext`, `LiveUpdatesProvider`) 와 `ui/src/pages/` · `ui/src/components/` 에 분산돼 있고, `ui/src/hooks/` 는 보조 클라이언트 훅을 담는다. 빌드 설정은 `ui/vite.config.ts` 한 파일에 모이고, Express 서버가 dev 모드에서 `vite.middlewareMode` 로 SPA 를 마운트해 **3100 번 포트 단일 origin** 에서 API 와 보드 UI 를 동시에 서빙한다 — 그래서 CORS 설정 없이 쿠키 기반 세션 요청이 단순해진다 — 별도의 CSRF 토큰 방어는 현재 코드 베이스에서 발견되지 않으며, Better Auth 의 쿠키·trusted origin 설정 (`server/src/auth/better-auth.ts:40-44`, `:64-90`) 이 그 자리를 맡는다. 컴포넌트 단위 검토용 **Storybook** 은 `ui/storybook/` 에 별도로 격리돼 있어 본 앱의 Vite 파이프라인에 간섭하지 않는다.

## 6. 출처

- [Vite — Why Vite](https://vite.dev/guide/why.html)
- [Vite — JavaScript API (middleware mode)](https://vite.dev/guide/api-javascript)
- [Vite — server.middlewareMode option](https://vite.dev/config/server-options)
- [shadcn/ui — Introduction](https://ui.shadcn.com/docs)
- [shadcn/ui — Registry](https://ui.shadcn.com/docs/registry)
- [TanStack Query — Overview](https://tanstack.com/query/latest/docs/framework/react/overview)
- [TanStack Query — Important Defaults](https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults)
