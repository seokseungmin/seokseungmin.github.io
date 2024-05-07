---
title: "Java Mini Project7"
excerpt: "[제로베이스] 로또 당첨 프로그램"

categories:
  - Project
tags:
  - [Project]

permalink: /categories/JavaMiniProject7/ # url

toc: true
toc_sticky: true

date: 2024-05-07
last_modified_at: 2024-05-07
---

## 로또 당첨 프로그램
수행 목적 : Scanner의 입력함수와 조건문 및 반복문과 배열을 통한 로또 당첨 로직 작성<br>
간략 소개 : 로또는 1-45개의 숫자 사이의 값중 6개를 맞추면 당첨되는 복권입니다. <br>
로또의 개수를 구매하고(구매수량 입력), 당첨번호를 생성한다. 이후, 구매한 로또의 당첨번호를 판단하는 프로그램을 작성해 보세요.<br>

## 필수 준수사항
로또 구매 수량 입력<br>
입력한 개수만큼의 로또 개수 생성<br>
로또 당첨 번호 생성(숫자값은 중복 배제 및 정렬해서 표시)<br>
당첨 번호와 구매 로또 비교하여 숫자 일치 여부 판단<br>
Collections.shuffle 함수 사용 금지!(shuffle함수는 과제의 취지와 맞지 않기 때문에, 사용시 0<br>

```java
import java.util.*;

public class Project7 {
    public static void main(String[] args) {
        LottoProgram();
    }

    private static void LottoProgram() {
        System.out.println("[로또 당첨 프로그램]\n");

        try {
            System.out.print("로또 개수를 입력해 주세요.(숫자 1 ~ 10): ");
            Scanner scanner = new Scanner(System.in);
            int lottoNum = scanner.nextInt();

            Random random = new Random();
            HashMap<Character, Set<Integer>> lottoMapping = new HashMap<>();
            Set<Integer> LottoNumbers = new HashSet<>();

            generateMyLottoNumbers(lottoNum, random, lottoMapping);
            printMyLottoNumbers(lottoMapping);
            printLottoNumbers(LottoNumbers, random);
            printMyLottoResults(lottoMapping, LottoNumbers);
        } catch (InputMismatchException e) {
            System.out.println("숫자를 입력해주세요.");
            e.printStackTrace();
        }
    }

    private static void generateMyLottoNumbers(int lottoNum, Random random, HashMap<Character, Set<Integer>> lottoMapping) {
        // A,B,C...J까지 최대 10개
        String alphabet = "ABCDEFGHIJ";
        for (int i = 0; i < lottoNum; i++) {
            Set<Integer> MyLottoNumbers = new HashSet<Integer>(); // 각 로또 번호 배열을 새로 생성
            while (MyLottoNumbers.size() < 6) {
                // A 배열에 각 6개의 랜덤값
                int randomNum = random.nextInt(45) + 1;
                MyLottoNumbers.add(randomNum);
            }
            lottoMapping.put(alphabet.charAt(i), MyLottoNumbers);
        }
    }

    private static void printMyLottoResults(HashMap<Character, Set<Integer>> lottoMapping, Set<Integer> LottoNumbers) {
        System.out.println("\n\n[내 로또 결과]");
        //map에 있는 각 키값에 대한 숫자들하고 로또로 뽑힌 6개의 숫자 비교해서 같은거 count변수에 담아서 출력
        for (char key : lottoMapping.keySet()) {
            int count = 0;
            Set<Integer> myNumbers = lottoMapping.get(key);
            for (Integer num : myNumbers) {
                if (LottoNumbers.contains(num)) {
                    count++;
                }
            }

            String numbers = "";
            Object[] array = myNumbers.toArray();
            Arrays.sort(array);
            for (Object obj : array) {
                Integer num = (Integer) obj;
                if (num < 10) {
                    numbers += "0"; // 한 자리 숫자면 앞에 0 추가
                }
                numbers += num + ",";
            }

            if (!numbers.isEmpty()) {
                numbers = numbers.substring(0, numbers.length() - 1); // 마지막 쉼표 제거
            }

            System.out.printf("%s\t\t %s => %2d개 일치\n", key, numbers, count);
        }
    }

    private static void printLottoNumbers(Set<Integer> LottoNumbers, Random random) {
        System.out.println("\n[로또 발표]");
        while (LottoNumbers.size() < 6) {
            // A 배열에 각 6개의 랜덤값
            int randomNum = random.nextInt(45) + 1;
            LottoNumbers.add(randomNum);
        }

        System.out.print("\t\t");

        Object[] LottoNumbersToArray = LottoNumbers.toArray();
        Arrays.sort(LottoNumbersToArray);

        boolean isFirst = true;

        for (Object obj : LottoNumbersToArray) {
            if (!isFirst) {
                System.out.print(",");
            } else {
                isFirst = false;
            }
            System.out.printf("%02d", obj);
        }
    }

    private static void printMyLottoNumbers(HashMap<Character, Set<Integer>> lottoMapping) {
        for (char key : lottoMapping.keySet()) {
            Object[] array = lottoMapping.get(key).toArray();
            Arrays.sort(array);
            System.out.print(key + "\t\t");
            for (int i = 0; i < array.length; i++) {
                System.out.printf("%02d", array[i]);
                if (i != array.length - 1) {
                    System.out.print(",");
                }
            }
            System.out.println();
        }
    }
}
```
