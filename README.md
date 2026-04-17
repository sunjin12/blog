# Blog

Hugo + PaperMod 기반 개인 블로그. GitHub Pages 에 배포되어 `https://sunjin12.github.io/blog/` 로 서비스됩니다.

## 1단계: 최초 배포 (글 작성 전용)

아직 이 폴더는 원격 저장소에 연결되어 있지 않습니다. 아래 순서대로 진행하세요.

### 1. GitHub 에서 빈 리포 생성
- Repository name: **`blog`**
- Visibility: **Public** (GitHub Pages 무료 플랜 조건)
- README/license/gitignore 모두 체크 해제 (빈 리포)

### 2. 로컬에서 Git 초기화 + PaperMod 테마 추가
이 디렉토리(`githubblog/`)에서 실행:

```bash
git init -b main
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
git add .
git commit -m "chore: initial Hugo + PaperMod scaffold"
git remote add origin https://github.com/sunjin12/blog.git
git push -u origin main
```

### 3. GitHub Pages 활성화
리포 **Settings → Pages → Build and deployment → Source**: `GitHub Actions` 선택.

### 4. 첫 배포 확인
push 후 Actions 탭에서 워크플로가 성공하면 1~2분 내에 `https://sunjin12.github.io/blog/` 접속 가능.

## 로컬 미리보기 (선택)

Hugo 는 Actions 에서 빌드되므로 로컬 설치가 **필수는 아님**. 미리보기가 필요하면:

```bash
# Windows (PowerShell 관리자)
winget install Hugo.Hugo.Extended

# macOS
brew install hugo

# 실행
hugo server -D
# → http://localhost:1313/blog/
```

## 글 쓰기

```bash
hugo new content posts/my-first-post.md         # 한국어
hugo new content posts/my-first-post.en.md      # 영어 번역 (파일명에 .en)
```

front matter 에서 `draft: false` 로 바꾸고 `git push` 하면 자동 배포.

## 2단계: 광고 삽입 (추후)

본 프로젝트는 광고 연동을 위한 **훅·파티얼을 미리 심어둔 상태**. 실제 활성화는 간단합니다.

### 준비물
- Google AdSense 승인 (독자 콘텐츠 10~20편, About·Privacy 페이지 요구)
- AdSense publisher ID (`ca-pub-XXXXXXXXXXXXXXXX`)
- 광고 단위(ad unit) slot ID × 2 (상·하단용)

### 활성화 방법
`hugo.toml` 에서:

```toml
[params.ads]
  enabled = true
  client = "ca-pub-XXXXXXXXXXXXXXXX"
  topSlot = "1234567890"
  bottomSlot = "0987654321"
```

그리고 AdSense 가 요구하는 `ads.txt` 배치:

```bash
# static/ads.txt 파일 생성, 내용 예시:
# google.com, pub-XXXXXXXXXXXXXXXX, DIRECT, f08c47fec0942fa0
```

커밋·푸시하면 자동 반영. 광고는 `layouts/partials/ad-top.html` / `ad-bottom.html` 이 `layouts/_default/single.html` 안에서 본문 위·아래에 호출하도록 이미 배선됨.

## 3단계: 커스텀 도메인 (추후)

예: `blog.example.com` 으로 전환.

1. 도메인 등록기관 DNS 에 CNAME 레코드 추가:
   - `blog` → `sunjin12.github.io`
   - (apex 도메인인 경우 A 레코드 4개: 185.199.108.153 등 GitHub IP)
2. `static/CNAME` 파일 생성, 내용 한 줄: `blog.example.com`
3. GitHub **Settings → Pages → Custom domain** 입력, **Enforce HTTPS** 체크
4. `hugo.toml` 의 `baseURL` 을 `"https://blog.example.com/"` 로 변경
5. `.github/workflows/hugo.yml` 은 `configure-pages@v5` 가 `base_url` 을 자동으로 제공하므로 수정 불필요

내부 링크에는 항상 Hugo 의 `relURL`, `{{< ref >}}`, `{{< relref >}}` 를 사용했는지 확인 (하드코딩된 `github.io` URL 이 있으면 링크 깨짐).

## 파일 구조

```
.
├── .github/workflows/hugo.yml   # 자동 빌드·배포
├── archetypes/default.md        # 새 글 템플릿
├── assets/                      # 커스텀 CSS/JS (hugo.toml assets 에서 참조)
├── content/
│   ├── _index.md / .en.md       # 홈
│   ├── about.md / .en.md        # 소개 페이지
│   ├── privacy.md / .en.md      # 개인정보처리방침 (AdSense 대비)
│   ├── search.md / .en.md       # 검색 페이지
│   └── posts/
│       ├── _index.md / .en.md
│       └── hello.md / .en.md
├── layouts/
│   ├── _default/single.html     # PaperMod single 오버라이드 + 광고 훅 배선
│   └── partials/
│       ├── extend_head.html     # AdSense 로더 스크립트
│       ├── ad-top.html          # 본문 위 광고 슬롯 (gated)
│       └── ad-bottom.html       # 본문 아래 광고 슬롯 (gated)
├── static/                      # 정적 파일 (ads.txt, CNAME, favicon 등)
├── themes/PaperMod/             # submodule (저장소에 파일로 들어가지 않음)
├── hugo.toml                    # 전체 설정
└── .gitignore
```

## 테마 업데이트

```bash
cd themes/PaperMod
git pull origin master
cd ../..
git add themes/PaperMod
git commit -m "chore: update PaperMod"
```

## 트러블슈팅

- **Actions 빌드 실패: "SCSS" 또는 "Sass"**: workflow 의 Hugo 가 `hugo_extended_*` 인지 확인. PaperMod 는 extended 버전 필수.
- **스타일이 깨짐**: `baseURL` 끝에 `/` 가 있는지 확인. `/blog` 가 아닌 `/blog/`.
- **submodule 이 비어있음**: `git submodule update --init --recursive` 실행. Actions 쪽은 `submodules: recursive` 플래그로 자동 처리됨.
- **한글 파일명 포스트가 렌더 안 됨**: `hugo.toml` 의 `hasCJKLanguage = true` 확인.
