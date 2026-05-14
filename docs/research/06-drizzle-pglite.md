---
format: report
target: A4
topic: Drizzle ORM + Embedded PostgreSQL
---

# Drizzle ORM 와 임베디드 PostgreSQL

## 1. Drizzle ORM 개요

**Drizzle ORM** 은 TypeScript 로 작성된 경량 데이터베이스 툴킷으로, 스스로를 "헤드리스(headless) TypeScript ORM" 이라 소개한다. 의존성이 0이고 약 31KB 수준의 번들 크기로 서버리스 환경을 1차 타깃으로 설계되었으며, **SQL 친화적인 쿼리 빌더(SQL-like query builder)** 와 **컴파일 타임 타입 안정성(type safety)** 을 동시에 제공한다 ([orm.drizzle.team/docs/overview](https://orm.drizzle.team/docs/overview)). 스키마는 TypeScript 파일로 선언하는 **스키마 우선(schema-first)** 방식을 따르며, 동반 CLI 인 **drizzle-kit** 의 핵심 마이그레이션 명령은 4 개다: `drizzle-kit generate` (스키마 diff 로 SQL 마이그레이션 파일 생성), `drizzle-kit migrate` (생성된 파일 적용), `drizzle-kit push` (파일 없이 스키마를 직접 동기화, 프로토타이핑용), `drizzle-kit pull` (기존 DB 스키마를 TS 로 introspect). 별개로 `drizzle-kit studio` 는 시각적 DB 관리 UI 명령이다 ([orm.drizzle.team/docs/migrations](https://orm.drizzle.team/docs/migrations), [Drizzle Kit Studio](https://orm.drizzle.team/docs/drizzle-kit-studio)).

## 2. Drizzle vs Prisma 비교 표

| 항목 | Drizzle | Prisma |
|---|---|---|
| 스키마 정의 | TypeScript 코드 ([drizzle](https://orm.drizzle.team/docs/overview)) | 자체 DSL `schema.prisma` ([prisma.io](https://www.prisma.io/docs/orm/prisma-schema)) |
| 쿼리 API | SQL 그대로(builder) + 관계형 API | 메서드 체인 추상화(`findMany` 등) |
| 의존성/크기 | 0 deps, ~31KB ([drizzle](https://orm.drizzle.team/docs/overview)) | 런타임 + Rust query engine 바이너리 동반 |
| 런타임 모델 | 라이브러리 그대로 호출 | 별도 query engine 프로세스/바이너리 |
| 엣지/서버리스 | "serverless-ready by design" ([drizzle](https://orm.drizzle.team/docs/overview)) | 엔진 바이너리 때문에 콜드스타트 부담 |
| 마이그레이션 | `drizzle-kit generate/migrate/push/pull` + 별도 `studio` UI ([drizzle](https://orm.drizzle.team/docs/migrations)) | `prisma migrate dev/deploy` |
| 학습 곡선 | SQL 알면 거의 그대로 | 자체 DSL 학습 필요 |
| 호환성 | Prisma 와 병행 사용 가능 (`drizzle-prisma-generator`) ([drizzle/prisma](https://orm.drizzle.team/docs/prisma)) | — |

요약하면 Prisma 가 SQL 을 추상화한다면 Drizzle 은 SQL 을 그대로 노출하면서 타입만 입힌다는 철학 차이로 정리된다 ([orm.drizzle.team/docs/overview](https://orm.drizzle.team/docs/overview)).

## 3. 임베디드 PostgreSQL 옵션

- **PGlite (electric-sql/pglite)**: WebAssembly 로 컴파일된 PostgreSQL 로, 약 **3MB gzipped** 크기로 브라우저, Node.js, Bun, Deno 안에서 별도 서버 없이 동작한다. 인메모리, 파일 시스템, 브라우저의 IndexedDB 에 영속화할 수 있고 pgvector, PostGIS 같은 확장도 지원한다. WASM 이 프로세스 fork 를 못 하기 때문에 **PostgreSQL single-user mode** 로 실행되어 동시에 하나의 연결만 처리할 수 있다는 근본적 제약이 있다 ([github.com/electric-sql/pglite](https://github.com/electric-sql/pglite), [pglite.dev](https://pglite.dev/)).

- **embedded-postgres (npm, leinelissen)**: WASM 이 아니라 **네이티브 PostgreSQL 바이너리**(zonky/embedded-postgres-binaries 기반)를 플랫폼별 옵셔널 의존성(`@embedded-postgres/{linux-x64, darwin-arm64, windows-x64, ...}`)으로 받아 Node 프로세스에서 클러스터를 `initialise()` → `start()` 시키는 방식이다. macOS(x64/arm64), Windows(x64), Linux(x64/arm/arm64/ia32/ppc64) 를 지원하며 PostgreSQL 14\~18 버전을 npm 태그로 선택할 수 있다. 즉 **풀 PostgreSQL 인스턴스**가 동시 연결, 확장, 도구 모두 그대로 동작한다 ([github.com/leinelissen/embedded-postgres](https://github.com/leinelissen/embedded-postgres)). Paperclip 의 `packages/db/package.json` 은 이 패키지를 `embedded-postgres@^18.1.0-beta.16` 으로 핀하여 dev 모드 DB 로 사용한다.

- **호스티드 PostgreSQL (Supabase, Neon, RDS 등)**: 운영(production) 환경용. 매니지드 백업, 다중 연결, 풀러, 리전 복제 등을 제공한다. Drizzle 은 PostgreSQL 호환 드라이버이면 동일 스키마 코드로 그대로 붙는다 ([orm.drizzle.team/docs/get-started-postgresql](https://orm.drizzle.team/docs/get-started-postgresql)).

세 옵션의 트레이드오프는 단순하다: PGlite 는 가장 가볍지만 단일 연결과 single-user 제약이 있고, embedded-postgres 는 진짜 Postgres 라 호환성은 100% 인 대신 플랫폼별 네이티브 바이너리를 깔아야 하며, 호스티드는 운영 안정성을 사지만 네트워크 의존이 생긴다.

## 4. Paperclip 과의 관계

Paperclip 의 `packages/db/` 는 Drizzle ORM 으로 **86 개의 `pgTable()` 정의를 78 개 스키마 파일** 에 선언한다 (Ch02 합의 사실). `drizzle.config.ts` 는 Drizzle Kit 설정에서 `process.env.DATABASE_URL` 을 읽어 `generate` 등을 처리하지만 (`packages/db/drizzle.config.ts`), `pnpm db:migrate` 의 실제 연결 결정은 `packages/db/src/migrate.ts` 가 호출하는 `resolveMigrationConnection()` 이 담당해 `DATABASE_URL` → `.paperclip` env/config → embedded-postgres fallback 순서로 해석한다 (`packages/db/src/migration-runtime.ts:183-193`). 개발 모드에서는 `embedded-postgres@^18.1.0-beta.16` 로 로컬 PostgreSQL 클러스터를 프로세스 안에서 띄우고, 운영에서는 동일한 `DATABASE_URL` 만 Supabase 등 호스티드 Postgres 로 바꾸면 같은 마이그레이션과 같은 쿼리 코드가 그대로 돈다. 즉 Drizzle 의 **DB-호환 드라이버 추상화** + embedded-postgres 의 **네이티브 dev 인스턴스** 조합이 fallback 체인 한 곳만 손대면 dev↔prod 전환을 가능하게 만든다.

## 5. 출처

- [Drizzle ORM Overview](https://orm.drizzle.team/docs/overview)
- [Drizzle Migrations](https://orm.drizzle.team/docs/migrations)
- [Drizzle Get Started: PostgreSQL](https://orm.drizzle.team/docs/get-started-postgresql)
- [Drizzle x Prisma extension](https://orm.drizzle.team/docs/prisma)
- [Prisma Schema docs](https://www.prisma.io/docs/orm/prisma-schema)
- [PGlite GitHub (electric-sql/pglite)](https://github.com/electric-sql/pglite)
- [PGlite official site](https://pglite.dev/)
- [embedded-postgres GitHub (leinelissen)](https://github.com/leinelissen/embedded-postgres)
- [embedded-postgres on npm](https://www.npmjs.com/package/embedded-postgres)
- [@embedded-postgres/linux-x64](https://www.npmjs.com/package/@embedded-postgres/linux-x64)
- [zonky embedded-postgres-binaries](https://github.com/zonkyio/embedded-postgres-binaries)
- Paperclip `packages/db/package.json`, `packages/db/drizzle.config.ts`
