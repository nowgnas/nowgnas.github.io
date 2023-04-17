---
title: 그래프 자료구조 정리 (프림, 크루스칼, 다익스트라)
categories: algorithm
date: 2022-08-27 14:56 +0900
description: 프림, 크루스칼, 다익스트라 알고리즘을 자바로 구현해보자
lastmod: 2022-08-27 14:56 +0900
tags: algorithm dijkstra java kruskal prim
---

# Prim 알고리즘

프림 알고리즘은 무향 그래프에서 MST(최소 스패닝 트리)를 찾는 알고리즘이다. 시작점에서 정점을 추가해 가면서 트리를 확장한다.

## 동작

정점 탐색 시 인접 정점 중 비용이 가장 작은 간선으로 연결된 정점을 선택해 연결한다

1. 시작 정점을 MST에 추가한다
2. MST 집합에 인접한 노드 중 최소 비용을 가지는 간선으로 연결된 노드를 선택, MST에 추가한다
3. MST가 정점개수 - 1 개의 간선을 가질 때까지 반복한다

프림 알고리즘은 인접 행렬과 우선순위 큐를 사용해서 구현할 수 있다. 이번 포스팅에서는 SWEA 1251 하나로 문제로 우선순위 큐를 사용한 프림 알고리즘 코드를 구현해 본다

