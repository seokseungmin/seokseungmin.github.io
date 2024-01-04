---
layout: single
title:  "Connection Pool"
excerpt: "Connection Pool"
categories: Jsp
tags:  Jsp

permalink: /categories/Connection Pool/ # url

toc: true
toc_sticky: true

date: 2024-01-03
last_modified_at: 2024-01-03
---

# Connection Pool

<center><img src="/assets/images/posts_img/readme/커넥션풀1.png" width="100%" height="100%"></center>

# context.xml

커넥션풀을 사용하기 위해 context.xml에 다음 코드를 삽입해준다.
```xml
<Resource auth="Container" type="javax.sql.DataSource"
              maxActive="4" maxWait="10000"
              username="scott" password="tiger"
              name="jdbc/Oracle11g"
              driverClassName="oracle.jdbc.OracleDriver"
              url="jdbc:oracle:thin:@localhost:1521:xe"/>
```
# newBook.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" 
   pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>Insert title here</title>
    </head>
    <body>
		
		<form action="newBook" method="post">
			book name : <input type="text" name="book_name"></br>
			book location : <input type="text" name="book_loc"></br>
			<input type="submit" value="book register">
		</form>
		
    </body>
</html>
```

# BookServlet.java
책 정보 조회

```java
package com.servlet;

import java.io.IOException;
import java.io.PrintWriter;
import java.util.ArrayList;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet("/bs")
public class BookServlet extends HttpServlet {

	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	
		response.setContentType("text/html; charset=UTF-8");
		PrintWriter out = response.getWriter();
		
		BookDAO bookDAO = new BookDAO();
		ArrayList<BookDTO> list = bookDAO.select();
		
		for (int i = 0; i < list.size(); i++) {
			BookDTO dto = list.get(i);
			int bookId = dto.getBookId();
			String bookName = dto.getBookName();
			String bookLoc = dto.getBookLoc();
			
			out.println("bookId : " + bookId + ", ");
			out.println("bookName : " + bookName + ", ");
			out.println("bookLoc : " + bookLoc + "</br>");
		}
		
	}

	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		doGet(request, response);
	}

}
```

# BookDTO.java

```java
package com.servlet;

public class BookDTO {
	
	int bookId;
	String bookName;
	String bookLoc;
	
	public BookDTO(int bookId, String bookName, String bookLoc) {
		this.bookId = bookId;
		this.bookName = bookName;
		this.bookLoc = bookLoc;
	}

	public int getBookId() {
		return bookId;
	}

	public String getBookName() {
		return bookName;
	}

	public String getBookLoc() {
		return bookLoc;
	}

}
```

# BookDAO.java

```java
package com.servlet;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.util.ArrayList;

import javax.naming.Context;
import javax.naming.InitialContext;
import javax.sql.DataSource;

public class BookDAO {

	DataSource dataSource;
	
	/*
	String driver = "oracle.jdbc.driver.OracleDriver";
	String url = "jdbc:oracle:thin:@localhost:1521:xe";
	String id = "scott";
	String pw = "tiger";
	*/
	
	public BookDAO() {
		try {
//			Class.forName(driver);
			Context context = new InitialContext();
			dataSource = (DataSource)context.lookup("java:comp/env/jdbc/Oracle11g");
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	
	public ArrayList<BookDTO> select() {
		
		ArrayList<BookDTO> list = new ArrayList<BookDTO>();
		
		Connection con = null;
		PreparedStatement pstmt = null;
		ResultSet res = null;
		
		try {
//			con = DriverManager.getConnection(url, id, pw);
			con = dataSource.getConnection();
			String sql = "SELECT * FROM book";
			pstmt = con.prepareStatement(sql);
			res = pstmt.executeQuery();
			while (res.next()) {
				int bookId = res.getInt("book_id");
				String bookName = res.getString("book_name");
				String bookLoc = res.getString("book_loc");
				
				BookDTO bookDTO = new BookDTO(bookId, bookName, bookLoc);
				list.add(bookDTO);
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				if(res != null) res.close();
				if(pstmt != null) pstmt.close();
				if(con != null) con.close();
			} catch (Exception e2) {
				e2.printStackTrace();
			}
		}
		
		return list;
	}
	
}
```

# 정리
```
Connection Pool은 데이터베이스 연결 관리를 효율적으로 처리하기 위한 기술입니다.
데이터베이스 연결은 비용이 높은 작업 중 하나이기 때문에, 매번 연결을 생성하고 해제하는 것은 성능에 부담을 줄 수 있습니다.
Connection Pool은 이러한 비용을 최소화하고 성능을 향상시키기 위해 사용됩니다.

Connection Pool의 동작은 대략 다음과 같습니다:

초기화: 애플리케이션이 시작될 때, 미리 정의된 개수의 데이터베이스 연결이 생성되고 풀에 저장됩니다.
요청 처리: 애플리케이션에서 데이터베이스에 연결이 필요할 때마다, Connection Pool에서 미리 생성된 연결 중 하나를 할당해줍니다.
사용 완료 후 반환: 사용이 끝난 연결은 다시 풀에 반환되어 재사용될 수 있습니다. 이때 연결의 상태가 초기화되고, 다음 사용을 대비합니다.
최소/최대 연결 수: Connection Pool은 일반적으로 최소 연결 수와 최대 연결 수를 설정할 수 있습니다.
최소 연결 수는 풀에 항상 유지되어야 하는 최소한의 연결 수를 의미하며, 최대 연결 수는 동시에 생성될 수 있는 최대 연결 수를 의미합니다.

Connection Pool을 사용하는 이점은 다음과 같습니다:

성능 향상: 연결 풀을 통해 미리 생성된 연결을 재사용하므로, 매번 연결을 새로 만들 필요가 없어지며 성능이 향상됩니다.
자원 관리: 연결 풀은 연결을 효과적으로 관리하고, 너무 많은 연결이나 너무 적은 연결로 인한 자원의 낭비를 방지합니다.
응답 시간 감소: 이미 생성된 연결을 재사용하므로 데이터베이스에 접속하기 위한 대기 시간이 감소하고, 응답 시간이 단축됩니다.

Connection Pool은 대부분의 웹 애플리케이션 서버 및 데이터베이스 연결 라이브러리에서 지원되며, Java에서는 대표적으로 Apache DBCP, HikariCP, C3P0 등이 Connection Pool을 구현한 라이브러리로 널리 사용됩니다.
출처 : chatGPT
```


