# 새 탭 위젯 만들기

새 탭 페이지의 카드 하나가 위젯이에요. 공통 형식과 보안은 [README](README.md)를 먼저 보세요.

## widget.json

```json
{
  "id": "me.countdown",
  "name": "카운트다운",
  "description": "위젯 추가 목록에 보이는 설명",
  "version": "1.0.0",
  "icon": "lucide:timer",
  "size": { "cols": 4, "rows": 3 },
  "network": ["https://api.example.com"],
  "settings": [
    { "key": "title", "label": "이름", "type": "text", "default": "시험" },
    { "key": "date", "label": "날짜 (YYYY-MM-DD)", "type": "date", "default": "2026-12-25" }
  ]
}
```

| 필드 | 설명 |
| --- | --- |
| `size.cols` | 처음 너비. 새 탭은 가로 12칸이에요 (4 = 3분의 1) |
| `size.rows` | 처음 높이. 1칸 = 44px |
| `settings` | 위젯 메뉴(⋯)에 "○○ 바꾸기…"로 나타나요. 바뀔 때마다 `init`이 다시 와요 |

크기는 사용자가 바꿀 수 있어요. 고정 px 대신 `%` · flex · grid로 배치하세요.

## 메시지

```js
// 1) 준비되면 알려요 → Journey가 'init'을 보내요
parent.postMessage({ journey: 'ready' }, '*')

// 2) Journey → 위젯 (처음, 그리고 설정을 바꿀 때마다)
addEventListener('message', (e) => {
  const m = e.data
  if (m?.journey !== 'init') return
  m.theme     // { text, muted, accent }  지금 새 탭에 맞는 CSS 색
  m.settings  // widget.json settings 값 (기본값 포함, 모두 문자열)
  m.data      // 전에 'save'로 저장한 값, 없으면 null
})

// 3) 위젯 → Journey
parent.postMessage({ journey: 'save', data: { count: 3 } }, '*')          // 저장 (객체, 100KB 미만)
parent.postMessage({ journey: 'open', url: 'https://example.com' }, '*')  // 새 탭으로 열기 (https만)
```

| 방향 | 메시지 | 비고 |
| --- | --- | --- |
| 위젯 → Journey | `{ journey: 'ready' }` | 이걸 받아야 `init`이 와요 |
| Journey → 위젯 | `{ journey: 'init', theme, settings, data }` | 설정이 바뀔 때도 다시 와요 |
| 위젯 → Journey | `{ journey: 'save', data }` | 객체만, JSON으로 100KB 미만. 새 탭 설정과 함께 저장 · 백업돼요 |
| 위젯 → Journey | `{ journey: 'open', url }` | `https://`만 열려요 |

## 네트워크

새 탭 위젯은 `fetch`로 직접 접속해요. 두 가지를 확인하세요.

1. `network`에 그 origin(`https://도메인`, 경로 없이)이 있어야 해요. 없으면 CSP가 막아요.
2. 상대 서버가 CORS를 허용해야 해요. 위젯의 origin은 `journey-widget://<id>`예요.

CORS를 허용하지 않는 사이트를 읽어야 한다면 라이브 폴더 출처로 만드는 게 맞아요. 라이브 폴더는 Journey가 대신 받아 줘요.

## 테마와 디자인

```js
for (const [k, v] of Object.entries(m.theme)) document.documentElement.style.setProperty('--' + k, v)
```

```css
html, body { margin: 0; height: 100%; background: transparent; color: var(--text);
  font: 13px -apple-system, BlinkMacSystemFont, 'Apple SD Gothic Neo', sans-serif; }
.sub { color: var(--muted); }
.num { color: var(--accent); }
```

- 배경은 투명하게 두세요. 카드와 흐림은 Journey가 그려요
- 색은 `--text` · `--muted` · `--accent`만 쓰면 어떤 테마와 배경 이미지에서도 잘 보여요
- 모서리 8–12px, 움직임은 `cubic-bezier(.32,.72,0,1)`
- 계속 도는 애니메이션은 피하세요. 화면 전체를 매 프레임 다시 그리게 해서 배터리를 써요

## 예제

- [examples/newtab/world-clock](examples/newtab/world-clock): 설정과 반응형 배치
- [examples/newtab/github-repo](examples/newtab/github-repo): API 호출과 링크 열기
- [templates/newtab-widget](templates/newtab-widget): 데이터 저장
- Journey 기본 위젯(D-Day, 환율)도 같은 폴더 형식으로 만들어졌어요.

## 문제 해결

| 증상 | 확인할 것 |
| --- | --- |
| 화면이 비어 있어요 | `entry` 파일 이름, `widget.json` 문법 |
| 색이 이상해요 | `ready`를 보냈는지, `init`에서 `theme`를 적용했는지 |
| `fetch`가 실패해요 | `network`의 origin, 상대 서버의 CORS |
| 저장이 안 돼요 | `data`가 객체인지, 100KB 미만인지 |
| 링크가 안 열려요 | `https://`로 시작하는지 |