[하나로 문제 보러가기](https://swexpertacademy.com/main/code/problem/problemDetail.do?contestProbId=AV15StKqAQkCFAYD&categoryId=AV15StKqAQkCFAYD&categoryType=CODE&problemTitle=하나로&orderBy=FIRST_REG_DATETIME&selectCodeLang=ALL&select-1=&pageSize=10&pageIndex=1)

문제에서 요구하는 것은 환경 부담금을 최소로 하는 섬들의 연결을 찾아 총 환경 부담금을 찾는 문제였다. 다른 그래프 문제와 다른 점은 정점들 사이에 간선을 직접 구해야 한다. 하나의 정점과 나머지 정점 사이의 모든 간선을 고려해서 MST를 생성해야 하는 문제였다.

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.*;

public class Solution {
    static int tc, N;
    static double E;

    static double[][] matrix;
    static boolean[] visited;

    static class Node {
        int no;
        double weight;

        public Node(int no, double weight) {
            this.no = no;
            this.weight = weight;
        }
    }

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

        tc = Integer.parseInt(br.readLine());
        matrix = new double[2][N];

        for (int i = 1; i <= tc; i++) {
            N = Integer.parseInt(br.readLine()); // 노드 개수
            for (int j = 0; j < 2; j++) { // 각 노드 입력 받기
                matrix[j] = Arrays.stream(br.readLine().split(" ")).mapToDouble(Double::parseDouble).toArray();
            }
            E = Double.parseDouble(br.readLine()); // 가중치
            // 입력 끝

            List<Node>[] queues = new ArrayList[N];
            visited = new boolean[N];

            for (int j = 0; j < N; j++) {
                queues[j] = new ArrayList<>();
                for (int k = 0; k < N; k++) {
                    double dist = weightCalculate(matrix[0][j], matrix[1][j], matrix[0][k], matrix[1][k]); // 간선 값
                    queues[j].add(new Node(k, dist));
                }
            } // 각 점에 모든 점과의 거리 저장

            Queue<Node> queue = new PriorityQueue<>((o1, o2) -> (int) (o1.weight - o2.weight));
            queue.offer(new Node(0, 0));
            int cnt = 0;
            double result = 0;
            while (!queue.isEmpty()) {
                // 신장 트리의 구성에 포함되지 않은 정점 중 최소 비용 선택
                Node miniVertex = queue.poll();

                if (visited[miniVertex.no]) continue; // 방문 되었으면 다음 값 보기

                // 신장 트리에 추가
                visited[miniVertex.no] = true; // 방문 처리
                result += miniVertex.weight; // 가중치 더해주기
                if (++cnt == N) break; // 모든 노드를 확인 했다

                // 신장 트리에 새롭게 추가되는 정점과 신장트리에 포함되지 않은 정점들의 기존 최소 비용과 비교
                for (int j = 0; j < queues[miniVertex.no].size(); j++) {
                    Node tmp = queues[miniVertex.no].get(j);
                    if (!visited[tmp.no]) {
                        queue.add(tmp);
                    }
                }
            }
            System.out.printf("#%d %.0f\n", i, result * E);
        }
    }

    static double weightCalculate(double x, double y, double x2, double y2) {
        double X = Math.abs(x - x2);
        double Y = Math.abs(y - y2);
        return X * X + Y * Y;
    }
}
```

정점 개수 만큼의 연결리스트를 만들어 주어 하나의 정점과 나머지 정점 사이의 거리를 모두 계산해 준다.

- MST를 위한 우선 순위 큐를 생성해 주고 0번 노드를 추가한 후 진행한다.
- 방문한 노드 번호는 방문 처리를 해주고 가중치에 더해준다. 노드의 개수가 만족되면 종료해 준다.
- 현재 노드 번호의 연결 리스트에 담겨 있는 노드들을 확인하면서 방문되지 않은 노드들을 추가해 준다.
- 모든 노드들을 확인하게 되면 가중치의 최소 값이 나오게 된다.

# Dijkstra 알고리즘

다익스트라 알고리즘은 DP를 사용하는 최단 경로 알고리즘이다. 최단 거리를 구하기 위해 이전 값을 사용하는 특징을 가지고 있다.

## 동작

1. 시작 노드를 정한다
2. 시작 노드를 기준으로 인접 노드의 최소 비용을 저장한다
3. 방문하지 않은 노드 중에서 가장 적은 비용을 가진 노드를 선택한다
4. 해당 노드를 거쳐 다음 노드로 가는 경우의 최소 비용을 갱신한다
5. 3과 4를 반복한다

다익스트라 알고리즘을 사용하여 풀었던 백준 4485 녹색 옷 입은 애가 젤다지? 문제를 풀어본다.

[녹색 옷 입은 애가 젤다지? 문제 보러가기](https://www.acmicpc.net/problem/4485)

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Arrays;
import java.util.PriorityQueue;
import java.util.Queue;
import java.util.StringTokenizer;

public class Main {
    static int N, tc, X, Y;
    static int[][] map;
    static int[] dy = {0, 1, 0, -1};
    static int[] dx = {1, 0, -1, 0};

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

        N = -1;
        while (true) {
            tc++;
            int N = Integer.parseInt(br.readLine());
            if (N == 0) break;
            map = new int[N][N];

            for (int i = 0; i < N; i++) {
                StringTokenizer st = new StringTokenizer(br.readLine(), " ");
                for (int j = 0; j < N; j++) {
                    map[i][j] = Integer.parseInt(st.nextToken());
                } // 각 테스트 케이스 입력 끝
            }

            Y = 0;
            X = 0;

            int D[][] = new int[N][N];
            boolean[][] visited = new boolean[N][N];

            for (int[] line :
                    D) {
                Arrays.fill(line, Integer.MAX_VALUE);
            }

            D[Y][X] = map[Y][X];

            Queue<int[]> queue = new PriorityQueue<>((o1, o2) -> o1[2] - o2[2]);
            queue.offer(new int[]{Y, X, map[Y][X]});
            visited[Y][X] = true;
            int cnt = 0;

            while (!queue.isEmpty()) {
                int[] cur = queue.poll();
                int curY = cur[0];
                int curX = cur[1];
                int curVal = map[curY][curX];

                for (int j = 0; j < 4; j++) {
                    int ny = curY + dy[j];
                    int nx = curX + dx[j];

                    if (ny >= 0 && nx >= 0 && ny < N && nx < N) {
                        if (D[ny][nx] > D[curY][curX] + map[ny][nx]) {
                            D[ny][nx] = D[curY][curX] + map[ny][nx];
                            queue.offer(new int[]{ny, nx, curVal + map[ny][nx]});
                        }
                    }
                }
            }
            System.out.println("Problem " + tc + ":" + " " + D[N - 1][N - 1]);
        }

    }
}
```

