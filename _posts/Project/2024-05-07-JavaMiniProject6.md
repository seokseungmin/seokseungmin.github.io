---
title: "Java Mini Project6"
excerpt: "[제로베이스] 가상 선거 및 당선 시뮬레이션 프로그램"

categories:
  - Project
tags:
  - [Project]

permalink: /categories/JavaMiniProject6/ # url

toc: true
toc_sticky: true

date: 2024-05-07
last_modified_at: 2024-05-07
---

## 가상 선거 및 당선 시뮬레이션 프로그램
수행 목적 : 조건문 및 반복문과 배열(or 컬렉션)을 통한 당선 시뮬레이션 로직 작성<br>
간략 소개 : 민주주의에서 선거를 대단히 중요한 의사 표현입니다. <br>
이런 선거를 미리 시뮬레이션을 통해서 진행하는 프로그램을 만들어 보고자 합니다. <br>
전체 투표수와 후보자를 입력받아서, 그 결과를 미리 확인하는 선거 및 당선 시뮬레이션 프로그램을 만들어 보세요.<br>

## 필수 준수사항
총 투표를 진행할 투표수를 입력 받음<br>
선거를 진행할 후보자 수를 입력 받고, 이에 대한 이름을 입력 받음<br>
각 입력받은 후보자는 순서대로 기호1, 기호2, 기호3… 형식으로 기호번호 부여함<br>
각 투표수의 결과는 선거를 진행할 후보자를 동일한 비율로 랜덤하게 발생<br>
임의번호는 Random함수의 nextInt()함수를 통해서 생성<br>
1표에 대한 투표한 결과에 대해서 투표자와 이에 대한 결과를 화면 출력해야 함<br>

### 아래 내용은 전제조건으로 진행
투표수는 1 ~ 10000 사이의 값을 입력하며, 그외 값 입력에 대한 예외는 없다고 가정함.<br>
후보자 인원은 2 ~ 10 사이의 값을 입력받으면, 그외 값 입력에 대한 예외는 없다고 가정함.<br>
후보자이름은 한글로 입력하며, 10자 미만으로 입력함. (역시, 그외 입력에 대한 예외는 없다고가정함.)<br>

```java
import java.util.HashMap;
import java.util.InputMismatchException;
import java.util.Random;
import java.util.Scanner;

public class Project6 {
    public static void main(String[] args) {
        getVotingProgress();
    }

    private static void getVotingProgress() {
        int totalVotes = 0;
        int candidatesNum = 0;
        int maxKey = 0;

        Scanner scanner = new Scanner(System.in);

        try {
            // 총 투표수 입력 받기
            System.out.print("총 진행할 투표수를 입력해주세요. ");
            totalVotes = scanner.nextInt();
            if (totalVotes < 1 || totalVotes > 10000) {
                throw new InputMismatchException("투표수는 1 ~ 10000 사이의 값을 입력해주세요.");
            }

            // 후보자 인원 입력 받기
            System.out.print("가상 선거를 진행할 후보자 인원을 입력해 주세요. ");
            candidatesNum = scanner.nextInt();
            if (candidatesNum < 2 || candidatesNum > 10) {
                throw new InputMismatchException("후보자 인원은 2 ~ 10 사이의 값을 입력해주세요.");
            }

            // 후보자 이름 입력 받기
            HashMap<Integer, String> candidateMapping = new HashMap<>();
            for (int i = 1; i <= candidatesNum; i++) {
                System.out.print(i + "번째 후보자 이름을 입력해 주세요: ");
                String candidateName = scanner.next();
                if (!candidateName.matches("^[가-힣]{1,10}$")) {
                    throw new InputMismatchException("후보자 이름은 한글로 입력하며, 10자 미만으로 입력해주세요.");
                }
                candidateMapping.put(i, candidateName);
            }

            Random random = new Random();

            // 각 후보별 투표수 저장
            HashMap<Integer, Integer> voteResults = new HashMap<>();

            // 투표 진행률 계산 및 출력
            for (int i = 1; i <= totalVotes; i++) {
                double progress = ((double) i / totalVotes) * 100;
                int randomCandidateNum = random.nextInt(candidatesNum) + 1; // 후보자 번호는 1부터 시작하므로 +1
                String randomCandidate = candidateMapping.get(randomCandidateNum);

                System.out.println("[투표진행률]: " + progress + "%, " + i + "명 투표 => " + randomCandidate);

                // 해당 후보자의 투표 수 증가
                voteResults.put(randomCandidateNum, voteResults.getOrDefault(randomCandidateNum, 0) + 1);

                for (int key : candidateMapping.keySet()) {
                    int votes = voteResults.getOrDefault(key, 0);
                    double votePercentage = (double) votes / totalVotes * 100;

                    System.out.printf("[기호:%d] %s:", key, candidateMapping.get(key)); //기호와 이름 출력

                    if (candidateMapping.get(key).length() <= 2) {  //출력화면에서 간격을 유지하기 위한 조건문
                        System.out.print("\t\t");
                    } else if (candidateMapping.get(key).length() < 4) {
                        System.out.print("\t");
                    }

                    System.out.printf("\t%.2f%% \t (투표수: %d)\n", votePercentage, votes); //개인별 비율과 투표수 출력
                }

                int max = Integer.MIN_VALUE;

                for (int key : voteResults.keySet()) {
                    if (voteResults.get(key) > max) {
                        max = voteResults.get(key);
                        maxKey = key;
                    }
                }

                System.out.println();
            }

            System.out.println("[투표결과] 당선인 : " + candidateMapping.get(maxKey));

        } catch (InputMismatchException e) {
            System.out.println(e.getMessage());
            System.out.println("처음부터 다시 입력해주세요.");
            scanner.nextLine(); // 입력 버퍼 비우기
            getVotingProgress(); // 예외 발생 시 다시 투표 진행 메서드 호출
        } finally {
            scanner.close(); // 사용한 scanner 객체 닫기
        }
    }
}
```
