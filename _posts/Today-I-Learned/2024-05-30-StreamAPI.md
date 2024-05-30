---
title: "Stream API"
excerpt: "[TIL] Stream API"

categories:
  - Today I Learned
tags:
  - [Today I Learned, Java]

permalink: /categories/StreamAPI/ # url

toc: true
toc_sticky: true

date: 2024-05-30
last_modified_at: 2024-05-30
---

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class Main {

    static class A {
        int x, y;

        public A(int x, int y) {
            this.x = x;
            this.y = y;
        }

    }

    public static void main(String[] args) {

        //Stream API 이용한 구현
        List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));

        //중간 처리 mapToInt -> IntStream
        //i라는 입력에 대해 i라는 출력으로 전환
        //max, OptionalInt -> 최종처리
        //리스트가 비어있는 경우 때문에 null 때문에 Optional처리
        //getAsInt -> Optional처리 값 가져오기

        int maxVal = list.stream().mapToInt(i -> i).max().getAsInt();
        System.out.println("maxVal : " + maxVal);

        //스트림의 특징
        //원본 데이터를 변경하지 않는다.
        //스트림은 일회용이다.
        //내부 반복으로 작업을 처리한다.
        //체이닝으로 작업

        //스트림 사용 예시
        //리스트를 배열로 반환
        //배열을 리스트로 반환

        //List<Integer> -> Integer[]
        //Integer = Wrapping class
        //list.stream(). -> Stream<Integer>
        //Integer[]::new) 항상 배열의 생성자를 만들어 줘야함.
        Integer[] array = list.stream().toArray(Integer[]::new);


        //List<Integer> -> int[]
        //list.stream() -> Stream<Integer>
        //mapToInt -> IntStream
        //toArray -> 이미 int[]로 만들어주는걸 알고 있기 때문에 배열의 생성자 필요X
        int[] intArray = list.stream().mapToInt(i -> i).toArray();


        //int[] -> List<Integer>
        //Arrays.stream(intArray) -> IntStream
        //boxed int가 Integer가 되는것 박싱 의미. -> stream<Integer>
        //collect 반환 받을때 특별한 값으로 반환 받기위한 작업
        //배열을 특정한 형식으로 바꿔주려할때
        //Collectors.toList() -> List<Integer>로 반환됨.

        //java 8.
        List<Integer> list1 = Arrays.stream(intArray).boxed().collect(Collectors.toList());
        //java 16이상
        List<Integer> list2 = Arrays.stream(intArray).boxed().toList();


        //여러 개의 배열을 동시에 반복.
        int[] array1 = {10, 20, 30, 40};
        int[] array2 = {100, 200, 300, 400};

        System.out.println("== 두 배열을 동시에 사용! 두배열 더하기 ==");
        //range(0, array1.length). -> 0~array.length-1 반복하는 IntStream
        //map(i -> array1[i] + array2[i]) -> 람다식을 이용한 기능 커스텀
        //System.out::println -> void
        IntStream.range(0, array1.length).map(i -> array1[i] + array2[i]).forEach(System.out::println);

        //두 배열을 결합하여 A객체 생성
        int[] array3 = {10, 20, 30, 40};
        int[] array4 = {100, 200, 300, 400};
        //IntStream.range(0, array3.length). -> IntStream
        //boxed -> Stream<Integer>
        //map(i -> new A(array3[i], array4[i])) -> Stream<A>
        //collect(Collectors.toList()) -> List<A>

        List<A> as = IntStream.range(0, array3.length).boxed().map(i -> new A(array3[i], array4[i])).collect(Collectors.toList());
        System.out.println("== 두 배열을 결합하여 A객체 생성 ==");
        System.out.println("List<A> : " + as);

        //절대값이 최대인 값 찾기
        List<Integer> listAbs = new ArrayList<>();
        listAbs.add(-10);
        listAbs.add(40);
        listAbs.add(110);
        listAbs.add(-140);

        //istAbs.stream() -> Stream<Integer>
        //max((x, y) -> Math.abs(x) - Math.abs(y)) -> Optional<Integer>
        //(x, y) -> Math.abs(x) - Math.abs(y) -> Comperator
        //get() -> Integer
        int getMaxVal = listAbs.stream().max((x, y) -> Math.abs(x) - Math.abs(y)).get();
        System.out.println("== 절대값이 최대인 값 찾기 ==");
        System.out.println("getMaxVal : " + getMaxVal);
        //숫자를 정렬후 문자열로 이어 붙이기
        //[10, 40, 9, 15] -> [9, 10, 15, 40] -> "9101450"

        List<Integer> listConcat = new ArrayList<>();
        listConcat.add(10);
        listConcat.add(40);
        listConcat.add(9);
        listConcat.add(15);

        //listConcat.stream(). -> Stream<Integer>
        //sorted() -> Stream<Integer>
        //map(String::valueOf) -> Stream<String>
        //collect(Collectors.joining()) -> String
        //joining() -> 사이에 문자열을 끼워서 문자열을 연결

        String result = listConcat.stream().sorted().map(String::valueOf).collect(Collectors.joining());
        System.out.println("== 숫자를 정렬후 문자열로 이어 붙이기 ==");
        System.out.println("listConcat : " + result);
    }
}
```

```
maxVal : 5
== 두 배열을 동시에 사용! 두배열 더하기 ==
110
220
330
440
== 두 배열을 결합하여 A객체 생성 ==
List<A> : [Main$A@49097b5d, Main$A@6e2c634b, Main$A@37a71e93, Main$A@7e6cbb7a]
== 절대값이 최대인 값 찾기 ==
getMaxVal : -140
== 숫자를 정렬후 문자열로 이어 붙이기 ==
listConcat : 9101540
```
