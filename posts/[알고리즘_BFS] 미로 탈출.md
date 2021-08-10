---
description: '이것이 코딩 테스트다'의 BFS 알고리즘 문제를 풀어 봅니다.
tag: Algorithm
---

## 문제

동빈이는 ![N times M](https://math.now.sh?from=N%5Ctimes%20M&color=black) 크기의 직사각형 형태의 미로에 갇혀 있다. 미로에는 여러 마리의 괴물이 있어 이를 피해 탈출해야 한다. 동빈이의 위치는 ![1 comma 1](https://math.now.sh?from=%281%2C%201%29&color=black)이고 미로의 출구는 ![N comma M](https://math.now.sh?from=%28N%2C%20M%29&color=black)의 위치에 존재하며 한 번에 한 칸씩 이동할 수 있다. 이때 괴물이 있는 부분은 0으로, 괴물이 없는 부분은 1로 표시되어 있다. 미로는 반드시 탈출할 수 있는 형태로 제시된다. 이때 동빈이가 탈출하기 위해 움직여야 하는 최소 칸의 개수를 구하시오. 칸을 셀 때는 시작 칸과 마지막 칸을 모두 포함해서 계산한다.

## 제한

| 시간 제한 | 메모리 제한 |
| :-------: | :---------: |
|    1초    |    128MB    |

## 입력 조건

- 첫째 줄에 두 정수 ![N](https://math.now.sh?from=N&color=black), ![Equation](https://math.now.sh?from=M&color=black)(![Equation](https://math.now.sh?from=4%5Cleq%20N%2C%20M%5Cleq%20200&color=black))이 주어집니다. 다음 ![N](https://math.now.sh?from=N&color=black)개의 줄에는 각각 ![Equation](https://math.now.sh?from=M&color=black)개의 정수(0 혹은 1)로 미로의 정보가 주어진다. 각각의 수들은 공백 없이 붙어서 입력으로 제시된다. 또한 시작 칸과 마지막 칸은 항상 1이다.

## 출력 조건

- 첫째 줄에 최소 이동 칸의 개수를 출력한다.

## 입력 예시

```
5 6
101010
111111
000001
111111
111111

```

## 출력 예시

```
10

```

## 풀이

이 문제는 시작점부터 도착점까지의 최소 이동 횟수를 구하는 문제이다. 이는 그래프에서 인접한 노드를 차례로 방문하는 너비 우선 탐색(이하 BFS)을 활용하면 해결할 수 있다. 하지만 BFS는 처음부터 끝까지 그래프의 모든 노드를 한 번씩 탐색하기 때문에 몇 가지 문제가 있다.

- 탐색을 하다가 도착점이 발견되면 멈춰야 하는지?
- 어떻게 시작점부터 도착점까지의 거리를 잴 수 있는지?

탐색을 하다가 도착점이 발견되면 그 시점에서 탐색을 중단해야 한다. 또한 시작점부터 도착점까지의 거리를 재는 방법은 현재 탐색중인 노드에서 방문할 수 있는 노드까지의 거리는 1이므로 현재 탐색중인 노드에 누적된 거리 + 1을 하면 된다. 또한 칸을 샐 때는 시작 칸과 마지막 칸을 포함하므로 시작점의 거리는 1로 시작한다.

```python
class Offset:
    def __init__(self, x, y) -> None:
        self.x = x
        self.y = y


offset = [Offset(1, 0), Offset(0, 1), Offset(-1, 0), Offset(0, -1)]

def bfs(graph, visited, current_node, width, height):
    queue = deque()
    queue.append(current_node)
    visited[current_node[0]][current_node[1]] = 1 # 첫 번째 노드 방문 처리
    while queue: # queue가 비지 않으면 안쪽 실행
        current_node = queue.popleft() # dequeue
        if current_node[0] == height - 1 and current_node[1] == width - 1: return # 현재 위치가 도착점이면 탐색 중단
        for idx in range(4): # 시계방향으로 탐색
            move_to_y = current_node[0] + offset[idx].y # 다음에 탐색할 위치
            move_to_x = current_node[1] + offset[idx].x # 다음에 탐색할 위치
            if 0 <= move_to_y < height and 0 <= move_to_x < width: # 다음에 탐색할 위치가 유효하지 않으면 다음 위치 탐색
                if graph[move_to_y][move_to_x] == '0': continue # 다음에 탐색할 위치가 0이면 이동 불가
                if visited[move_to_y][move_to_x] == 0: # 다음에 탐색할 위치가 방문하지 않은 위치이면 enqueue하고 방문 처리
                    queue.append((move_to_y, move_to_x, ))
                    visited[move_to_y][move_to_x] = visited[current_node[0]][current_node[1]] + 1
```

이 문제의 답은 `visited[height - 1][width - 1]`이다.