---
title: "깊이 우선 탐색(DFS)과 너비 우선 탐색(BFS)의 차이와 대표적인 사용 예시"  
categories:  
  - Algorithm  
tags:  
  - Algorithm  
  - DFS  
  - BFS  
  - Graph  
  - Java  
last_modified_at:   
---

### 깊이 우선 탐색(DFS)과 너비 우선 탐색(BFS)의 차이와 대표적인 사용 예시

깊이 우선 탐색(DFS, Depth-First Search)과 너비 우선 탐색(BFS, Breadth-First Search)은 그래프나 트리를 탐색하는 대표적인 알고리즘으로, 탐색 방식과 사용 사례에서 차이가 있음  
DFS는 한 경로를 끝까지 탐색하고, BFS는 가까운 노드부터 차례로 탐색  

---

#### 📌 용어 설명  
- DFS (깊이 우선 탐색): 한 경로를 최대한 깊게 탐색한 후, 이전 단계로 돌아가 다른 경로 탐색  
- BFS (너비 우선 탐색): 시작 노드에서 가까운 노드부터 순차적으로 탐색  
- 그래프: 노드(정점)와 엣지(간선)로 구성된 자료구조  
- visited 배열: 방문한 노드를 추적해 중복 탐색 방지  
- 재귀/스택/큐: DFS는 재귀 또는 스택, BFS는 큐를 사용해 구현  

---

#### 📌 DFS와 BFS의 차이점  

1. 탐색 방식:  
   - DFS: 한 경로를 끝까지 탐색(깊게 파고듦) 후, 막히면 백트래킹으로 다른 경로 탐색  
   - BFS: 시작 노드에서 가까운 노드부터 단계적으로 탐색, 같은 거리의 노드를 모두 확인  

2. 구현 방식:  
   - DFS: 재귀 호출 또는 스택 사용  
     ```java  
     import java.util.ArrayList;

     public class DFSExample {
         static ArrayList<Integer>[] graph;
         static boolean[] visited;

         public static void main(String[] args) {
             int n = 5; // 노드 수
             graph = new ArrayList[n + 1];
             visited = new boolean[n + 1];
             for (int i = 1; i <= n; i++) {
                 graph[i] = new ArrayList<>();
             }
             // 예시 그래프: 1-2, 1-3, 2-4, 3-5
             graph[1].add(2); graph[1].add(3);
             graph[2].add(1); graph[2].add(4);
             graph[3].add(1); graph[3].add(5);
             graph[4].add(2); graph[5].add(3);

             dfs(1);
         }

         static void dfs(int node) {
             visited[node] = true;
             System.out.print(node + " "); // 출력: 1 2 4 3 5
             for (int next : graph[node]) {
                 if (!visited[next]) {
                     dfs(next);
                 }
             }
         }
     }
     ```  
   - BFS: 큐를 사용해 노드를 순차적으로 처리  
     ```java  
     import java.util.ArrayList;
     import java.util.LinkedList;
     import java.util.Queue;

     public class BFSExample {
         static ArrayList<Integer>[] graph;
         static boolean[] visited;

         public static void main(String[] args) {
             int n = 5;
             graph = new ArrayList[n + 1];
             visited = new boolean[n + 1];
             for (int i = 1; i <= n; i++) {
                 graph[i] = new ArrayList<>();
             }
             // 예시 그래프: 1-2, 1-3, 2-4, 3-5
             graph[1].add(2); graph[1].add(3);
             graph[2].add(1); graph[2].add(4);
             graph[3].add(1); graph[3].add(5);
             graph[4].add(2); graph[5].add(3);

             bfs(1);
         }

         static void bfs(int start) {
             Queue<Integer> queue = new LinkedList<>();
             queue.offer(start);
             visited[start] = true;

             while (!queue.isEmpty()) {
                 int node = queue.poll();
                 System.out.print(node + " "); // 출력: 1 2 3 4 5
                 for (int next : graph[node]) {
                     if (!visited[next]) {
                         queue.offer(next);
                         visited[next] = true;
                     }
                 }
             }
         }
     }
     ```

3. 성능 및 메모리:  
   - DFS: 재귀 호출로 스택 메모리 사용, 깊은 그래프에서 스택 오버플로우 가능  
   - BFS: 큐 사용으로 메모리 소모 많음, 넓은 그래프에서 큐 크기 증가  

4. 탐색 순서:  
   - DFS: 깊게 들어가므로 경로가 긴 경우 유리 (예: 1 → 2 → 4)  
   - BFS: 가까운 노드부터 탐색, 최단 경로 찾기에 적합 (예: 1 → 2, 3 → 4, 5)  

---

#### 📌 대표적인 사용 예시  

