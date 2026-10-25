# 독서 기록 (readlog-app)

폰에서 책 감상을 짧게 남기고, 정리된 기록을 책별로 보고, 주간 복습지를 읽는 웹앱입니다.
`index.html` 한 파일로 동작하며 외부 라이브러리·서버·DB가 없습니다.

- 메모는 비공개 저장소 `readlog-data`의 `inbox/YYYYMMDD_HHMMSS.txt`로 저장됩니다.
- GitHub Actions가 메모를 정리해 `data/entries.jsonl`, `books/`, `reviews/`를 만들고, 이 앱은 그 결과를 읽어서 보여 줍니다.
- 아이디·저장소 이름·토큰은 코드에 들어 있지 않습니다. 처음 설정에서 입력한 값이 **그 기기의 브라우저(localStorage)에만** 저장되고, `api.github.com`으로만 전송됩니다.

## 1. 토큰 만들기 (처음 한 번)

GitHub → Settings → Developer settings → Personal access tokens → **Fine-grained tokens** → Generate new token

| 항목 | 값 |
| --- | --- |
| Repository access | Only select repositories → `readlog-data` 하나만 |
| Contents | Read and write (메모 저장, 기록 읽기) |
| Actions | Read and write ("지금 만들기" 버튼) |
| Metadata | Read-only (자동 선택됨) |

만료일은 짧게(예: 90일) 두고, 만료되면 새로 만들어 설정에 다시 넣으면 됩니다.

## 2. 처음 설정

1. 앱 주소를 엽니다 (예: `https://<아이디>.github.io/readlog-app/`).
2. GitHub 아이디, 저장소 이름(기본 `readlog-data`), 토큰을 넣고 **연결 확인 후 저장**.
3. 연결 확인(`GET /repos/{아이디}/{저장소}`)이 성공해야 저장됩니다.

설정은 오른쪽 위 ⚙에서 다시 열 수 있습니다. 토큰 칸을 비워 두고 저장하면 기존 토큰을 그대로 씁니다.
**이 기기에서 설정 지우기**를 누르면 토큰과 임시저장 메모가 이 기기에서 지워집니다.

### 홈 화면에 추가
- iPhone(Safari): 공유 → 홈 화면에 추가
- Android(Chrome): ⋮ → 홈 화면에 추가 / 앱 설치

## 3. 쓰는 법

### 기록 탭
- 위에 최근에 읽은 책 5권이 버튼으로 나옵니다. 누르면 그 책으로 선택되고, **✏️ 직접 입력**으로 새 책 제목을 넣을 수 있습니다. **선택 해제**하면 책 없이 메모만 저장합니다.
- 큰 입력칸에 감상을 적거나, 키보드의 마이크 버튼으로 말해서 넣습니다. 브라우저가 음성 인식을 지원하면 **🎤 받아쓰기** 버튼도 나옵니다.
- **저장**하면 `inbox/`에 txt 파일이 생깁니다. 책을 골랐으면 첫 줄이 `책: 제목`이 됩니다.
- 저장에 실패하면 입력한 내용은 그대로 남고 이유가 표시됩니다. 입력 중인 내용은 자동으로 임시저장되어 앱을 닫았다 열어도 남아 있습니다.
- **정리 대기**는 아직 정리되지 않은 메모 수, 아래에는 최근 정리된 기록 3개가 보입니다.

### 책 탭
- 책별 기록 수와 최근 날짜. 검색칸은 제목·메모·태그(문구, 질문 포함)를 찾습니다. 태그를 누르면 그 태그로 검색합니다.
- 책을 누르면 **기억할 문구 → 적용할 행동 → 날짜별 기록**(한 줄, 메모, 질문, 연결, 받아쓰기 원문 접기) 순서로 보입니다.
- 기억할 문구는 받아쓰기 요약에서 뽑은 것이라 책 원문과 다를 수 있습니다.

### 복습 탭
- **복습 카드**: 질문이 먼저 나옵니다. 떠올려 본 뒤 **내 기록 보기**로 책·날짜·메모를 확인하고, **다른 카드**로 넘깁니다.
- **주간 복습지**: `reviews/`에서 이름순으로 가장 최근 `.md`를 보여 줍니다.
- **지금 만들기**: `readlog.yml` 워크플로를 `mode: review`로 실행합니다. 몇 분 뒤 ↻(새로고침)를 누르면 새 복습지가 보입니다.

## 4. 배포 (GitHub Pages)

공개 저장소 `readlog-app`에 이 폴더의 파일을 올리고 Settings → Pages → Branch `main` / root 로 켭니다.

```
index.html  manifest.webmanifest  icon.svg  icon-192.png  icon-512.png  README.md
```

`file://`로 직접 열어도 대부분 동작하지만 manifest(홈 화면 아이콘)는 http(s)에서만 적용됩니다.
로컬에서 확인하려면:

```bash
python -m http.server 8000 --directory readlog-app
```

## 5. 보안 메모
- 화면에 보이는 모든 데이터는 escape 후 출력합니다. 주간 복습지 마크다운은 전체를 escape한 뒤 제목·목록·체크박스·인용·굵게·`<details>/<summary>`만 다시 변환합니다.
- Content-Security-Policy로 스크립트는 이 파일 안의 것만, 네트워크 연결은 `https://api.github.com`만 허용합니다.
- 토큰은 브라우저 저장소에 평문으로 남습니다. 공용 기기에서는 쓰지 말고, 기기를 잃어버리면 GitHub에서 토큰을 바로 삭제(Revoke)하세요.

## 6. 테스트

```bash
python -m pytest tests/test_app.py -q
```

Playwright로 GitHub API를 가짜 응답으로 대신해 390×844 / 1280×800, 라이트 / 다크에서 전체 흐름과 가로 스크롤, escape를 검사합니다. 스크린샷은 `tests/screenshots/`에 저장됩니다.
