---
layout: single
title:  "Servlet Life-Cycle"
excerpt: "Servlet Life-Cycle"
categories: Jsp
tags:  Jsp

permalink: /categories/2023-12-28-Servlet Life-Cycle/ # url

toc: true
toc_sticky: true

date: 2023-12-31
last_modified_at: 2023-12-31
---

# Servlet Life-Cycle

<center><img src="/assets/images/posts_img/readme/서블릿생명주기.png" width="50%" height="50%"></center>

```
서블릿은 처음에 생성을 할수가 있고, 그 생성 단계를 우리가 init() 단계라고 함.
서블릿이 막 일을해요. 우리 개발자가 구현한 방식으로 구현된 기능을 이용해서 여러가지 일을 할 수가 있겠죠.
그럴때 그것을 서비스 단계라고 하고요 다음에 일을 다 끝내면, 서블릿이 컨테이너에서 소멸될 때 종료되는 시점이 오게 됨.
그런것을 우리는 destory() 단계라고함.
서블릿이 시작을 한다 하기 전에 준비를 하는 단계가 있음, 그것을 우리가 @PostConstruct 라고함.
서블릿이 일을 다 한다음에 더이상 필요가 없으면, 이 서블릿을 정리하기 위한 단계로 @PreDestroy라는 단계를 거침.
```

# 생명주기 관련 메서드

<center><img src="/assets/images/posts_img/readme/생명주기관련메서드.png" width="50%" height="50%"></center>

```
단계별로 호출, 콜백 메서드를 제공함으로써 각 단계별로 우리가 어떠한 특정한 작업을 더 할수가 있음.
init() 단계에서는 init() 메서드를 우리가 override 재정의해서 이안에다가 어떤 우리 개발자가
기술해주고 싶은 내용을 기술 해주면 그 기능이 컨테이너에서 작동을 함.
다음에 doGet()단계, 서비스 단계인데 서비스 메서드는 잘 이용을 안하고요.
실제로 작업을 하는건 doget()에서 많이 작업을 합니다.
그래서 여기서 우리가 실제로 서블릿이 하는 일, 주요 업무는 여기에 다 기술을 해주면 되겠고요.
다음에 서블릿이 작업을 다해서 종료하고 끝날 때는 destroy()라는 메서드를 오버라이드해서 이곳에다가
우리가 기능에 대한 부분을 기술해주면됨.
만약에 init() 단계와 destroy() 단계가 필요가 없으면, 실행 단계만 있으면 된다하면 굳이 init() 메서드와
destroy()메서드를 오버라이드 하지 않고 doGet()이나 doPost()만 작성을 하면됨.
init() 단계와 destroy() 단계 전에 @postConstruct라는 어노테이션을 붙여서 나만의, 개발자만의 별다른 별도의
메서드를 하나 정의를 해서, 선언해서 사용을 할 수가 있음.
서블릿이 종료를 하고 소멸되는 시점에 호출되는 메서드를 하나 만들수가 있는데,
그 메서드로 @preDestroy라는 어노테이션을 이용해서 개발자가 별도의 이름을 만들어서, 별도의 메서드로 하나 선언을 해서
사용할 수도 있음.
```

```Java
package com.servlet;

import java.io.IOException;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet("/tsc")
public class TestServletClass extends HttpServlet {
	private static final long serialVersionUID = 1L;
       
	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		response.getWriter().append("Served at: ").append(request.getContextPath());
		System.out.println(" --- doGet() ---");

	}

	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		// TODO Auto-generated method stub
		doGet(request, response);
	}

	@PostConstruct
	public void funPc() {
		System.out.println(" --- @PostConstruct ---");
	}
	@Override
	public void init() throws ServletException {
		System.out.println(" --- init() ---");
	}
	
	@Override
	public void destroy() {
		System.out.println(" --- destroy() ---");
	}
	
	@PreDestroy
	private void funPd() {
		System.out.println(" --- @PreDestroy ---");
	}
}
```

    --- @PostConstruct ---
    --- init() ---
    --- doGet() ---
