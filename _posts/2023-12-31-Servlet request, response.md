---
layout: single
title:  "Servlet request, response"
excerpt: "Servlet request, response"

categories: Jsp
tags:  Jsp

permalink: /categories/Servlet request, response/ # url

toc: true
toc_sticky: true

date: 2023-12-31
last_modified_at: 2023-12-31
---

# Servlet request, response

<center><img src="/assets/images/posts_img/readme/HttpServlet.png" width="50%" height="50%"></center>
참고 : 실선은 상속관계, 점선은 인터페이스 구현관계에 있는 다이어그램을 나타냄.

# HttpServlet
ServletEx 클래스를 만들기 위해선 HttpServlet 클래스를 상속받아서 만들어야함.
이렇게 많은 인터페이스와 추상클래스를 상속 받아서 서블릿을 만드는 이유는 로컬에서 작업하는것이 아니라,
웹 서비스라는건, 웹서버, 웹 어플리케이션 서버와의 통신, 프로토콜을 이용한 통신이라고 볼수가 있겠죠?
그 과정에서 많은 데이터가 오고 가고 다양한 데이터가 오고 갈 수 있기 때문에 우리가 그런 것을
다 구현을 해내려면 많은 기능들이 필요한데요.
그 많은 기능들을 이미 표준화해서 이렇게 많이 만들어 놓았습니다.
그래서 우리 개발자는 쉽게 HttpServlet만 상속을 받아서 사용을 하면 이 많은 기능들을 다 사용을 할 수가 있음.
참고 : 클래스 파일이 아니라 서블릿 파일을 만들면 자동으로 extends HttpServlet를 해줌!

# TestServletClass.java

```java
package com.servlet;

import java.io.IOException;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet("/tsc")
public class TestServletClass extends HttpServlet {
	private static final long serialVersionUID = 1L;
       
	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		// TODO Auto-generated method stub
		response.getWriter().append("Served at: ").append(request.getContextPath());
	}

	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		// TODO Auto-generated method stub
		doGet(request, response);
	}

}
```
# HttpServletRequest
요청에 대한 정보를 가지고 있는 객체
user (Request) -> servlet | 웹 컨테이너(tomcat)

```
request.getCookies();
request.getSession();
request.getAttribute(null);
request.setAttribute(null, null);
request.getParameter(null);
request.getParameterNames();
request.getParameterValues(null);
```

# HttpServletResponse
응답에 대한 정보를 가지고 있는 객체
user <- (Response) servlet | 웹 컨테이너(tomcat)

```
response.addCookie(null);
response.getStatus();
response.sendRedirect(null);
response.getWriter();
response.getOutputStream();
```
