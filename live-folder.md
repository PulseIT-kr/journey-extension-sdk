# 라이브 폴더 출처 만들기

라이브 폴더 출처는 화면에 보이지 않는 샌드박스에서 실행돼요. 항목 목록만 돌려주면 Journey가 사이드바 폴더에 쌓고, 새 글이 오면 알려 줘요. 공통 형식과 보안은 [README](README.md)를 먼저 보세요.

## widget.json

```jsonc
{
  "id": "me.lobsters",
  "type": "live",                         // 필수: 라이브 폴더 출처
  "name": "Lobsters",
  "description": "새 라이브 폴더 목록에 보이는 설명",
  "icon": "ph:link-simple",
  "network": ["https://lobste.rs"],       // fetch-url로 받을 수 있는 곳
  "settings": [                           // 라이브 폴더를 만들 때 묻는 값
    { "key": "list", "label": "목록", "type": "select", "options": ["hottest", "newest"] }
  ],
  "filters": { "limit": 15 }              // 새 폴더의 처음 필터 (사용자가 바꿀 수 있어요)
}
```

| 필드 | 설명 |
| --- | --- |
| `"type": "live"` | 필수. 없으면 새 탭 위젯으로 설치돼요 |
| `settings` | 새 라이브 폴더 화면에 입력칸으로 나와요. `select`는 고르는 칸이 돼요. `placeholder`는 "예: …"로 보여요 |
| `filters` | 새 폴더의 처음 필터. `limit`, `maxAgeDays`, `include`, `exclude` |

## 흐름

```
출처 ──ready──────────────▶ Journey
출처 ◀──fetch {settings, filter}── Journey
출처 ──fetch-url {id, url}─▶ Journey   (필요한 만큼 반복)
출처 ◀──fetch-result {id, status, body, type}── Journey
출처 ──entries {items}──────▶ Journey   (또는 error {message})
```

| 방향 | 메시지 | 비고 |
| --- | --- | --- |
| 출처 → Journey | `{ journey: 'ready' }` | 이걸 받으면 `fetch`가 와요 |
| Journey → 출처 | `{ journey: 'fetch', settings, filter }` | `settings`: 만들 때 고른 값. `filter`: 사용자가 정한 필터 (정한 것만) |
| 출처 → Journey | `{ journey: 'fetch-url', id, url }` | Journey가 대신 받아요. `network`의 https origin만 돼요 |
| Journey → 출처 | `{ journey: 'fetch-result', id, status, body, type }` | `body`는 문자열. 실패하면 `{ id, error }` |
| 출처 → Journey | `{ journey: 'entries', items }` | 끝. 한 번에 최대 50개 |
| 출처 → Journey | `{ journey: 'error', message }` | 사용자에게 그대로 보여요 |

새로고침할 때마다 출처 페이지가 새로 열리고, 답을 보내면 닫혀요. 30초 안에 `entries`나 `error`를 보내지 않으면 실패로 처리돼요.

`fetch-url`은 Journey가 대신 요청해서 CORS를 신경 쓰지 않아도 돼요. 응답은 3MB까지, 리디렉션은 5번까지이고 리디렉션 주소도 `network` 목록으로 확인해요.

## 항목

| 필드 | 필수 | 설명 |
| --- | --- | --- |
| `url` | 예 | `http(s)://` 주소. 없으면 그 항목은 빠져요 |
| `title` | | 300자까지. 없으면 `url` |
| `subtitle` | | 작은 글씨 한 줄 (점수, 댓글 수…), 200자까지. 키워드 필터가 여기도 봐요 |
| `date` | | ISO 문자열이나 밀리초 숫자. 기간 필터에 쓰여요 |
| `id` | | 바뀌지 않는 값. 없으면 `url`. 읽음 표시가 이걸로 유지돼요 |

항목은 **최신순**으로 보내 주세요. 개수 필터가 앞에서부터 남겨요.

## 필터

사용자가 정하는 필터는 Journey가 받은 항목에 직접 적용해요. 출처가 구현할 필요는 없어요. 순서는 키워드 → 기간 → 개수예요.

| 키 | 예 | 동작 |
| --- | --- | --- |
| `limit` | `10` | 앞에서부터 이 개수만 남겨요 |
| `maxAgeDays` | `7` | `date`가 이보다 오래된 항목을 빼요 (`date` 없는 항목은 남아요) |
| `include` | `"rust, wasm"` | 제목이나 `subtitle`에 이 단어가 하나라도 있는 것만 (대소문자 무시) |
| `exclude` | `"광고, sponsored"` | 이 단어가 있으면 빼요 |

`fetch`의 `filter`는 힌트예요. 이렇게 쓰면 좋아요.

- API에 개수 · 날짜 조건이 있으면 넘겨서 덜 받기
- 키워드 필터로 줄어들 수 있으니 `limit`보다 넉넉히 보내기 (예: 2배, 최대 50)
- 기간 필터가 동작하도록 `date` 채우기

## 웹 페이지 · XML 읽기

`fetch-result.body`는 문자열이에요.

```js
const doc = new DOMParser().parseFromString(body, 'text/html')       // HTML 페이지
const xml = new DOMParser().parseFromString(body, 'application/xml') // RSS · Atom
```

## 예제

- [examples/live/hn-search](examples/live/hn-search): JSON API와 설정값 검색
- [examples/live/rss-feed](examples/live/rss-feed): RSS/Atom XML 읽기
- [templates/live-folder](templates/live-folder): 가장 작은 출처
- Journey 기본 포함된 GitHub 트렌딩 라이브 폴더도 같은 폴더 형식으로 만들어졌어요.

## 문제 해결

| 증상 | 확인할 것 |
| --- | --- |
| "30초 안에 답하지 않았어요" | `ready`를 보냈는지, 오류가 나도 `error`를 보내는지 |
| "network 목록에 없어요" | `network`의 origin, 리디렉션되는 주소의 origin |
| 항목이 하나도 안 보여요 | 항목마다 `url`이 있는지, 필터가 다 걸러 내지 않았는지 |
| 읽음 표시가 계속 풀려요 | `id`가 새로고침해도 같은지 |
