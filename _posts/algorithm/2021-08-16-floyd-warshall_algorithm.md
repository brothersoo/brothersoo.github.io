---
title: "최단 경로 찾기 알고리즘(3) - 플로이드 와샬(Floyd-Warshall)"
categories:
  - algorithm
tags:
  - graph
  - shortest path
toc: true
permalink: /algorithm/:title
---
<br>

# 플로이드 와샬 알고리즘

플로이드 와샬 알고리즘은 다익스트라 알고리즘, 벨만 포드와 같이 정점간의 최단거리를 찾기 위해 고안된 알고리즘 입니다.

그러나 다른 두 알고리즘과는 달리 플로이드 와샬 알고리즘은 한 정점으로부터가 아닌 모든 정점으로부터 모든 정점까지의 최단거리를 구하는 알고리즘입니다.

이것만 보아도 꽤나 큰 시간 복잡도를 가질 것 같다는 느낌이 들죠?^^*
<br>
<br>
<br>
<br>

# 개념

플로이드 와샬 알고리즘의 원리는 굉장히 간단합니다.

현재 계산된 한 정점 S로부터 다른 한 정점 E로 가는 것의 비용보다

또 다른 어떠한 정점 M을 거쳐 가는 것의 비용이 적은 경우

S -> E을 S -> M -> E로 변경하기만 하면됩니다.

거쳐가는 노드 M, 시작하는 노드 S, 도착하는 노드 E, 모두 N개의 노드를 한번씩 지정해야 하므로

 시간복잡도는 O(\|V\|^3)이 됩니다.

`|V| : 정점의 갯수`
<br>
<br>
<br>
<br>

# 구현 코드

플로이드 와샬 알고리즘은 [백준 11404 플로이드](https://www.acmicpc.net/problem/11404) 문제를 통해 구현해보았습니다.

```java
// 플로이드

package BOJ;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;

public class BOJ11404 {

  static int MAX = 1000000001;

  static String 플로이드_와샬(int[][] 비용, int 도시수) {

    for (int 거치는도시 = 0; 거치는도시 < 도시수; 거치는도시++) {
      for (int 시작도시 = 0; 시작도시 < 도시수; 시작도시++) {
        for (int 도착도시 = 0; 도착도시 < 도시수; 도착도시++) {
          if (비용[시작도시][거치는도시] + 비용[거치는도시][도착도시] < 비용[시작도시][도착도시]) {
            비용[시작도시][도착도시] = 비용[시작도시][거치는도시] + 비용[거치는도시][도착도시];
          }
        }
      }
    }

    // 플로이드 와샬 알고리즘은 위에서 끝!

    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < 비용.length; i++) {
      for (int j = 0; j < 비용[i].length; j++) {
        sb.append(((비용[i][j] == MAX) ? 0 : 비용[i][j]) + " ");
      }
      sb.deleteCharAt(sb.length() - 1);
      sb.append("\n");
    }
    return sb.toString();
  }

  public static void main(String[] args) throws IOException {
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));

    int n = Integer.parseInt(br.readLine());

    int[][] map = new int[n][n];
    for (int i = 0; i < n; i++) {
      for (int j = 0; j < n; j++) {
        map[i][j] = (i == j) ? 0 : MAX;
      }
    }
    int m = Integer.parseInt(br.readLine());
    for (int i = 0; i < m; i++) {
      String[] abc = br.readLine().split(" ");
      int a = Integer.parseInt(abc[0]) - 1;
      int b = Integer.parseInt(abc[1]) - 1;
      int c = Integer.parseInt(abc[2]);

      if (c < map[a][b]) map[a][b] = c;
    }

    bw.write(플로이드_와샬(map, n));
    bw.flush();
  }
}
```

<br>
<br>

주의해야 했던 점은 배열을 모두 최대값으로 지정해야하는데 이를 Integer.MAX_VALUE와 같은 타입 최댓값을 넣으면 안된다는 점입니다.

Integer.MAX_VALUE + 1은 Integer의 최소값이 되어버립니다.

`시작 도시`에서 `거치는 도시`로 가는 경로가 없어 `비용[시작도시][거치는도시] = Integer.MAX_VALUE`이고

`거치는 도시`에서 `도착 도시`로 가는 경로의 비용은 1인 경우에

`비용[시작도시][거치는도시] + 비용[거치는도시][도착도시]`는 Integer의 최소값이 되어버립니다.

결국 존재하는 경로가 없으므로 false가 되어야하는

`비용[시작도시][거치는도시] + 비용[거치는도시][도착도시] < 비용[시작도시][도착도시]` 가

true 가 되어버릴 수 있습니다.
<br>
<br>
<br>
<br>

<details>
<summary>제가 했던 바보같은 실수</summary>
<div markdown="1">

<br>

문제를 보면 `시작 도시와 도착 도시를 연결하는 노선은 하나가 아닐 수 있다.` 라는 문장이 있습니다.

무조건 경로의 최소값을 보여주면 되겠다 싶어 비용 표를 `PriorityQueue<Integer>[n][n]`로 생성하려 했습니다.

구현하면서도 메모리 사용이 장난이 아니겠는데? 생각했습니다.

그냥 저장할때 작은 경우에만 저장하면 됐습니다^^;

</div>
</details>

<br>