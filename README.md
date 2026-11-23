# Journey 확장 SDK

Journey는 **폴더 하나**로 기능을 더할 수 있어요. 빌드 도구도, 계정도, 서명도 필요 없어요.

| 만들 것 | 하는 일 | 설명서 | 시작 템플릿 |
| --- | --- | --- | --- |
| 새 탭 위젯 | 새 탭 페이지에 카드로 보여요 (시계, 할 일, 환율…) | [newtab-widget.md](newtab-widget.md) | [templates/newtab-widget](templates/newtab-widget) |
| 라이브 폴더 출처 | 사이드바 라이브 폴더에 새 글을 채워요 (사이트, API…) | [live-folder.md](live-folder.md) | [templates/live-folder](templates/live-folder) |

두 가지 모두 같은 폴더 형식을 써요. `widget.json`에 `"type": "live"`가 있으면 라이브 폴더 출처, 없으면 새 탭 위젯이에요.

```
my-extension/
├─ widget.json   ← 이름 · 아이콘 · 설정 · 접속 허용 주소
└─ index.html    ← 실행되는 페이지 (js · css · 이미지를 더 넣어도 돼요)
```

## 시작하기

1. 템플릿 폴더를 복사하고 `widget.json`의 `id`와 `name`을 바꿔요.
2. Journey에 설치해요.
   - 새 탭 위젯: 새 탭 → **편집** → **위젯 추가** → **위젯 확장 설치…**
   - 라이브 폴더: 사이드바 › **새 라이브 폴더** → **확장 설치…**
3. `widget.json`이 있는 폴더를 고르면 끝이에요. 고친 뒤 같은 방법으로 다시 설치하면 덮어써요.

## 예제

| 폴더 | 종류 | 보여 주는 것 |
| --- | --- | --- |
| [examples/newtab/world-clock](examples/newtab/world-clock) | 새 탭 위젯 | 설정(`select`), 크기에 맞춰 바뀌는 배치, 네트워크 없음 |
| [examples/newtab/github-repo](examples/newtab/github-repo) | 새 탭 위젯 | `network`로 API 호출, 링크 열기, 불러오는 중·오류 표시 |
| [examples/live/hn-search](examples/live/hn-search) | 라이브 폴더 | JSON API, 설정값으로 검색, 필터 힌트 사용 |
| [examples/live/rss-feed](examples/live/rss-feed) | 라이브 폴더 | RSS/Atom XML을 `DOMParser`로 읽기 |
| [templates/newtab-widget](templates/newtab-widget) | 새 탭 위젯 | 데이터 저장(`save`) |
| [templates/live-folder](templates/live-folder) | 라이브 폴더 | 가장 작은 출처 (Lobsters JSON) |

Journey에 기본 포함된 확장(D-Day, 환율, GitHub 트렌딩, Lobsters)도 같은 폴더 형식으로 만들어졌어요.

## widget.json 공통 필드

| 필드 | 설명 |
| --- | --- |
| `id` (필수) | 소문자 · 숫자 · `.` · `-` · `_`, 2–64자, 첫 글자는 영문 소문자나 숫자. `me.clock`처럼 겹치지 않게 지어 주세요. 같은 `id`로 다시 설치하면 덮어써요 |
| `name` (필수) | 목록에 보이는 이름 |
| `description` | 목록에 보이는 한 줄 설명 |
| `version` | 자유 형식 (`1.0.0`) |
| `icon` | [Iconify](https://icon-sets.iconify.design) 이름. 예: `lucide:timer`, `ph:lightning` |
| `entry` | 시작 파일 (기본 `index.html`). `..`은 쓸 수 없어요 |
| `network` | 접속할 수 있는 **https origin** 목록. 예: `["https://api.github.com"]`. 경로나 `http://`는 무시돼요 |
| `settings` | 사용자가 정하는 값 (아래) |
| `type` | `"live"`면 라이브 폴더 출처. 없으면 새 탭 위젯 |

`settings` 항목: `key`, `label`, `type`(`text` · `number` · `date` · `select`), `default`, `placeholder`, `options`(`select`일 때). 값은 항상 **문자열**로 와요. 숫자는 `Number()`로 바꿔 쓰세요.

## 보안 모델

확장은 샌드박스 iframe(`journey-widget://<id>/`)에서 실행되고, Journey와는 `postMessage`로만 이야기해요.

- 할 수 있는 것: 자기 폴더의 파일과 인라인 스크립트 · 스타일, `network`에 적은 https 주소로의 접속, https · `data:` 이미지
- 할 수 없는 것: Journey의 탭 · 기록 · 비밀번호 · 설정 읽기, 다른 주소 접속, 폼 전송, 페이지 이동, 팝업, `localStorage` · 쿠키
