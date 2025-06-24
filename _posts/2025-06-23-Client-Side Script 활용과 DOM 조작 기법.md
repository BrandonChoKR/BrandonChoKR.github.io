---
layout: post
title: "Client-Side Script 활용과 DOM 조작 기법"
date: 2025-06-23 10:00:00 +0900
categories: [XSS]
tags: [XSS, location, DOM, 스크립트 우회]
---
## Client Script 활용 기법
1. 페이지 리디렉션 (Page Redirect)  
   사용자의 브라우저를 악의적인 사이트 등으로 강제로 이동시킬 수 있는 스크립트입니다.

   - **location.href 사용**
     - 사용자를 다른 주소로 강제 이동시킵니다.  
     - 브라우저 히스토리에 기록됨.
     ```javascript
     &lt;script&gt;location.href = "http://attacker.com/";&lt;/script&gt;
     ```

   - **location.replace() 사용**
     - 마찬가지로 리디렉션하지만 브라우저 히스토리에 현재 페이지를 남기지 않음 (뒤로가기 방지용)
     ```javascript
     location.replace("http://attacker.com/");
     ```

2. 주소창 변조 (Address Bar Spoofing)  
   사용자가 보는 주소창의 내용을 조작해 신뢰할 수 없는 페이지를 신뢰하도록 유도합니다.

   - **history.pushState() 사용**
     - 실제 페이지는 변하지 않지만 주소창에 표시되는 URL을 임의로 바꿀 수 있음  
     - 보통 Reflected XSS 등과 함께 사용되어 실제 악성 페이지를 신뢰할 수 있는 주소처럼 위장하는 데 활용됨
     ```javascript
     history.pushState(null, null, 'fakepath');
     ```

## Javascript로 DOM 객체 접근 방법
자바스크립트에서 HTML페이지에 접근을 할 때는 무조건 document라는 객체를 사용해서 접근해야합니다. 

1. document.getElementBy...    
    JavaScript에서 웹페이지의 HTML 요소(DOM 요소)를 선택(찾기) 할 때 사용하는 DOM 메서드입니다.  
    document는 현재 웹 페이지(문서)를 의미하며 그 뒤에 붙는 getElementById, getElementsByClassName 등은 선택 방식입니다.

<img src="{{ '/assets/img/2025-06-23-Client-Side Script 활용과 DOM 조작 기법/2025-06-23-screenshot1.png' | relative_url }}" style="width:700px; height:auto;" alt="테스트 이미지">

하나의 요소는 id와 class 두 가지 정보를 통해 식별될 수 있습니다.  
id는 HTML 문서 내에서 고유한 값으로, 한 번만 사용되어야 하며 특정 요소 하나를 정확히 지정할 때 사용됩니다.  
반면, class는 요소들을 그룹화하거나 분류하는 용도로 사용되며, 여러 요소가 동일한 class명을 가질 수 있습니다.  

<img src="{{ '/assets/img/2025-06-23-Client-Side Script 활용과 DOM 조작 기법/Screenshot 2025-06-23 at 9.44.56 pm.png' | relative_url }}" style="width:700px; height:auto;" alt="테스트 이미지">  

### Javascript로 DOM 객체 접근하는 단계

1. document.getElementById(’userName’) 으로 입력하면, 아래와 같이 특정 객체에 접근 할 수 있습니다.  
<img src="{{ '/assets/img/2025-06-23-Client-Side Script 활용과 DOM 조작 기법/Screenshot 2025-06-24 at 12.44.45 am.png' | relative_url }}" style="width:700px; height:auto;" alt="테스트 이미지">  

2. document.getElementsByClassName('card-title')
- docuemnt.getElementsByClassName(’card-title’);[0] → 브라켓+숫자를 통하여 원하는 데이터를 가져올 수 있습니다.
- docuemnt.getElementsByClassName(’card-title’);[1]
<img src="{{ '/assets/img/2025-06-23-Client-Side Script 활용과 DOM 조작 기법/Screenshot 2025-06-24 at 12.46.37 am.png' | relative_url }}" style="width:700px; height:auto;" alt="테스트 이미지">  

<img src="{{ '/assets/img/2025-06-23-Client-Side Script 활용과 DOM 조작 기법/Screenshot 2025-06-24 at 12.52.01 am.png' | relative_url }}" style="width:700px; height:auto;" alt="테스트 이미지">  

3. document.getElementById(’userName’).class 를 입력하여 속성값을 가져올 수 있습니다 (.class, .id, .className)

<img src="{{ '/assets/img/2025-06-23-Client-Side Script 활용과 DOM 조작 기법/Screenshot 2025-06-24 at 12.55.18 am.png' | relative_url }}" style="width:700px; height:auto;" alt="테스트 이미지">  

4. document.getElementById(’userName’).innerHTML 를 입력하여 태그 안에 있는 것을 가져올 수 있습니다 (글자 또는 안에 태그가 있으면 태그를 가져옵니다.)

<img src="{{ '/assets/img/2025-06-23-Client-Side Script 활용과 DOM 조작 기법/Screenshot 2025-06-24 at 12.57.26 am.png' | relative_url }}" style="width:700px; height:auto;" alt="테스트 이미지">  