---
title: "최단 경로 찾기 알고리즘(1) - 다익스트라(Dijkstra)"
categories:
  - algorithm
tags:
  - graph
  - shortest path
toc: true
permalink: /algorithm/:title
---
<br>

# 최단 경로 알고리즘

알고리즘 문제를 풀다보면 빈번히 나오는 유형 중 하나가 바로 최단 경로 찾기입니다.

최단경로 찾기 알고리즘 중 대표적으로 쓰이는 세 알고리즘은 다익스트라(Dijkstra) 알고리즘, 벨만 포드(Bellman Ford) 알고리즘, 그리고 플로이드 와샬(Floyd Warshall) 알고리즘 입니다.
<br>
<br>
<br>
<br>

# 다익스트라 알고리즘

네비게이션 길 찾기 등 실제 자주 사용되는 다익스트라 알고리즘은 한 정점으로부터 다른 모든 정점까지의 최단 경로를 찾는 알고리즘입니다.

값이 음수인 간선이 존재해서는 안됩니다. (음수가 있다면 벨만 포드 알고리즘을 사용해야 합니다.)

다익스트라 알고리즘의 가장 핵심적인 내용은 다음과 같습니다.

> S: 원점\
> N, M: 어떤 연결된 두 노드
>
> 현재 밝혀진 S -> N으로 가는 가장 짧은 경로보다\
> S -> M으로 가는 가장 짧은 경로 더하기\
> M에서 N으로 가는 경로의 길이가 더 짧으면\
> S -> N으로 가는 가장 짧은 길은 S -> M -> N 이 되는 것입니다.

```
if ((S -> M) + (M -> N) < S -> N):
    S -> N = S -> M -> N
```

최대한 이해가 잘 되도록 써보려 노력했지만 난해한 느낌이네요^^;
<br>
<br>
<br>
<br>

# 시간복잡도

