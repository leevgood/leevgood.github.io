알고리즘-테스트-The-candy-war
---

<br>

### 문제
https://www.acmicpc.net/problem/9037

<br>

### 코드(본인작성)

```java
import java.io.*;
import java.util.*;


public class Main {
    static int n;


    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        StringTokenizer st;

        int t = Integer.parseInt(br.readLine());
        StringBuilder sb = new StringBuilder();

        for (int a = 0; a < t; a++) {
            //반복
            st = new StringTokenizer(br.readLine());
            n = Integer.parseInt(st.nextToken());
            boolean isSame = false;

            st = new StringTokenizer(br.readLine());

            int[] child = new int[n];
            int[] give = new int[n];

            //아이 초기화
            for (int i = 0; i < n; i++) {
                child[i] = Integer.parseInt(st.nextToken());

                //홀수 개면 한개 더 줌
                if (child[i] % 2 == 1) {
                    child[i]++;
                }
            }

            int count = 0;
            while (true) {

                //모든 사탕 개수 같으면 스트링빌더에 더하고 아니면 카운트 증가하고 반복
                int sameCount = 0;
                for (int i = 0; i < child.length - 1; i++) {
                    if (child[i] != child[i + 1]) {
                        continue;
                    }
                    sameCount++;
                }

                if (sameCount == child.length - 1) {
                    isSame = true;
                }

                if (isSame == true) {
                    sb.append(count+"\n");
                    break;
                } else {
                    count++;
                }

                //주고
                for (int i = 0; i < n; i++) {
                    give[i] = child[i] / 2;
                    child[i] -= give[i];
                }

                //받고
                child[0] += give[n - 1];
                for (int i = 1; i < child.length; i++) {
                    child[i] += give[i - 1];
                }

                //사탕이 홀수개면 한개 추가
                for (int i = 0; i < child.length; i++) {
                    if (child[i] % 2 == 1) {
                        child[i]++;
                    }
                }
            }

        }

        System.out.println(sb.toString());
    }
}




```


<br>

### 코멘트

- 단순구현

<br>

### 출처
