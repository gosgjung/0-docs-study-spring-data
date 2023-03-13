---
layout: default
title: Intellij 에러 대처법들
nav_order: 3
has_children: false
parent: 에러나요!!!
permalink: /docs/error-solutions/intellij
---

# Intellij 에러 대처법들
{: .no_toc }
<br>
<br>

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


### java.lang.classnotfoundexception
{: .fs-6 .fw-700 }

#### 1\) 윈도우에서 파일 경로에 한글 또는 특수문자, ',' 가 있을 경우 
{: .fs-5 .fw-700 }

- **프로젝트를 모아두는 디렉터리는 가급적 한글명 디렉터리,특수문자 디렉터리는 피하는게 좋다.**
<br>

검색해보다가 다 나하고 상관 없는 내용이었고, [java.lang.classnotfoundexception 에러](https://masssal.tistory.com/39) 에 있는 내용에서 힌트를 얻었다.

위의 링크에서는 아래의 원인들을 이야기해준다.
- 1\) 모듈 -> 소스 -> src/main/java 가 잡혀있지 않다. 
  - 인텔리제이 메뉴에서 Mark as 메뉴로 해결가능하다.
  - 나한테는 해당되지 않았다.

- 2\) 프로젝트 파일명에 "_" 가 들어간 경우 수정해주면 해결된다/
  - 나한테 해당될수도 있겠다 싶었다.
<br>


그래서 소스폴더를 평소 코드를 자주 모아두는 곳에서 다시 실행해보니 잘 실행된다.
<br>
<br>