이 문제는 주어진 지도에서 목적지(N-1, N-1)까지 최소의 비용으로 도달 하는 것이 목적이다. 시작 지점은 (0, 0)이다. 테스트 케이스 개수가 따로 주어지지 않으므로 while 문을 사용해 N이 0이 될때 종료하도록 해준다.

- 시작점은 0, 0이다
- 다익스트라 배열을 2차원 배열로 초기화 한다
- 우선 순위 큐에 시작 점과 빼야할 값을 함께 저장한다
- 4 방향 탐색을 하면서 다음 갈 다익스트라 배열에 저장된 거리와 현재 다익스트라 배열 거리 + 지도의 다음 거리를 비교해 다음 좌표가 더 가까우면(작으면) 다익스트라 배열을 갱신해 주고 큐에 다음 좌표를 추가해 준다.
- 반복해 주면 N-1, N-1 까지 가면서 최소 비용을 구할 수 있다.
- 다익스트라 배열의 N-1, N-1이 답이 된다.

# Kruskal 알고리즘

크루스칼 알고리즘은 주어진 모든 정점을 가장 적은 비용으로 연결할 수 있다. 정점 사이에 사이클이 발생하면 안된다는 특징이 있다. 프로그래머스 섬 연결하기 문제로 크루스칼 알고리즘을 알아본다.

[섬 연결하기 문제 보러가기](https://school.programmers.co.kr/learn/courses/30/lessons/42861)

섬 연결하기 문제는 (start, end, 간선) 으로 정보가 주어진다. 간선의 최소를 만들 수 있도록 크루스칼 알고리즘을 사용한다

```java
import java.util.Arrays;

public class Solution {
    static int n, cnt, cost;
    static int[] cycle;

    public static void main(String[] args) {
        n = 4;
        int[][] costs = \{\{0, 1, 1}, {0, 2, 2}, {1, 2, 5}, {1, 3, 1}, {2, 3, 8}};

        cnt = 0;
        // 사이클 테이블 선언
        cycle = new int[n];
        for (int i = 0; i < n; i++) {
            cycle[i] = i;
        }

        // 비용 순으로 정렬
        Arrays.sort(costs, ((o1, o2) -> o1[2] - o2[2]));

        for (int i = 0; i < costs.length; i++) {
            int[] item = costs[i];

            // 사이클 테이블 갱신
            int start = item[0]; // source
            int end = item[1]; // dest
            cost = item[2];

            // 만약에 부모가 다르면 사이클이 안생기고 cost 를 더해주면서 연결해주면뎀
            if (find(start) != find(end)) {
                cnt += cost;
                union(start, end);
            }
        }
        System.out.println(cnt);
//        return cnt;
    }

    private static int find(int node) {
        if (cycle[node] == node) return node;
        return cycle[node] = find(cycle[node]);
    }

    private static void union(int start, int end) {
        int startParent = find(start);
        int endParent = find(end);

        if (startParent > endParent) cycle[startParent] = endParent;
        else cycle[endParent] = startParent;
    }
}
```

크루스칼 알고리즘은 부모 노드 확인을 위한 parent 배열과 노드를 연결 시킬 union-find 알고리즘을 사용한다

- 노드 개수 만큼 parent 배열을 초기화 해 준다. parent 배열의 초기화는 노드 자기 자신의 값으로 초기화 해준다
- 정보에 start와 end가 주어지는데 각 정점의 부모를 찾아 다르면 작은 부모 노드로 갱신하게 된다
  - find 메서드로 현재 노드 값과 부모 값이 같으면 반환하고 그렇지 않으면 다시 부모를 찾는다
- start와 end의 부모가 같지 않으면, union(합집합) 메서드를 사용해 연결해 준다. 연결과 동시에 간선을 더해 준다
  - union 연산은 각 노드의 부모를 찾은 후 더 작은 부모로 갱신한다
- 모든 동작이 끝나게 되면 parent 배열의 값이 하나로 통일 된다(모두 연결이 가능한 경우)
