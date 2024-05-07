---
title: "getOrDefault"
excerpt: "[TIL] getOrDefault"

categories:
  - Today I Learned
tags:
  - [Today I Learned]

permalink: /categories/getOrDefault/ # url

toc: true
toc_sticky: true

date: 2024-05-07
last_modified_at: 2024-05-07
---


## getOrDefault
찾는 키가 존재한다면 찾는 키의 값을 반환하고 없다면 기본 값을 반환하는 메서드

## 사용 방법
getOrDefault(Object key, V DefaultValue)
매개 변수: 이 메서드는 두 개의 매개 변수를 허용합니다.

key: 값을 가져와야 하는 요소의 키입니다.
defaultValue: 지정된 키로 매핑된 값이 없는 경우 반환되어야 하는 기본값입니다.

반환 값: 찾는 key가 존재하면 해당 key에 매핑되어 있는 값을 반환하고, 그렇지 않으면 디폴트 값이 반환됩니다.

다음은 getOrDefault 메서드의 사용법입니다.

```java
import java.util.HashMap;

public class MapGetOrDefaultEx {
    public static void main(String arg[]) {
        String[] alphabet = {"A", "B", "C", "A"};
        HashMap<String, Integer> hashmap = new HashMap<>();
        for (String key : alphabet) hashmap.put(key, hashmap.getOrDefault(key, 0) + 1);
        System.out.println("결과 : " + hashmap);
        // 결과 : {A=2, B=1, C=1}
    }
}
```

HashMap의 경우 동일 키 값을 추가할 경우 Value의 값이 덮어쓰기가 됩니다.<br>
따라서 기존 key 값의 value를 계속 사용하고 싶을 경우 getOrDefault 메서드를 사용하여 위의 예와 같이 사용할 수 있습니다.<br>

출처: [코딩 시그널:티스토리](https://junghn.tistory.com/entry/JAVA-Map-getOrDefault-이란-사용법-및-예제)