<details>
<summary>아직 공부가 더 필요합니다^^; 이해가 잘 안되는 부분이 너무 많네요^^* (보지마세요! 잘못된 내용이 그득합니다!)</summary>
<div markdown="1">


  현재 노드까지로의 최소 거리를 구하고 다음 노드로 넘어갈 때, 다음 노드를 선정하는 기준은 아래와 같습니다.

      아직 방문하지 않은 노드
      방문된 노드들에 연결된 간선 중 가중치가 가장 작은 간선에 연결되어있는 노드

  <br>

  <details>
  <summary>가중치가 가장 작은 간선을 선택하는 이유</summary>
  <div markdown="1">

  가중치가 가장 작은 간선을 찾는데 O(V)를 써버리니 왜 가장 작은 가중치를 찾는가 궁금해졌습니다.

  물론 제가 수학적 증명을 할 수는 없으니 검색을 해보았습니다^^*

  [여기 저랑 똑같은 의문을 가진 사람이 약 11년전 질문글을 stack overflow에 올렸었네요 ^^*](https://stackoverflow.com/questions/2856670/why-does-dijkstras-algorithm-work)

  이유는 간단하고 당연했습니다.

  다익스트라 알고리즘은 최단 경로를 찾는 알고리즘입니다.

  어떤 정점 N으로 가는 간선 중 최단 길이가 아닌 다른 간선을 택한다면, 이후 해당 정점 N을 거치는 모든 경로는 최단 거리가 될 수 없습니다.


  </div>
  </details>
  <br>
  <br>

  [위키피디아 다익스트라 알고리즘의 Running time 문단](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm#Running_time)을 보면

  어떠한 자료구조를 사용하던 다익스트라 알고리즘의 시간복잡도는 다음과 같습니다.

  ## O(|E|*Tdk + |V|*Tem)
  <br>
  ```
  |E| : 정점의 수
  |V|: 간선의 수
  Tdk: 키 감소 복잡도
  Tem: 최소값 찾기 복잡도
  ```

  ### 인접 행렬로 구현된 그래프

  최소 힙을 사용하지 않는다면 키 감소에 대한 복답도(Tdk)는 신경쓰지 않아도 됩니다.

  Tdk = 1 이고 최소값을 찾기 위해서는 모든 정점을 검색해야 하므로 Tem = |V|가 될 것입니다.

  최소값을 찾는 복잡도가 |V|일 수 있는 이유는 아래 구현 코드를 보면 알겠지만

  항상 시작점에서부터 다른 모든 점까지의 최단 거리를 저장하는 배열(int[] 최단경로)을 가지고 있습니다.

  boolean[] visited을 통해 방문하지 않은 노드인지 확인므로 최단경로 배열만 한번 순회하면

  방문하지 않은

  즉, O(|E|*1 + |V|*|V|) = O(|V|^2)의 복잡도를 가지게 됩니다.
  <br>
  <br>

  ### 인접 리스트로 구현된 그래프

  마찬가지로 최소 힙을 사용하지 않으므로 Tdk = 1입니다.


  <br>
  <br>
  <br>

  ### 최소 힙을 사용한 알고리즘

  여기서 한차례 더 시간복잡도를 낮출 수 있는데 바로 최소 힙을 사용하는 방법입니다.
  <br>

  매 방문마다 최소값을 확인하는 O(V) 혹은 O(E)를 줄일 수 있게 됩니다.

  모든 노드를 방문하는 것은 마찬가지로 O(V),

  한 방문마다 연결된 간선을 최소 힙에 삽입할 것이므로 삽입에 O(MlogE),\
  (M은 해당 노드에 연결된 간선의 수, (M(1) + M(2) + ... + M(V)) = E)

  즉 싸이클은 총 V번 돌고 매 싸이클마다 최소 힙으로의 삽입을 M번씩 하지만 결국엔 삽입을 고정된 E번 하므로

  O(V) + O(ElogE) = O(V + ElogE)의 복잡도가 될 것입니다.

  보통의 경우에는 간선의 수가 정점의 수보다 많으므로 O(ElogV)라고 생각하면 되고

  정점은 많지만 정점 사이가 많이 연결되지 않은 경우는 O(VlogV)라고 보면 될 것 같습니다.
  <br>
  <br>

  ~~인접행렬에서 최소 힙을 사용한다면 V번의 싸이클 내에서 V번의 삽입이 이루어질 것이므로 O(V^2logE)의 복잡도를 가지게 될 것입니다.~~
  <br>
  <br>
  <br>

  ## 정리
  <br>

  | Time Complexity |    인접 행렬   |    인접리스트   |
  |:---------------:|:------------:|:------------:|
  |   일반 이중 루프    |    O(V^2)   |     O(VE)    |
  |     최소 힙       |~~O(V^2logE)~~| O(V + ElogE) |

  `V : 간선의 수`\
  `E : 노드의 수`
  <br>
  <br>
  <br>
  <br>

</div>
</details>
<br>
<br>

# 사용 가능한 조건

다익스트라 알고리즘을 사용할 수 있는 그래프는 아래 세 조건을 만족해야 합니다.

1. 유향 그래프이어야 한다. (Directed)
2. 비순환 그래프이어야 한다. (Acyclic)
3. 음수 가중치를 가지는 간선이 있어서는 안된다.
<br>
<br>
<br>
<br>

# 구현 코드

백준 사이트에 있는 가장 대표적인 다익스트라 문제 18352 최단 경로를
1. 인접 행렬
2. 인접 행렬과 우선순위큐
3. 인접리스트
4. 인접리스트와 우선순위큐

를 사용하여 각각 구현해보았습니다.

주석을 달지 않아보고자 한글로 코딩을 했는데 익숙치 않네요^^*

주의! 아래 답안들을 사용하시면 시간 효율이 조금 떨어질 수 있습니다.

메인 함수에서 인접행렬과 인접 리스트를 모두 생성하고보므로 메모리 제한에 주의하세요^0^

~~사실 인접행렬을 사용한 두 답안은 모두 메모리 초과에 걸려서 정확히 맞는 답인지 모릅니다^^;~~


<br>

<details>
<summary>구현 코드</summary>
<div markdown="1">

<br>

```java
// 최단경로

package BOJ;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.PriorityQueue;

public class BOJ18352 {

  static int 무한대 = Integer.MAX_VALUE;

  //============================== 인접행렬 ==============================//

  static String 인접행렬(int 정점수, int 간선수, int 시작점, int[][] 가중치) {
    int[] 최단경로 = new int[정점수];
    Arrays.fill(최단경로, 무한대);
    최단경로[시작점] = 0;
    boolean[] 방문함 = new boolean[정점수];
    방문함[시작점] = true;

    int 현재정점 = 시작점;
    for (int i = 0; i < 정점수; i++) {
      int 최소값 = 무한대;
      for (int 번호 = 0; 번호 < 정점수; 번호++) {
        if (방문함[번호]) continue;
        if (최단경로[번호] >= 최소값) continue;
        현재정점 = 번호;
        최소값 = 최단경로[번호];
      }
      방문함[현재정점] = true;

      for (int 연결된정점 = 0; 연결된정점 < 정점수; 연결된정점++) {

        if (방문함[연결된정점]) continue;
        if (가중치[현재정점][연결된정점] == 0) continue;
        if (최단경로[현재정점] + 가중치[현재정점][연결된정점] >= 최단경로[연결된정점]) continue;

        최단경로[연결된정점] = 최단경로[현재정점] + 가중치[현재정점][연결된정점];
      }
    }

    StringBuilder 결과 = new StringBuilder();
    for (int 정점번호 = 0; 정점번호 < 정점수; 정점번호++) {
      결과.append(((최단경로[정점번호] == 무한대) ? "INF" : 최단경로[정점번호]) + "\n");
    }
    return 결과.toString();
  }
  //=====================================================================//





  //=========================== 인접행렬과 최소힙 ===========================//

  static String 인접행렬_최소힙(int 정점수, int 간선수, int 시작점, int[][] 가중치) {
    int[] 최단경로 = new int[정점수];
    Arrays.fill(최단경로, 무한대);
    최단경로[시작점] = 0;
    PriorityQueue<정점> 힙 = new PriorityQueue<>();
    힙.add(new 정점(시작점, 0));

    while (!힙.isEmpty()) {
      정점 현재정점 = 힙.remove();

      if (최단경로[현재정점.번호] < 현재정점.가중치) continue;

      for (int 연결된정점 = 0; 연결된정점 < 정점수; 연결된정점++) {
        if (가중치[현재정점.번호][연결된정점] == 0) continue;
        if (최단경로[현재정점.번호] + 가중치[현재정점.번호][연결된정점] >= 최단경로[연결된정점]) continue;

        최단경로[연결된정점] = 최단경로[현재정점.번호] + 가중치[현재정점.번호][연결된정점];
        힙.add(new 정점(연결된정점, 최단경로[연결된정점]));
      }
    }

    StringBuilder 결과 = new StringBuilder();
    for (int 정점번호 = 0; 정점번호 < 정점수; 정점번호++) {
      결과.append(((최단경로[정점번호] == 무한대) ? "INF" : 최단경로[정점번호]) + "\n");
    }
    return 결과.toString();
  }
  //=====================================================================//





  //============================= 인접리스트 ==============================//

  static String 인접리스트(int 정점수, int 간선수, int 시작점, List<정점>[] 가중치) {
    int[] 최단경로 = new int[정점수];
    Arrays.fill(최단경로, 무한대);
    최단경로[시작점] = 0;
    boolean[] 방문함 = new boolean[정점수];
    방문함[시작점] = true;

    int 현재정점 = 시작점;
    for (int i = 0; i < 정점수; i++) {
      int 최소값 = 무한대;
      for (int 번호 = 0; 번호 < 정점수; 번호++) {
        if (방문함[번호]) continue;
        if (최단경로[번호] >= 최소값) continue;
        현재정점 = 번호;
        최소값 = 최단경로[번호];
      }
      방문함[현재정점] = true;

      for (정점 연결된정점 : 가중치[현재정점]) {
        if (방문함[연결된정점.번호]) continue;
        if (최단경로[현재정점] + 연결된정점.가중치 >= 최단경로[연결된정점.번호]) continue;

        최단경로[연결된정점.번호] = 최단경로[현재정점] + 연결된정점.가중치;
      }
    }

    StringBuilder 결과 = new StringBuilder();
    for (int 정점번호 = 0; 정점번호 < 정점수; 정점번호++) {
      결과.append(((최단경로[정점번호] == 무한대) ? "INF" : 최단경로[정점번호]) + "\n");
    }
    return 결과.toString();
  }
  //==================================================================//





  //========================= 인접리스트와 최소힙 =========================//

  static String 인접리스트_최소힙(int 정점수, int 간선수, int 시작점, List<정점>[] 가중치) {
    int[] 최단경로 = new int[정점수];
    Arrays.fill(최단경로, 무한대);
    최단경로[시작점] = 0;
    PriorityQueue<정점> 힙 = new PriorityQueue<>();
    힙.add(new 정점(시작점, 0));
//    boolean[] 방문함 = new boolean[정점수];

    while (!힙.isEmpty()) {
      정점 현재정점 = 힙.remove();

      if (현재정점.가중치 > 최단경로[현재정점.번호]) continue;
//      if (방문함[현재정점.번호]) continue;
//      방문함[현재정점.번호] = true;

      for (정점 연결된정점 : 가중치[현재정점.번호]) {
        if (최단경로[현재정점.번호] + 연결된정점.가중치 < 최단경로[연결된정점.번호]) {
          최단경로[연결된정점.번호] = 최단경로[현재정점.번호] + 연결된정점.가중치;
          힙.add(new 정점(연결된정점.번호, 최단경로[연결된정점.번호]));
        }
      }
    }

    StringBuilder 결과 = new StringBuilder();
    for (int 정점번호 = 0; 정점번호 < 정점수; 정점번호++) {
      결과.append(((최단경로[정점번호] == 무한대) ? "INF" : 최단경로[정점번호]) + "\n");
    }
    return 결과.toString();
  }
  //==================================================================//

  static class 정점 implements Comparable<정점> {
    int 번호;
    int 가중치;

    public 정점(int 번호, int 가중치) {
      this.번호 = 번호;
      this.가중치 = 가중치;
    }

    @Override
    public int compareTo(정점 상대정점) {
      return this.가중치 - 상대정점.가중치;
    }
  }


  public static void main(String[] args) throws IOException {
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));

    String[] VE = br.readLine().split(" ");
    int V = Integer.parseInt(VE[0]);
    int E = Integer.parseInt(VE[1]);
    int K = Integer.parseInt(br.readLine()) - 1;

    int[][] matrixMap = new int[V][V];
    List<정점>[] listMap = new ArrayList[V];
    for (int i = 0; i < V; i++) {
      listMap[i] = new ArrayList<>();
    }
    for (int i = 0; i < E; i++) {
      String[] uvw = br.readLine().split(" ");
      int u = Integer.parseInt(uvw[0]) - 1;
      int v = Integer.parseInt(uvw[1]) - 1;
      int w = Integer.parseInt(uvw[2]);
      matrixMap[u][v] = w;
      listMap[u].add(new 정점(v, w));
    }

     bw.write(인접행렬(V, E, K, matrixMap));
     bw.write(인접행렬_최소힙(V, E, K, matrixMap));

     bw.write(인접리스트(V, E, K, listMap));
    bw.write(인접리스트_최소힙(V, E, K, listMap));
    bw.flush();
  }
}
```

</div>
</details>
<br>
<br>