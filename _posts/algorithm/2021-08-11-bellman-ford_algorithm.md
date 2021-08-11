---
title: "최단 경로 찾기 알고리즘(2) - 벨만 포드(Bellman-Ford)"
categories:
  - algorithm
tags:
  - graph
  - shortest path
toc: true
permalink: /algorithm/:title
---
<br>

# Bellamn-Ford

Bellman-Ford's algorithm의 이름은 알고리즘을 발표한 `Richard Bellman`과 `Lester Ford Jr.`의 이름을 따와서 만들어졌습니다.

<details>
<summary>[TMI ALERT!] 이름에 대한 유래 </summary>
<div markdown="1">

---

그러나 [위키피디아](https://en.wikipedia.org/wiki/Bellman%E2%80%93Ford_algorithm)를 보면 벨만 포드 알고리즘을 처음 고안한 사람은 `Richard Bellman`도, `Lester Ford Jr.` 도 아닌 바로 `Alfonso Shimbel`이라고 합니다.

1954년 Shimbel의 고안 이후 1957년 Edward Moore가 더욱 정교화를 진행했습니다.

그래서 Bellman-Ford 알고리즘은 때때로 Bellman-Ford_Moore 알고리즘으로도 불린다고 합니다.

또 1957년 Max Woodbury와 George Dantzig 또한 알고리즘에 대한 연구를 진행했지만 발표는 하지 않았다고 합니다.

1958년 드디어 Richard Bellman이 해당 알고리즘을 발표했습니다.

Bellman's algorithm이 아닌 Bellman-Ford's algorithm 이라고 불리는 이유는 Bellman이 Ford의 [`relaxing edge`](https://towardsdatascience.com/algorithm-shortest-paths-1d8fa3f50769) 탐색 공식을 사용했기 떄문이라고 합니다.

Bellman-Ford's algorithm은 때때로 Bellman-Kalaba's algorithm 혹은 Bellman-Shimbel's algorithm 이라고도 불린다고 합니다.

<br>

참고 자료

[Wikipedia: Bellman–Ford algorithm](https://en.wikipedia.org/wiki/Bellman%E2%80%93Ford_algorithm)

[StackExchange: Why is the Bellman-Ford's shortest path algorithm sometimes called Bellman-Kalaba?](https://or.stackexchange.com/questions/6476/why-is-the-bellman-fords-shortest-path-algorithm-sometimes-called-bellman-kalab)

---

</div>
</details>
<br>
<br>


벨만 포드 알고리즘은 다익스트라 알고리즘과 마찬가지로 한 정점으로부터 다른 모든 정점까지의 최단거리를 찾는 알고리즘입니다.

하지만 성능은 다익스트라 알고리즘보다 조금 떨어진다고 합니다.

그대신 다익스트라 알고리즘은 음수 가중치를 가지는 간선이 있는 그래프에는 적용할 수 없지만

벨만 포드 알고리즘은 음수의 간선이 존재하는 그래프에서도 최단거리를 찾을 수 있다는 장점이 있습니다.
<br>
<br>
<br>
<br>

# 알고리즘

벨만포드 알고리즘의 핵심 내용은 다음과 같습니다.

    S: 시작 정점 (Start)
    CV: 어떤 간선 (Current Vertex)
    FP: CV의 출발 정점까지의 최단거리 (From Path)
    TP: CV의 도착 정점까지의 최단거리 (To Path)
    W: CV의 가중치 (Weight)
    V: 총 정점의 갯수 (Vertex)
    E: 총 간선의 갯수 (Edge)

1. N - 1번의 순회에서 각 순회마다 모든 간선을 확인한다.
2. F가 한번이라도 방문한 정점인지 확인하고, 방문하지 않은 정점이라면 다음 간선으로 넘어간다.
3. 다익스트라와 비슷하게 `FP + W < TP 라면 TP = FP + W`를 적용한다.
4. 3이 적용되는 T를 방문처리한다.
5. 마지막으로 한번 더 모든 간선에 대해 2, 3, 4를 처리한다.

기본적으로 N - 1번만 모든 간선을 확인하면 되지만 마지막 한번 더 진행하는 이유는 음수 싸이클을 확인하기 위해서 입니다.

싸이클을 이루는 간선 중 음수의 가중치를 가지는 간선이 있다면

싸이클 내의 정점들까지의 최단 경로 비용은

음수 무한대로 발산할것입니다.

결국 음수 싸이클이 존재하는지 알기 위해서 완성작이라고 확신하는 N - 1번의 순회 후 생성된 최단경로표에서

한번 더 작업을 수행했을 시 변동이 있다면 음수 싸이클이 존재한다는 뜻입니다.
<br>
<br>
<br>
<br>

# 구현 코드

간단히 벨만포드 알고리즘만으로 해결할 수 있는 백준 사이트의 타임머신(11657번) 문제로 벨만 포드 알고리즘을 연습해보았습니다.

```java
// 타임머신

package BOJ;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.util.Arrays;

public class BOJ11657 {

  static String 벨만포드(int 도시수, int 노선수, 노선[] 노선도) {
    long[] 최단경로 = new long[도시수];
    Arrays.fill(최단경로, 무한대);
    최단경로[시작도시] = 0;
    boolean[] 방문함 = new boolean[도시수];
    방문함[시작도시] = true;

    for (int 반복수 = 1; 반복수 <= 노선수; 반복수++) {
      for (노선 현재노선 : 노선도) {
        if (!방문함[현재노선.출발도시]) continue;
        // if (최단경로[현재노선.출발도시] == 무한대) continue;

        if (최단경로[현재노선.출발도시] + 현재노선.가중치 < 최단경로[현재노선.도착도시]) {
          if (반복수 == 노선수) return "-1";
          최단경로[현재노선.도착도시] = 최단경로[현재노선.출발도시] + 현재노선.가중치;
          방문함[현재노선.도착도시] = true;
        }
      }
    }

    StringBuilder 결과 = new StringBuilder();
    for (int 도시 = 1; 도시 < 도시수; 도시++) {
      결과.append(((최단경로[도시] == 무한대) ? "-1" : 최단경로[도시]) + "\n");
    }
    return 결과.deleteCharAt(결과.length() - 1).toString();
  }

  public static void main(String[] args) throws IOException {
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));

    String[] NM = br.readLine().split(" ");
    int N = Integer.parseInt(NM[0]);
    int M = Integer.parseInt(NM[1]);

    노선[] map = new 노선[M];

    for (int i = 0; i < M; i++) {
      String[] ABC = br.readLine().split(" ");
      int A = Integer.parseInt(ABC[0]) - 1;
      int B = Integer.parseInt(ABC[1]) - 1;
      int C = Integer.parseInt(ABC[2]);

      map[i] = new 노선(A, B, C);
    }

    bw.write(벨만포드(N, M, map));
    bw.flush();
  }

  static final int 무한대 = Integer.MAX_VALUE;
  static final int 시작도시 = 0;

  static class 노선 {
    int 출발도시;
    int 도착도시;
    int 가중치;

    public 노선(int 출발도시, int 도착도시, int 가중치) {
      this.출발도시 = 출발도시;
      this.도착도시 = 도착도시;
      this.가중치 = 가중치;
    }
  }
}
```

저는 직관적으로 방문을 하였는것을 보기 위해 boolean[] 방문함 배열을 생성하였지만

대부분의 구현하신 코드를 보면 boolean 배열을 따로 생성하지 않고

최단경로 배열의 값이 무한대일 경우 아직 방문하지 않은 도시라고 판단하는 것으로 보였습니다.

따로 boolean 배열을 생성하지 않는 것이 메모리를 조금 더 아낄 수 있겠죠?^^*