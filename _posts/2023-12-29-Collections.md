---
layout: single
title:  "Collections"
excerpt: "Collections"

categories: Java
tags:  Java

permalink: /categories/2023-12-29-Collections/ # url

toc: true
toc_sticky: true

date: 2023-12-29
last_modified_at: 2023-12-30
---

# Collections
# List
```
List는 인터페이스로 이를 구현한 클래스는 인덱스를 이용해서 데이터를 관리한다.
List(구현) -> Vector
           -> ArrayList
           -> LinkedList
특징
인덱스를 이용한다.
데이터 중복이 가능하다.
```
```Java
package pjtTest;

import java.util.ArrayList;

public class MainClass {
	public static void main(String[] args) {
		
		//ArrayList 객체 생성
		ArrayList<String> list = new ArrayList<String>();
		//Ctrl + Shift + O 하면 자동 임포트
		System.out.println("list.size : "+ list.size());
		 
		//데이터 추가
		list.add("Hello");
		list.add("Java");
		list.add("World");
		System.out.println("list.size : " + list.size());
		System.out.println("list : " + list);
		
		list.add(2, "Programing"); //추가
		System.out.println("list : " + list);
		
		list.set(1, "C"); //변경
		System.out.println("list : " + list);
		
		//데이터 추출
		String str = list.get(2);
		System.out.println("list.get(2) : " + str);
		System.out.println("list : " + list);
		
		//데이터 제거
		str = list.remove(2);
		System.out.println("list.remove(2) : " + str);
		System.out.println("list : " + list);
		
		//데이터 전체 제거
		list.clear();
		System.out.println("list : " + list);
		
		//데이터 유무
		boolean b = list.isEmpty();
		System.out.println("list.isEmpty() : " + b);
		
		System.out.println("==================================");
		
	}
}
```
    list.size : 0
    list.size : 3
    list : [Hello, Java, World]
    list : [Hello, Java, Programing, World]
    list : [Hello, C, Programing, World]
    list.get(2) : Programing
    list : [Hello, C, Programing, World]
    list.remove(2) : Programing
    list : [Hello, C, World]
    list : []
    list.isEmpty() : true
    ==================================

# Map
# Map은 인터페이스로 이를 구현한 클래스는 Key를 이용해서 데이터를 관리한다.
```
HashMap (구현) -> Map
특징
-> Key를 이용한다.
-> Key는 중복될 수 없다.
-> 데이터 중복이 가능하다.
```
```Java
package pjtTest;

import java.util.HashMap;

public class MainClass {
	public static void main(String[] args) {
		
		//HashMap 객체 생성
		HashMap<Integer, String> map = new HashMap<Integer, String>();
		System.out.println("map.size() : " + map.size());

		//데이터 추가
		map.put(5, "Hello");
		map.put(6, "Java");
		map.put(7, "World");
		System.out.println("map : " + map);
		System.out.println("map.size() : " + map.size());
		
		map.put(8, "!!");
		System.out.println("map : " + map);
		
		//데이터 교체
		map.put(6, "c");
		System.out.println("map : " + map);
		
		//데이터 추출
		String str = map.get(5);
		System.out.println("map.get(5) : " + str);
		
		//데이터 제거
		map.remove(8);
		System.out.println("map : " + map);
		
		//특정 데이터 포함 유무
		boolean b = map.containsKey(7);
		System.out.println("map.containsKey(7) : " + b);
		
		b = map.containsValue("World");
		System.out.println("map.containsValue(\"World\") : " + b);
		
		//데이터 전체 제거
		map.clear();
		System.out.println("map : " + map);
		
		//데이터 유무
		b = map.isEmpty();
		System.out.println("map.isEmpty() : " + b);
		
		System.out.println("==================================");
	}
}
```
    map.size() : 0
    map : {5=Hello, 6=Java, 7=World}
    map.size() : 3
    map : {5=Hello, 6=Java, 7=World, 8=!!}
    map : {5=Hello, 6=c, 7=World, 8=!!}
    map.get(5) : Hello
    map : {5=Hello, 6=c, 7=World}
    map.containsKey(7) : true
    map.containsValue("world") : true
    map : {}
    map.isEmpty() : true
    ==================================
