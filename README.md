# Night Radio Station

잠들지 못한 도시를 위해 심야 라디오 방송국을 운영하는 스토리 중심 방치형 데스크톱 게임입니다.

## 빠른 시작

```bash
yarn install
yarn dev
```

데스크톱 앱 실행:

```bash
yarn tauri dev
```

Tauri 데스크톱 개발에는 Rust toolchain이 필요합니다.

## 데스크톱 앱 기준

- 기본 창 크기는 `420x640`으로, 화면 한쪽에 작게 띄워두는 방치형 앱을 기준으로 합니다.
- 최소 창 크기는 `360x520`으로 제한해 핵심 UI가 무너지지 않게 합니다.
- `main`/`develop`에는 직접 커밋하지 않고 `feature/*` 브랜치에서 PR로 병합합니다.

## 명령어

| 명령어 | 설명 |
| --- | --- |
| `yarn dev` | SvelteKit 개발 서버를 시작합니다. |
| `yarn tauri dev` | Tauri 데스크톱 앱을 시작합니다. |
| `yarn run check` | Svelte 타입 검사를 실행합니다. |
| `yarn build` | 정적 프론트엔드를 빌드합니다. |
| `yarn preview` | 프로덕션 프론트엔드 빌드를 미리 봅니다. |

## 현재 MVP 루프

- 앱이 열려 있는 동안 방송이 계속 송출됩니다.
- 시간이 지나면 청취자 사연이 도착합니다.
- 신호, 청취자, 평판, 이야기 조각이 방치 플레이로 증가합니다.
- 안테나와 송신기 업그레이드로 방송국 진행이 강화됩니다.
- 첫 slice에서는 브라우저 localStorage에 진행 상태를 저장합니다.
