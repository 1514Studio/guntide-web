# guntide-web

[Guntide](https://github.com/1514Studio/guntide)의 **웹 빌드 산출물만** 담은 저장소.
소스는 여기 없다 — 위 링크가 소스 트리다.

GitHub Pages로 서빙된다. 저장소를 나눈 까닭은 하나다: 78MB짜리 빌드물이 소스
저장소에 쌓이면 클론할 때마다 따라오고, 커밋 하나하나가 새 사본을 남긴다.

## 이 빌드가 만들어진 방식

소스 저장소에서:

```bash
godot --headless --path . --export-release "Web" build/web/index.html
```

그 `build/web/` 내용을 이 저장소 루트에 그대로 옮긴 것이다.
`build/`는 소스 저장소에서 gitignore 되어 있다.

## 웹 빌드에서만 다른 것 두 가지

**렌더러가 Compatibility(WebGL2)다.** 브라우저에는 Vulkan도 D3D12도 없어서
Forward+ 템플릿 자체가 존재하지 않는다. 소스의 `project.godot`은
`renderer/rendering_method.web` 오버라이드로 **웹에서만** 내려 쓰고, 데스크톱은
Forward+ 그대로다. 볼류메트릭 포그·SDFGI 같은 Forward+ 전용 표현은 웹에서 빠지거나
다르게 보인다.

**한글 폰트를 번들한다.** 데스크톱에서는 Godot이 OS 폰트를 뒤져 한글을 대신 그려
주지만, 브라우저에는 뒤질 OS 폰트가 없다. 폰트를 안 넣으면 화면의 한글이 전부
두부(□)가 된다 — 라틴 문자는 내장 폰트에 있어서 제목만 읽히고 나머지가 다 깨진다.
Noto Sans KR 한국어 서브셋(4.6MB, OFL)이 들어가 있다.

## 스레드가 꺼져 있다

`web_nothreads` 템플릿으로 빌드했다. **GitHub Pages는 HTTP 헤더를 설정할 수 없어서**
`Cross-Origin-Opener-Policy` / `Cross-Origin-Embedder-Policy`를 못 붙이고, 그 둘이
없으면 브라우저가 `SharedArrayBuffer`를 막는다. 스레드 빌드는 그 위에서 돌아가지
않는다.

대가는 성능이다. 물리와 렌더가 한 스레드에 얹히므로 데스크톱 빌드보다 느리다.
헤더를 붙일 수 있는 호스팅(Netlify, Cloudflare Pages, 자체 서버)으로 옮긴다면
`export_presets.cfg`에서 `variant/thread_support=true`로 바꾸고 다시 내보내면 된다.

## `.nojekyll`

GitHub Pages는 기본으로 Jekyll을 돌리는데, 그러면 밑줄로 시작하는 파일이 무시되고
빌드 단계가 한 겹 낀다. 이 저장소에 그럴 파일은 없지만, 꺼 두는 편이 빠르고
예상 밖의 일이 없다.
