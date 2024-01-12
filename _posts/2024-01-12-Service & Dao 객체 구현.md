---
layout: single
title: "Service & Dao 객체 구현"
excerpt: "Service & Dao 객체 구현"

categories: Spring
tags: Spring

permalink: /categories/Service & Dao 객체 구현/ # url

toc: true
toc_sticky: true

date: 2024-01-12
last_modified_at: 2024-01-12
---

# Service & Dao 객체 구현

## 웹 어플리케이션 구조
<center><img src="/assets/images/posts_img/readme/웹어플리케이션구조.png" width="100%" height="100%"></center>

## 웹 어플리케이션 준비
<center><img src="/assets/images/posts_img/readme/웹어플리케이션준비.png" width="100%" height="100%"></center>

<center><img src="/assets/images/posts_img/readme/웹어플리케이션준비2.png" width="100%" height="100%"></center>

## 한글처리
<center><img src="/assets/images/posts_img/readme/한글처리.png" width="100%" height="100%"></center>

### 한글처리를 위해 web.xml에 추가하기

```xml
<filter>
<filter-name>encodingFilter</filter-name>
<filter-class>
org.springframework.web.filter.CharacterEncodingFilter 
</filter-class>
<init-param>
<param-name>encoding</param-name>
<param-value>UTF-8</param-value>
</init-param>
<init-param>
<param-name>forceEncoding</param-name>
<param-value>true</param-value>
</init-param>
</filter>
<filter-mapping>
<filter-name>encodingFilter</filter-name>
<url-pattern>/*</url-pattern>
</filter-mapping>
```

## 서비스 객체 구현
<center><img src="/assets/images/posts_img/readme/서비스객체구현.png" width="100%" height="100%"></center>

## DAO 객체 구현
<center><img src="/assets/images/posts_img/readme/DAO객체구현.png" width="100%" height="100%"></center>

