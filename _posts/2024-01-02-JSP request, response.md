---
layout: single
title:  "JSP request, response"
excerpt: "JSP request, response"
categories: Jsp
tags:  Jsp

permalink: /categories/JSP request, response/ # url

toc: true
toc_sticky: true

date: 2024-01-02
last_modified_at: 2024-01-02
---

# JSP request, response

# request 객체

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="EUC-KR">
<title>Insert title here</title>
</head>
<body>

	<form action="mSignUp.jsp" method="get">
		name : <input type="text" name="m_name"><br>
		password : <input type="password" name="m_password"><br>
		gender : <input type="text" name="m_gender"><br>
		hobby : sport<input type="checkbox" name="m_hobby" value="sport">,
		        cooking<input type="checkbox" name="m_hobby" value="cooking">,
		        travel<input type="checkbox" name="m_hobby" value="travel"><br>
		residence : <input type="text" name="m_residence"><br>
		<input type="submit" value="sign up">
	</form>
</body>
</html>
```

```jsp
<%@ page language="java" contentType="text/html; charset=EUC-KR"
    pageEncoding="EUC-KR"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="EUC-KR">
<title>Insert title here</title>
</head>
<body>
	<%!
		String m_name;
		String m_password;
		String m_gender;
		String[] m_hobby;
		String m_residence;
	%>
	
	<%
		m_name = request.getParameter("m_name");
		m_password = request.getParameter("m_password");
		m_gender = request.getParameter("m_gender");
		m_hobby = request.getParameterValues("m_hobby");
		m_residence = request.getParameter("m_residence");
	%>
	
	    m_name : <%= m_name %> <br>
	    m_password : <%= m_password %> <br>
		m_gender : <%= m_gender %> <br>
		m_hobby : 
		<%
		for(int i = 0;  i < m_hobby.length; i++){
		%>
		<%= m_hobby[i] %>
		<%
		}
		%><br>	
		m_residence : <%= m_residence %>
</body>
</html>
```

    m_name : seok seung min
    m_password : 1017
    m_gender : M
    m_hobby : sport cooking travel
    m_residence : Young-in

# response 객체

```jsp
<%@ page language="java" contentType="text/html; charset=EUC-KR"
    pageEncoding="EUC-KR"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="EUC-KR">
<title>Insert title here</title>
</head>
<body>
	<p>First Page!!</p>
	
	<%
		response.sendRedirect("secondPage.jsp");
	%>
</body>
</html>
```

```jsp
<%@ page language="java" contentType="text/html; charset=EUC-KR"
    pageEncoding="EUC-KR"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="EUC-KR">
<title>Insert title here</title>
</head>
<body>
	<p>second Page!!</p>
</body>
</html>
```

```
http://localhost:8090/testPjt/firstPage.jsp -> 요청하면 http://localhost:8090/testPjt/secondPage.jsp가 응답함.
```