1. DFS 사용 예시:  
   - 경로 탐색: 미로 찾기, 모든 가능한 경로 탐색  
   - 연결 요소 확인: 그래프에서 연결된 노드 그룹 찾기  
   - 백트래킹 문제: 순열, 조합, N-Queen 문제  
   ```java  
   // N-Queen 예시 (간략화)
   static int n = 4;
   static int[] queen = new int[n];
   static int count = 0;

   static void nQueen(int row) {
       if (row == n) {
           count++;
           return;
       }
       for (int col = 0; col < n; col++) {
           if (isSafe(row, col)) {
               queen[row] = col;
               nQueen(row + 1);
           }
       }
   }
   ```

2. BFS 사용 예시:  
   - 최단 경로: 미로에서 최단 경로 찾기, 네트워크의 최소 홉 수 계산  
   - 레벨별 탐색: 트리에서 레벨 순회, 소셜 네트워크의 친구 거리 계산  
   - 연결된 영역 탐색: 플러드 필(Flood Fill), 섬의 개수 문제  
   ```java  
   // 섬의 개수 (격자 그래프 BFS)
   static int[][] grid;
   static boolean[][] visited;
   static int[] dx = {-1, 1, 0, 0};
   static int[] dy = {0, 0, -1, 1};

   static void bfsIsland(int x, int y, int rows, int cols) {
       Queue<int[]> queue = new LinkedList<>();
       queue.offer(new int[]{x, y});
       visited[x][y] = true;

       while (!queue.isEmpty()) {
           int[] cur = queue.poll();
           for (int i = 0; i < 4; i++) {
               int nx = cur[0] + dx[i];
               int ny = cur[1] + dy[i];
               if (nx >= 0 && nx < rows && ny >= 0 && ny < cols && !visited[nx][ny] && grid[nx][ny] == 1) {
                   queue.offer(new int[]{nx, ny});
                   visited[nx][ny] = true;
               }
           }
       }
   }
   ```

3. IntelliJ 디버깅 팁:  
   - DFS: 재귀 호출 스택과 `visited` 배열 변화 추적  
   - BFS: 큐의 상태와 `visited` 배열 점검  

---

#### 📌 DFS와 BFS 비교  

| 항목 | DFS | BFS |  
| --- | --- | --- |  
| 탐색 방식 | 깊이 우선, 한 경로 끝까지 | 너비 우선, 가까운 노드부터 |  
| 자료구조 | 재귀/스택 | 큐 |  
| 메모리 | 깊은 그래프에서 적음 | 넓은 그래프에서 많음 |  
| 사용 예시 | 경로 탐색, 백트래킹 | 최단 경로, 레벨별 탐색 |  

---

#### 📌 추가 팁  

- 입력 처리: 그래프 입력 시 `Scanner`로 처리, 버퍼 정리 주의  
   ```java  
   import java.util.Scanner;

   Scanner scanner = new Scanner(System.in);
   int n = scanner.nextInt();
   scanner.nextLine(); // 버퍼 정리
   ```  
- Spring Boot 연계: 그래프 탐색 결과를 API로 제공  
   ```java  
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RequestParam;
   import org.springframework.web.bind.annotation.RestController;
   import java.util.ArrayList;
   import java.util.LinkedList;
   import java.util.Queue;

   @RestController
   public class GraphController {
       @GetMapping("/bfs")
       public List<Integer> bfs(@RequestParam int start) {
           ArrayList<Integer>[] graph = new ArrayList[6];
           boolean[] visited = new boolean[6];
           for (int i = 1; i <= 5; i++) {
               graph[i] = new ArrayList<>();
           }
           // 예시 그래프
           graph[1].add(2); graph[1].add(3);
           graph[2].add(1); graph[2].add(4);
           graph[3].add(1); graph[3].add(5);
           graph[4].add(2); graph[5].add(3);

           List<Integer> result = new ArrayList<>();
           Queue<Integer> queue = new LinkedList<>();
           queue.offer(start);
           visited[start] = true;

           while (!queue.isEmpty()) {
               int node = queue.poll();
               result.add(node);
               for (int next : graph[node]) {
                   if (!visited[next]) {
                       queue.offer(next);
                       visited[next] = true;
                   }
               }
           }
           return result;
       }
   }
   ```  
   application.yml:  
   ```yml  
   spring:
     mvc:
       throw-exception-if-no-handler-found: true
   ```  
- 효율성 최적화: BFS에서 큐 크기 예측, DFS에서 재귀 깊이 제한 확인  

---

#### 📌 주의사항  

* 무한 루프: 순환 그래프에서 `visited` 배열 누락 시 무한 탐색 발생  
* 메모리 관리: BFS는 큰 그래프에서 큐 메모리 사용량 주의  
* 입력 검증: 그래프 입력 시 노드/엣지 범위 확인