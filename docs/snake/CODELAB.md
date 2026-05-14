## 개요

- 대상: `turtle` 이후 `pygame`으로 처음 넘어오는 초급 학습자
- 수업 구성: class당 40~50분, "작동하는 코드 + 기능 1개 확장" 방식
- 최종 산출물: class 4 종료 시 실행 가능한 Snake 게임 1개
- 실행 환경: Python 3.10+ / `pip install pygame`

## 목차

1. class 1. 화면 만들기 + 격자 이동
2. class 2. 몸통 리스트 + 먹이 + 길이 증가
3. class 3. 충돌 판정 + 게임오버 + 재시작
4. class 4. 점수/HUD + 시작/일시정지 + 마무리 폴리싱

---

## class 1. 화면 만들기 + 격자 이동

### 목표

- `pygame` 창을 열고 게임 루프 구조를 익힌다.
- 화면 좌표를 픽셀이 아닌 격자(칸) 단위로 다룬다.
- 타이머 이벤트로 "한 칸씩" 이동을 구현한다.

### 핵심 변수/함수

- `CELL_SIZE`, `GRID_W`, `GRID_H`: 격자 크기와 보드 크기 정의
- `snake_x`, `snake_y`: 뱀 머리의 격자 좌표
- `dx`, `dy`: 이동 방향 벡터
- `draw_grid()`: 격자선을 렌더링
- `draw_cell()`: 격자 좌표를 실제 픽셀 사각형으로 그려줌

### 단계별 구현

#### 단계 1) 기본 창/루프 만들기

##### 세부목표
  - 초기화와 화면 생성 코드를 연결해 실행 가능한 기본 게임 화면을 만든다.
  - 현재 단계의 상태값이 다음 단계 동작으로 어떻게 이어지는지 이해한다.

```python
import pygame
import sys

pygame.init()
screen = pygame.display.set_mode((600, 400))
pygame.display.set_caption("Snake Codelab - Session 1")
clock = pygame.time.Clock()
```

##### 선언한 변수/함수의 목적
  - `screen`: 화면 출력 대상 객체
  - `clock`: 프레임 속도 제어 객체

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `screen = pygame.display.set_mode((600, 400))`: 게임 창 크기를 확정해 좌표 기준을 만든다.
  - `pygame.init()`: 키보드 입력, 화면 출력, 사운드 같은 pygame 하위 모듈을 한 번에 초기화해 하드웨어와 통신할 준비를 마친다.
  - `pygame.display.set_caption("Snake Codelab - Session 1")`: 창 제목을 설정해 실행 중인 게임 세션을 OS 창 표시줄에서 바로 식별할 수 있게 한다.
  - `clock = pygame.time.Clock()`: `clock` 객체를 만들어 프레임 간 시간 간격을 제어해, PC 성능이 달라도 게임 속도가 크게 흔들리지 않게 한다.

#### 단계 2) 격자 상수로 화면 크기 재정의

##### 세부목표
  - 초기화와 화면 생성 코드를 연결해 실행 가능한 기본 게임 화면을 만든다.
  - 현재 단계의 상태값이 다음 단계 동작으로 어떻게 이어지는지 이해한다.

```python
CELL_SIZE = 20
GRID_W = 30
GRID_H = 20
WIDTH = CELL_SIZE * GRID_W
HEIGHT = CELL_SIZE * GRID_H

screen = pygame.display.set_mode((WIDTH, HEIGHT))
```

##### 선언한 변수/함수의 목적
  - `CELL_SIZE`: 격자 한 칸의 픽셀 크기
  - `GRID_W`: 격자 가로 칸 수
  - `GRID_H`: 격자 세로 칸 수
  - `WIDTH`: 게임 화면 가로 크기
  - `HEIGHT`: 게임 화면 세로 크기
  - `screen`: 화면 출력 대상 객체

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `screen = pygame.display.set_mode((WIDTH, HEIGHT))`: 게임 창 크기를 확정해 좌표 기준을 만든다.
  - `CELL_SIZE = 20`: `CELL_SIZE`를 기준 단위로 정해 격자 좌표 계산과 실제 픽셀 렌더링 비율을 일치시킨다.
  - `GRID_W = 30`: 가로 칸 수를 고정해 x축 이동 한계와 벽 충돌 기준을 명확히 만든다.
  - `GRID_H = 20`: 세로 칸 수를 고정해 y축 이동 한계와 벽 충돌 기준을 명확히 만든다.

#### 단계 3) 격자 그리기 함수 추가

##### 세부목표
  - 그리기 코드를 함수로 분리해 같은 렌더링 규칙을 재사용 가능하게 만든다.
  - 학습 목표는 `def`/매개변수/반환값 기준으로 코드를 읽고 기능 단위로 나누는 것이다.

```python
def draw_grid(surface):
    color = (55, 55, 55)
    for x in range(0, WIDTH, CELL_SIZE):
        pygame.draw.line(surface, color, (x, 0), (x, HEIGHT))
    for y in range(0, HEIGHT, CELL_SIZE):
        pygame.draw.line(surface, color, (0, y), (WIDTH, y))
```

##### 선언한 변수/함수의 목적
  - `draw_grid()`: 격자선을 화면에 그리는 함수
  - `surface`: 그리기 대상 Surface
  - `color`: 색상 값

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `def draw_grid(surface):`: `draw_grid()` 함수 정의를 시작해 관련 로직을 기능 단위로 분리한다.
  - `for x in range(0, WIDTH, CELL_SIZE):`: 가로축을 `CELL_SIZE` 간격으로 순회해 격자 세로선을 빠짐없이 그린다.
  - `color = (55, 55, 55)`: 현재 그릴 대상의 색상을 명시해 같은 도형 호출에서도 역할 차이가 화면에서 바로 보이게 한다.
  - `pygame.draw.line(surface, color, (x, 0), (x, HEIGHT))`: 지정한 x좌표에 세로선을 그려, 보드의 열 경계를 눈에 보이게 만든다.

#### 단계 4) 머리를 격자 좌표로 표시

##### 세부목표
  - 그리기 코드를 함수로 분리해 같은 렌더링 규칙을 재사용 가능하게 만든다.
  - 학습 목표는 `def`/매개변수/반환값 기준으로 코드를 읽고 기능 단위로 나누는 것이다.

```python
snake_x, snake_y = 10, 10

def draw_cell(surface, color, gx, gy):
    rect = pygame.Rect(gx * CELL_SIZE, gy * CELL_SIZE, CELL_SIZE, CELL_SIZE)
    pygame.draw.rect(surface, color, rect)
```

##### 선언한 변수/함수의 목적
  - `draw_cell()`: 격자 좌표의 한 칸을 그리는 함수
  - `surface`: 그리기 대상 Surface
  - `color`: 색상 값
  - `gx`: 격자 칸 좌표 입력값이다. CELL_SIZE와 곱해 픽셀 위치로 바꿔 그린다.
  - `gy`: 격자 칸 좌표 입력값이다. CELL_SIZE와 곱해 픽셀 위치로 바꿔 그린다.
  - `snake_x`: 뱀 위치/몸통 상태 데이터
  - `snake_y`: 뱀 위치/몸통 상태 데이터
  - `rect`: 충돌/렌더링용 사각형 객체

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `def draw_cell(surface, color, gx, gy):`: `draw_cell()` 함수 정의를 시작해 관련 로직을 기능 단위로 분리한다.
  - `snake_x, snake_y = 10, 10`: 뱀 시작 위치를 가운데 근처로 잡아, 시작 직후 이동과 경계 충돌을 안정적으로 확인하게 한다.
  - `rect = pygame.Rect(gx * CELL_SIZE, gy * CELL_SIZE, CELL_SIZE, CELL_SIZE)`: 사각형 좌표/크기를 `Rect`로 묶어 렌더링과 충돌 판정에서 같은 기준 데이터를 재사용하게 한다.
  - `pygame.draw.rect(surface, color, rect)`: 격자 좌표를 실제 사각형 픽셀로 렌더링해 뱀/먹이 같은 게임 객체를 화면에 표시한다.

#### 단계 5) 방향키 입력 + 타이머 이동

##### 세부목표
  - 입력 이벤트를 상태값 변화와 연결해 사용자의 조작이 게임 동작으로 이어지게 만든다.
  - 학습 목표는 `if/elif` 분기로 입력 조건을 나누고 상태 업데이트 순서를 이해하는 것이다.

```python
MOVE_EVENT = pygame.USEREVENT + 1
pygame.time.set_timer(MOVE_EVENT, 180)

dx, dy = 1, 0
```

```python
for event in pygame.event.get():
    if event.type == pygame.QUIT:
        pygame.quit()
        sys.exit()
    elif event.type == pygame.KEYDOWN:
        if event.key == pygame.K_UP:
            dx, dy = 0, -1
        elif event.key == pygame.K_DOWN:
            dx, dy = 0, 1
        elif event.key == pygame.K_LEFT:
            dx, dy = -1, 0
        elif event.key == pygame.K_RIGHT:
            dx, dy = 1, 0
    elif event.type == MOVE_EVENT:
        snake_x += dx
        snake_y += dy
        snake_x %= GRID_W
        snake_y %= GRID_H
```

##### 선언한 변수/함수의 목적
  - `MOVE_EVENT`: 뱀 이동 이벤트 ID
  - `dx`: x축 이동량으로 프레임마다 좌우 이동 거리를 결정한다.
  - `dy`: y축 이동량으로 프레임마다 상하 이동 거리를 결정한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `pygame.time.set_timer(MOVE_EVENT, 180)`: 일정 주기 이벤트를 등록해 업데이트 박자를 고정한다.
  - `if event.type == pygame.QUIT:`: 이벤트 타입을 비교해 입력/업데이트 처리를 분기한다.
  - `snake_x %= GRID_W`: 모듈로 대입으로 좌표를 보드 범위 안에 유지한다.
  - `elif event.type == pygame.KEYDOWN:`: 이벤트 타입을 비교해 입력/업데이트 처리를 분기한다.

### class 1 최종 코드

```python
import pygame
import sys

pygame.init()

CELL_SIZE = 20
GRID_W = 30
GRID_H = 20
WIDTH = CELL_SIZE * GRID_W
HEIGHT = CELL_SIZE * GRID_H

BG = (25, 25, 25)
GRID = (55, 55, 55)
HEAD = (120, 240, 120)

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Snake Codelab - Session 1")
clock = pygame.time.Clock()

snake_x, snake_y = 10, 10
dx, dy = 1, 0

MOVE_EVENT = pygame.USEREVENT + 1
pygame.time.set_timer(MOVE_EVENT, 180)

def draw_grid(surface):
    for x in range(0, WIDTH, CELL_SIZE):
        pygame.draw.line(surface, GRID, (x, 0), (x, HEIGHT))
    for y in range(0, HEIGHT, CELL_SIZE):
        pygame.draw.line(surface, GRID, (0, y), (WIDTH, y))

def draw_cell(surface, color, gx, gy):
    rect = pygame.Rect(gx * CELL_SIZE, gy * CELL_SIZE, CELL_SIZE, CELL_SIZE)
    pygame.draw.rect(surface, color, rect)

while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        elif event.type == pygame.KEYDOWN:
            if event.key == pygame.K_UP:
                dx, dy = 0, -1
            elif event.key == pygame.K_DOWN:
                dx, dy = 0, 1
            elif event.key == pygame.K_LEFT:
                dx, dy = -1, 0
            elif event.key == pygame.K_RIGHT:
                dx, dy = 1, 0
        elif event.type == MOVE_EVENT:
            snake_x += dx
            snake_y += dy
            snake_x %= GRID_W
            snake_y %= GRID_H

    screen.fill(BG)
    draw_grid(screen)
    draw_cell(screen, HEAD, snake_x, snake_y)
    pygame.display.flip()
    clock.tick(60)
```

---

## class 2. 몸통 리스트 + 먹이 + 길이 증가

### 목표

- 단일 좌표에서 "몸통 리스트" 구조로 확장한다.
- 뱀과 겹치지 않는 먹이 좌표를 생성한다.
- 먹이 섭취 시 길이를 1칸 늘린다.

### 핵심 변수/함수

- `snake`: `[(x, y), ...]` 형태의 몸통 좌표 리스트
- `direction`: `(dx, dy)` 튜플
- `place_food(snake_body)`: 뱀과 겹치지 않는 먹이 좌표 반환

### 단계별 구현

#### 단계 1) 뱀 자료구조를 리스트로 변경

##### 세부목표
  - 뱀 자료구조를 리스트로 변경 단계의 코드를 게임 루프에 연결해 실제 동작으로 확인한다.
  - 현재 단계의 상태값이 다음 단계 동작으로 어떻게 이어지는지 이해한다.

```python
snake = [(10, 10), (9, 10), (8, 10)]
direction = (1, 0)
```

##### 선언한 변수/함수의 목적
  - `snake`: 뱀 위치/몸통 상태 데이터
  - `direction`: (dx, dy) 방향 벡터를 저장해 다음 머리 좌표 계산 기준으로 사용한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `snake = [(10, 10), (9, 10), (8, 10)]`: 머리+몸통이 포함된 초기 길이를 만들어 이동, 성장, 자기충돌 규칙을 바로 테스트할 수 있게 한다.
  - `direction = (1, 0)`: 초기 진행 방향을 오른쪽으로 고정해 첫 프레임 이동 결과가 예측 가능하게 만든다.

#### 단계 2) 이동 로직을 "새 머리 삽입"으로 변경

##### 세부목표
  - 리스트 조작으로 객체 상태를 갱신해 이동/성장/정리 규칙을 구현한다.
  - 현재 단계의 상태값이 다음 단계 동작으로 어떻게 이어지는지 이해한다.

```python
head_x, head_y = snake[0]
dx, dy = direction
new_head = (head_x + dx, head_y + dy)

new_head = (new_head[0] % GRID_W, new_head[1] % GRID_H)
snake.insert(0, new_head)
snake.pop()
```

##### 선언한 변수/함수의 목적
  - `head_x`: 객체 위치 좌표를 저장한다. 매 프레임 Rect 생성과 그리기에 사용되어 화면 위치가 정해진다.
  - `head_y`: 객체 위치 좌표를 저장한다. 매 프레임 Rect 생성과 그리기에 사용되어 화면 위치가 정해진다.
  - `dx`: x축 이동량으로 프레임마다 좌우 이동 거리를 결정한다.
  - `dy`: y축 이동량으로 프레임마다 상하 이동 거리를 결정한다.
  - `new_head`: 다음 프레임 머리 좌표

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `snake.insert(0, new_head)`: 리스트 앞에 새 머리를 넣어 전진 이동을 표현한다.
  - `snake.pop()`: 리스트 마지막 요소를 제거해 길이 유지/정리 처리를 한다.
  - `head_x, head_y = snake[0]`: 머리 좌표를 분리해 다음 칸 계산을 몸통 리스트 처리와 독립적으로 단순화한다.
  - `dx, dy = direction`: 방향 벡터를 축별 값으로 나눠 다음 머리 좌표 계산식을 읽기 쉽게 만든다.

#### 단계 3) 먹이 배치 함수 작성

##### 세부목표
  - 반복되는 규칙을 함수로 분리해 가독성과 유지보수성을 높인다.
  - 학습 목표는 `def`/매개변수/반환값 기준으로 코드를 읽고 기능 단위로 나누는 것이다.

```python
import random

def place_food(snake):
    while True:
        pos = (random.randint(0, GRID_W - 1), random.randint(0, GRID_H - 1))
        if pos not in snake:
            return pos

food = place_food(snake)
```

##### 선언한 변수/함수의 목적
  - `place_food()`: 뱀과 겹치지 않는 먹이 좌표를 생성하는 함수
  - `snake`: 뱀 위치/몸통 상태 데이터
  - `pos`: 입력 좌표를 잠깐 저장해 클릭/겹침 판정에 사용한다.
  - `food`: 먹이 좌표 값

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `def place_food(snake):`: `place_food()` 함수 정의를 시작해 관련 로직을 기능 단위로 분리한다.
  - `pos = (random.randint(0, GRID_W - 1), random.randint(0, GRID_H - 1))`: 무작위 값을 생성해 좌표/패턴 다양성을 만든다.
  - `if pos not in snake:`: 뱀 몸통과 겹치지 않는 좌표일 때만 먹이 위치로 채택한다.
  - `while True:`: 게임이 종료될 때까지 입력 처리→상태 업데이트→렌더링을 프레임 단위로 반복시키는 메인 루프다.

#### 단계 4) 먹이 섭취 분기 처리

##### 세부목표
  - 리스트 조작으로 객체 상태를 갱신해 이동/성장/정리 규칙을 구현한다.
  - 현재 단계의 상태값이 다음 단계 동작으로 어떻게 이어지는지 이해한다.

```python
if new_head == food:
    food = place_food(snake)  # 꼬리를 제거하지 않으므로 길이 +1
else:
    snake.pop()
```

##### 선언한 변수/함수의 목적
  - `food`: 먹이 좌표 값

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `snake.pop()`: 리스트 마지막 요소를 제거해 길이 유지/정리 처리를 한다.
  - `if new_head == food:`: 새 머리 좌표가 먹이와 겹칠 때만 성장과 점수 증가를 적용한다.
  - `food = place_food(snake)  # 꼬리를 제거하지 않으므로 길이 +1`: 뱀과 겹치지 않는 새 먹이 좌표로 갱신해, 성장 직후에도 다음 목표가 즉시 유지되게 한다.
  - `else:`: 앞 조건이 거짓일 때 대체 로직으로 분기해 게임 규칙의 예외 경로를 처리한다.

#### 단계 5) 머리/몸통/먹이 색상 분리 렌더링

##### 세부목표
  - 머리/몸통/먹이 색상 분리 렌더링 단계의 코드를 게임 루프에 연결해 실제 동작으로 확인한다.
  - 현재 단계의 상태값이 다음 단계 동작으로 어떻게 이어지는지 이해한다.

```python
for i, part in enumerate(snake):
    color = (140, 255, 140) if i == 0 else (90, 220, 110)
    draw_cell(screen, color, part[0], part[1])

draw_cell(screen, (255, 90, 90), food[0], food[1])
```

##### 선언한 변수/함수의 목적
  - `color`: 색상 값

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `for i, part in enumerate(snake):`: 머리부터 꼬리까지 순회하며 뱀 몸통을 좌표대로 차례대로 그린다.
  - `color = (140, 255, 140) if i == 0 else (90, 220, 110)`: 머리와 몸통 색을 다르게 정해 진행 방향을 한눈에 구분할 수 있게 한다.
  - `draw_cell(screen, color, part[0], part[1])`: 뱀의 각 몸통 좌표를 한 칸씩 그려 현재 길이와 위치를 프레임마다 갱신해 보여준다.
  - `draw_cell(screen, (255, 90, 90), food[0], food[1])`: 먹이 좌표를 별도 색으로 표시해 플레이어가 다음 목표 위치를 바로 인식하게 한다.

### class 2 최종 코드

```python
import pygame
import random
import sys

pygame.init()

CELL_SIZE = 20
GRID_W = 30
GRID_H = 20
WIDTH = CELL_SIZE * GRID_W
HEIGHT = CELL_SIZE * GRID_H

BG = (22, 22, 22)
GRID = (50, 50, 50)
HEAD = (140, 255, 140)
BODY = (90, 220, 110)
FOOD = (255, 90, 90)

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Snake Codelab - Session 2")
clock = pygame.time.Clock()

snake = [(10, 10), (9, 10), (8, 10)]
direction = (1, 0)

MOVE_EVENT = pygame.USEREVENT + 1
pygame.time.set_timer(MOVE_EVENT, 170)

def draw_grid(surface):
    for x in range(0, WIDTH, CELL_SIZE):
        pygame.draw.line(surface, GRID, (x, 0), (x, HEIGHT))
    for y in range(0, HEIGHT, CELL_SIZE):
        pygame.draw.line(surface, GRID, (0, y), (WIDTH, y))

def draw_cell(surface, color, gx, gy):
    rect = pygame.Rect(gx * CELL_SIZE, gy * CELL_SIZE, CELL_SIZE, CELL_SIZE)
    pygame.draw.rect(surface, color, rect)

def place_food(snake_body):
    while True:
        pos = (random.randint(0, GRID_W - 1), random.randint(0, GRID_H - 1))
        if pos not in snake_body:
            return pos

food = place_food(snake)

while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        elif event.type == pygame.KEYDOWN:
            if event.key == pygame.K_UP and direction != (0, 1):
                direction = (0, -1)
            elif event.key == pygame.K_DOWN and direction != (0, -1):
                direction = (0, 1)
            elif event.key == pygame.K_LEFT and direction != (1, 0):
                direction = (-1, 0)
            elif event.key == pygame.K_RIGHT and direction != (-1, 0):
                direction = (1, 0)
        elif event.type == MOVE_EVENT:
            head_x, head_y = snake[0]
            dx, dy = direction
            new_head = ((head_x + dx) % GRID_W, (head_y + dy) % GRID_H)

            snake.insert(0, new_head)
            if new_head == food:
                food = place_food(snake)
            else:
                snake.pop()

    screen.fill(BG)
    draw_grid(screen)
    draw_cell(screen, FOOD, food[0], food[1])
    for i, part in enumerate(snake):
        draw_cell(screen, HEAD if i == 0 else BODY, part[0], part[1])

    pygame.display.flip()
    clock.tick(60)
```

---

## class 3. 충돌 판정 + 게임오버 + 재시작

### 목표

- 벽 충돌, 자기 몸 충돌을 감지한다.
- `game_over` 상태를 분리해 업데이트를 제어한다.
- `R`로 재시작, `Q`로 종료를 처리한다.

### 핵심 변수/함수

- `game_over`: 라운드 종료 상태
- `reset_game()`: 초기 상태를 한 번에 복구
- `hit_wall`, `hit_self`: 충돌 여부를 명시적으로 계산

### 단계별 구현

#### 단계 1) 경계 래핑 제거 + 벽 충돌 판정

##### 세부목표
  - 충돌/경계 판정을 구현해 실패·성공 상태 전환 조건을 정확히 만든다.
  - 현재 단계의 상태값이 다음 단계 동작으로 어떻게 이어지는지 이해한다.

```python
new_head = (head_x + dx, head_y + dy)

hit_wall = (
    new_head[0] < 0 or new_head[0] >= GRID_W or
    new_head[1] < 0 or new_head[1] >= GRID_H
)
```

##### 선언한 변수/함수의 목적
  - `new_head`: 다음 프레임 머리 좌표
  - `hit_wall`: 벽 충돌 여부를 담는 중간값이다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `new_head = (head_x + dx, head_y + dy)`: 다음 프레임에 머리가 들어갈 좌표를 미리 계산해 충돌 판정과 이동 적용 순서를 분리한다.
  - `hit_wall = (`: 벽 충돌 여부를 불리언으로 고정해 게임오버 분기를 한 줄 조건으로 단순화한다.
  - `new_head[0] < 0 or new_head[0] >= GRID_W or`: 새 머리의 x좌표가 보드 범위를 벗어났는지 검사해 벽 충돌 조건을 만든다.
  - `new_head[1] < 0 or new_head[1] >= GRID_H`: 새 머리의 y좌표가 보드 범위를 벗어났는지 검사해 상하 벽 충돌까지 포함한다.

#### 단계 2) 자기 몸 충돌 판정

##### 세부목표
  - 충돌/경계 판정을 구현해 실패·성공 상태 전환 조건을 정확히 만든다.
  - 현재 단계의 상태값이 다음 단계 동작으로 어떻게 이어지는지 이해한다.

```python
hit_self = new_head in snake
if hit_wall or hit_self:
    game_over = True
```

##### 선언한 변수/함수의 목적
  - `hit_self`: 자기 몸 충돌 여부를 담는 중간값이다.
  - `game_over`: 게임 종료 상태를 저장해 업데이트 중단과 재시작 입력 허용을 제어한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if hit_wall or hit_self:`: 벽 또는 자기몸 충돌 중 하나라도 참이면 즉시 게임오버로 전환한다.
  - `hit_self = new_head in snake`: 자기 몸 충돌 여부를 따로 계산해 벽 충돌과 합쳐 명확한 종료 조건을 만든다.
  - `game_over = True`: 게임 상태를 즉시 종료 상태로 전환해 입력/업데이트 루프가 플레이 모드에서 빠져나오게 한다.

#### 단계 3) 상태값 도입

##### 세부목표
  - 상태값 도입 단계의 코드를 게임 루프에 연결해 실제 동작으로 확인한다.
  - 현재 단계의 상태값이 다음 단계 동작으로 어떻게 이어지는지 이해한다.

```python
game_over = False

if not game_over:
    # 이동/먹이/충돌 업데이트
    pass
```

##### 선언한 변수/함수의 목적
  - `game_over`: 게임 종료 상태를 저장해 업데이트 중단과 재시작 입력 허용을 제어한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if not game_over:`: 게임오버가 아닐 때만 일반 업데이트를 수행해 종료 후 상태 변경을 막는다.
  - `game_over = False`: 게임 상태를 즉시 종료 상태로 전환해 입력/업데이트 루프가 플레이 모드에서 빠져나오게 한다.
  - `pass`: 아직 동작을 채우지 않은 자리로 두되, 문법을 유지해 루프 흐름을 계속 진행하게 한다.

#### 단계 4) 재시작 초기화 함수 작성

##### 세부목표
  - 입력 이벤트를 상태값 변화와 연결해 사용자의 조작이 게임 동작으로 이어지게 만든다.
  - 학습 목표는 `def`/매개변수/반환값 기준으로 코드를 읽고 기능 단위로 나누는 것이다.

```python
def reset_game():
    snake = [(10, 10), (9, 10), (8, 10)]
    direction = (1, 0)
    food = place_food(snake)
    return snake, direction, food, False
```

```python
if game_over and event.type == pygame.KEYDOWN:
    if event.key == pygame.K_r:
        snake, direction, food, game_over = reset_game()
    elif event.key == pygame.K_q:
        pygame.quit()
        sys.exit()
```

##### 선언한 변수/함수의 목적
  - `reset_game()`: 게임 상태를 초기값으로 되돌리는 함수
  - `snake`: 뱀 위치/몸통 상태 데이터
  - `direction`: (dx, dy) 방향 벡터를 저장해 다음 머리 좌표 계산 기준으로 사용한다.
  - `food`: 먹이 좌표 값
  - `game_over`: 게임 종료 상태를 저장해 업데이트 중단과 재시작 입력 허용을 제어한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `def reset_game():`: `reset_game()` 함수 정의를 시작해 관련 로직을 기능 단위로 분리한다.
  - `if game_over and event.type == pygame.KEYDOWN:`: 이벤트 타입을 비교해 입력/업데이트 처리를 분기한다.
  - `if event.key == pygame.K_r:`: R 키 입력에서만 재시작 로직을 실행해 다른 키 입력과 기능이 섞이지 않게 한다.
  - `snake = [(10, 10), (9, 10), (8, 10)]`: 머리+몸통이 포함된 초기 길이를 만들어 이동, 성장, 자기충돌 규칙을 바로 테스트할 수 있게 한다.

#### 단계 5) 게임오버 메시지 렌더링

##### 세부목표
  - 게임오버 메시지 렌더링 단계의 코드를 게임 루프에 연결해 실제 동작으로 확인한다.
  - 학습 목표는 상태 갱신 이후 렌더링 순서를 지켜 화면과 내부 데이터가 일치하도록 만드는 것이다.

```python
font = pygame.font.SysFont(None, 36)
small = pygame.font.SysFont(None, 24)
```

```python
if game_over:
    t1 = font.render("Game Over", True, (240, 240, 240))
    t2 = small.render("R: Restart / Q: Quit", True, (220, 220, 220))
    screen.blit(t1, (WIDTH // 2 - t1.get_width() // 2, HEIGHT // 2 - 30))
    screen.blit(t2, (WIDTH // 2 - t2.get_width() // 2, HEIGHT // 2 + 8))
```

##### 선언한 변수/함수의 목적
  - `font`: 텍스트 렌더링용 폰트 객체
  - `small`: 텍스트 렌더링용 폰트 객체
  - `t1`: 첫 번째 안내 텍스트 Surface다.
  - `t2`: 두 번째 안내 텍스트 Surface다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `t1 = font.render("Game Over", True, (240, 240, 240))`: 문자열을 화면에 표시할 텍스트 Surface로 변환한다.
  - `screen.blit(t1, (WIDTH // 2 - t1.get_width() // 2, HEIGHT // 2 - 30))`: Surface를 지정 위치에 배치해 UI/텍스트를 출력한다.
  - `if game_over:`: 종료 상태일 때만 게임오버 메시지를 표시해 플레이 화면과 안내 화면을 분리한다.
  - `font = pygame.font.SysFont(None, 36)`: HUD/안내문 렌더링에 쓸 폰트를 먼저 만들어 프레임마다 텍스트를 안정적으로 그릴 수 있게 한다.

### class 3 최종 코드

```python
import pygame
import random
import sys

pygame.init()

CELL_SIZE = 20
GRID_W = 30
GRID_H = 20
WIDTH = CELL_SIZE * GRID_W
HEIGHT = CELL_SIZE * GRID_H

BG = (22, 22, 22)
GRID = (50, 50, 50)
HEAD = (140, 255, 140)
BODY = (90, 220, 110)
FOOD = (255, 90, 90)
TEXT = (240, 240, 240)

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Snake Codelab - Session 3")
clock = pygame.time.Clock()
font = pygame.font.SysFont(None, 36)
small = pygame.font.SysFont(None, 24)

MOVE_EVENT = pygame.USEREVENT + 1
pygame.time.set_timer(MOVE_EVENT, 160)

def draw_grid(surface):
    for x in range(0, WIDTH, CELL_SIZE):
        pygame.draw.line(surface, GRID, (x, 0), (x, HEIGHT))
    for y in range(0, HEIGHT, CELL_SIZE):
        pygame.draw.line(surface, GRID, (0, y), (WIDTH, y))

def draw_cell(surface, color, gx, gy):
    rect = pygame.Rect(gx * CELL_SIZE, gy * CELL_SIZE, CELL_SIZE, CELL_SIZE)
    pygame.draw.rect(surface, color, rect)

def place_food(snake_body):
    while True:
        pos = (random.randint(0, GRID_W - 1), random.randint(0, GRID_H - 1))
        if pos not in snake_body:
            return pos

def reset_game():
    snake_body = [(10, 10), (9, 10), (8, 10)]
    direction_vec = (1, 0)
    food_pos = place_food(snake_body)
    is_over = False
    return snake_body, direction_vec, food_pos, is_over

snake, direction, food, game_over = reset_game()

while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

        if event.type == pygame.KEYDOWN:
            if not game_over:
                if event.key == pygame.K_UP and direction != (0, 1):
                    direction = (0, -1)
                elif event.key == pygame.K_DOWN and direction != (0, -1):
                    direction = (0, 1)
                elif event.key == pygame.K_LEFT and direction != (1, 0):
                    direction = (-1, 0)
                elif event.key == pygame.K_RIGHT and direction != (-1, 0):
                    direction = (1, 0)
            else:
                if event.key == pygame.K_r:
                    snake, direction, food, game_over = reset_game()
                elif event.key == pygame.K_q:
                    pygame.quit()
                    sys.exit()

        elif event.type == MOVE_EVENT and not game_over:
            head_x, head_y = snake[0]
            dx, dy = direction
            new_head = (head_x + dx, head_y + dy)

            hit_wall = (
                new_head[0] < 0 or new_head[0] >= GRID_W or
                new_head[1] < 0 or new_head[1] >= GRID_H
            )
            hit_self = new_head in snake

            if hit_wall or hit_self:
                game_over = True
            else:
                snake.insert(0, new_head)
                if new_head == food:
                    food = place_food(snake)
                else:
                    snake.pop()

    screen.fill(BG)
    draw_grid(screen)
    draw_cell(screen, FOOD, food[0], food[1])
    for i, part in enumerate(snake):
        draw_cell(screen, HEAD if i == 0 else BODY, part[0], part[1])

    if game_over:
        t1 = font.render("Game Over", True, TEXT)
        t2 = small.render("R: Restart / Q: Quit", True, TEXT)
        screen.blit(t1, (WIDTH // 2 - t1.get_width() // 2, HEIGHT // 2 - 30))
        screen.blit(t2, (WIDTH // 2 - t2.get_width() // 2, HEIGHT // 2 + 8))

    pygame.display.flip()
    clock.tick(60)
```

---

## class 4. 점수/HUD + 시작/일시정지 + 마무리 폴리싱

### 목표

- 점수와 최고점수를 HUD로 표시한다.
- `SPACE`로 시작/일시정지를 전환한다.
- 게임오버 후 재시작 흐름을 완성한다.

### 현재 구현 기준 참고

- 코드에 `speed = min(20, 9 + score // 3)` 로직이 포함되어 HUD 값은 증가한다.
- 이동 틱은 `pygame.time.set_timer(MOVE_EVENT, 100)`으로 고정되어 있어, 실제 이동 속도는 이 타이머 값의 영향을 가장 크게 받는다.

### 핵심 변수/함수

- `started`: 첫 시작 전 대기 상태인지 여부
- `paused`: 일시정지 여부
- `score`, `best_score`: 점수 상태
- `reset_round()`: 라운드 초기값 복구

### 단계별 구현

#### 단계 1) 상태/점수 변수 추가

##### 세부목표
  - 상태/점수 변수 추가 단계의 코드를 게임 루프에 연결해 실제 동작으로 확인한다.
  - 현재 단계의 상태값이 다음 단계 동작으로 어떻게 이어지는지 이해한다.

```python
score = 0
best_score = 0
started = False
paused = False
speed = 9
```

##### 선언한 변수/함수의 목적
  - `score`: 점수/시도 횟수 값
  - `best_score`: 점수/시도 횟수 값
  - `started`: 첫 시작 여부를 저장해 안내 화면과 플레이 상태 전환 시점을 제어한다.
  - `paused`: 일시정지 상태를 저장해 업데이트 루프 중단 여부를 제어한다.
  - `speed`: 게임 진행 속도 값으로 `clock.tick(speed)`에 들어가 템포를 조절한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `score = 0`: 현재 판 점수를 0부터 시작해 먹이 획득 시 증가량이 정확히 누적되게 한다.
  - `best_score = 0`: 최고 점수 기록을 별도로 유지해 게임오버/재시작 이후에도 이전 성과를 비교할 수 있게 한다.
  - `started = False`: 시작 전 상태를 명확히 두어 첫 입력 전에는 안내 화면을 유지하고 로직 오작동을 막는다.
  - `paused = False`: 일시정지 기본값을 `False`로 두어 게임 시작 시 업데이트 루프가 정상 진행되게 한다.

#### 단계 2) 먹이 섭취 시 점수/속도값 갱신

##### 세부목표
  - 리스트 조작으로 객체 상태를 갱신해 이동/성장/정리 규칙을 구현한다.
  - 현재 단계의 상태값이 다음 단계 동작으로 어떻게 이어지는지 이해한다.

```python
if new_head == food:
    score += 1
    best_score = max(best_score, score)
    food = place_food(snake)
    speed = min(20, 9 + score // 3)
else:
    snake.pop()
```

##### 선언한 변수/함수의 목적
  - `best_score`: 점수/시도 횟수 값
  - `food`: 먹이 좌표 값
  - `speed`: 게임 진행 속도 값으로 `clock.tick(speed)`에 들어가 템포를 조절한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `snake.pop()`: 리스트 마지막 요소를 제거해 길이 유지/정리 처리를 한다.
  - `if new_head == food:`: 새 머리 좌표가 먹이와 겹칠 때만 성장과 점수 증가를 적용한다.
  - `score += 1`: 먹이를 먹은 순간 점수를 1 올려, 성장 이벤트가 HUD 숫자 변화로 즉시 보이게 한다.
  - `best_score = max(best_score, score)`: 최고 점수 기록을 별도로 유지해 게임오버/재시작 이후에도 이전 성과를 비교할 수 있게 한다.

#### 단계 3) `SPACE`로 시작/일시정지 처리

##### 세부목표
  - 입력 이벤트를 상태값 변화와 연결해 사용자의 조작이 게임 동작으로 이어지게 만든다.
  - 학습 목표는 `if/elif` 분기로 입력 조건을 나누고 상태 업데이트 순서를 이해하는 것이다.

```python
if event.type == pygame.KEYDOWN:
    if event.key == pygame.K_SPACE and not game_over:
        if not started:
            started = True
        else:
            paused = not paused
```

##### 선언한 변수/함수의 목적
  - `started`: 첫 시작 여부를 저장해 안내 화면과 플레이 상태 전환 시점을 제어한다.
  - `paused`: 일시정지 상태를 저장해 업데이트 루프 중단 여부를 제어한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if event.type == pygame.KEYDOWN:`: 이벤트 타입을 비교해 입력/업데이트 처리를 분기한다.
  - `if event.key == pygame.K_SPACE and not game_over:`: 게임오버가 아닐 때의 Space 입력만 시작/일시정지 제어로 연결한다.
  - `if not started:`: 아직 시작 전일 때만 시작 플래그를 바꿔 첫 입력과 이후 입력 동작을 구분한다.
  - `started = True`: 시작 전 상태를 명확히 두어 첫 입력 전에는 안내 화면을 유지하고 로직 오작동을 막는다.

#### 단계 4) 업데이트 조건 분리

##### 세부목표
  - 입력 이벤트를 상태값 변화와 연결해 사용자의 조작이 게임 동작으로 이어지게 만든다.
  - 학습 목표는 `if/elif` 분기로 입력 조건을 나누고 상태 업데이트 순서를 이해하는 것이다.

```python
if event.type == MOVE_EVENT and started and not paused and not game_over:
    # 이동/충돌/먹이 업데이트
    pass
```

##### 선언한 변수/함수의 목적
  - `state`: 초기화 함수 반환 상태를 잠깐 담아 여러 상태 변수를 한 번에 갱신한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if event.type == MOVE_EVENT and started and not paused and not game_over:`: 이벤트 타입을 비교해 입력/업데이트 처리를 분기한다.
  - `pass`: 아직 동작을 채우지 않은 자리로 두되, 문법을 유지해 루프 흐름을 계속 진행하게 한다.

#### 단계 5) HUD + 상태 메시지 표시

##### 세부목표
  - HUD + 상태 메시지 표시 단계의 코드를 게임 루프에 연결해 실제 동작으로 확인한다.
  - 학습 목표는 상태 갱신 이후 렌더링 순서를 지켜 화면과 내부 데이터가 일치하도록 만드는 것이다.

```python
hud = small.render(f"Score: {score} Best: {best_score} Speed: {speed}", True, TEXT)
screen.blit(hud, (10, 10))
```

```python
if not started:
    msg = font.render("Press SPACE to Start", True, TEXT)
elif paused:
    msg = font.render("Paused (SPACE to Resume)", True, TEXT)
```

##### 선언한 변수/함수의 목적
  - `hud`: 점수/속도 정보를 합쳐 렌더링한 HUD 텍스트 Surface다.
  - `msg`: 상황별 안내 문구를 렌더링한 보조 텍스트 Surface다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `msg = font.render("Press SPACE to Start", True, TEXT)`: 문자열을 화면에 표시할 텍스트 Surface로 변환한다.
  - `screen.blit(hud, (10, 10))`: Surface를 지정 위치에 배치해 UI/텍스트를 출력한다.
  - `if not started:`: 아직 시작 전일 때만 시작 플래그를 바꿔 첫 입력과 이후 입력 동작을 구분한다.
  - `hud = small.render(f"Score: {score} Best: {best_score} Speed: {speed}", True, TEXT)`: 점수·최고점·속도 정보를 한 번에 렌더링한 텍스트로 만들어 HUD 영역에 즉시 출력할 수 있게 한다.

### class 4 최종 코드

```python
import pygame
import random
import sys

pygame.init()

CELL_SIZE = 20
GRID_W = 30
GRID_H = 20
WIDTH = CELL_SIZE * GRID_W
HEIGHT = CELL_SIZE * GRID_H

BG = (20, 22, 26)
GRID = (45, 49, 58)
HEAD = (145, 255, 165)
BODY = (92, 210, 116)
FOOD = (255, 105, 105)
TEXT = (238, 242, 245)

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Snake Codelab - Session 4")
clock = pygame.time.Clock()
font = pygame.font.SysFont(None, 36)
small = pygame.font.SysFont(None, 24)

MOVE_EVENT = pygame.USEREVENT + 1
pygame.time.set_timer(MOVE_EVENT, 100)

def draw_grid(surface):
    for x in range(0, WIDTH, CELL_SIZE):
        pygame.draw.line(surface, GRID, (x, 0), (x, HEIGHT))
    for y in range(0, HEIGHT, CELL_SIZE):
        pygame.draw.line(surface, GRID, (0, y), (WIDTH, y))

def draw_cell(surface, color, gx, gy):
    rect = pygame.Rect(gx * CELL_SIZE, gy * CELL_SIZE, CELL_SIZE, CELL_SIZE)
    pygame.draw.rect(surface, color, rect)

def place_food(snake_body):
    while True:
        pos = (random.randint(0, GRID_W - 1), random.randint(0, GRID_H - 1))
        if pos not in snake_body:
            return pos

def reset_round():
    snake_body = [(10, 10), (9, 10), (8, 10)]
    direction_vec = (1, 0)
    food_pos = place_food(snake_body)
    return snake_body, direction_vec, food_pos

snake, direction, food = reset_round()
game_over = False
started = False
paused = False
score = 0
best_score = 0
speed = 9

while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE and not game_over:
                if not started:
                    started = True
                else:
                    paused = not paused

            if not game_over:
                if event.key == pygame.K_UP and direction != (0, 1):
                    direction = (0, -1)
                elif event.key == pygame.K_DOWN and direction != (0, -1):
                    direction = (0, 1)
                elif event.key == pygame.K_LEFT and direction != (1, 0):
                    direction = (-1, 0)
                elif event.key == pygame.K_RIGHT and direction != (-1, 0):
                    direction = (1, 0)
            else:
                if event.key == pygame.K_r:
                    snake, direction, food = reset_round()
                    game_over = False
                    started = False
                    paused = False
                    score = 0
                    speed = 9
                elif event.key == pygame.K_q:
                    pygame.quit()
                    sys.exit()

        if event.type == MOVE_EVENT and started and not paused and not game_over:
            head_x, head_y = snake[0]
            dx, dy = direction
            new_head = (head_x + dx, head_y + dy)

            hit_wall = (
                new_head[0] < 0 or new_head[0] >= GRID_W or
                new_head[1] < 0 or new_head[1] >= GRID_H
            )
            hit_self = new_head in snake

            if hit_wall or hit_self:
                game_over = True
            else:
                snake.insert(0, new_head)
                if new_head == food:
                    score += 1
                    best_score = max(best_score, score)
                    food = place_food(snake)
                    speed = min(20, 9 + score // 3)
                else:
                    snake.pop()

    screen.fill(BG)
    draw_grid(screen)
    draw_cell(screen, FOOD, food[0], food[1])

    for i, part in enumerate(snake):
        draw_cell(screen, HEAD if i == 0 else BODY, part[0], part[1])

    hud = small.render(f"Score: {score} Best: {best_score} Speed: {speed}", True, TEXT)
    screen.blit(hud, (10, 10))

    if not started and not game_over:
        msg = font.render("Press SPACE to Start", True, TEXT)
        screen.blit(msg, (WIDTH // 2 - msg.get_width() // 2, HEIGHT // 2 - 20))
    elif paused and not game_over:
        msg = font.render("Paused (SPACE to Resume)", True, TEXT)
        screen.blit(msg, (WIDTH // 2 - msg.get_width() // 2, HEIGHT // 2 - 20))
    elif game_over:
        m1 = font.render("Game Over", True, TEXT)
        m2 = small.render("R: Restart / Q: Quit", True, TEXT)
        screen.blit(m1, (WIDTH // 2 - m1.get_width() // 2, HEIGHT // 2 - 28))
        screen.blit(m2, (WIDTH // 2 - m2.get_width() // 2, HEIGHT // 2 + 10))

    pygame.display.flip()
    clock.tick(speed)
```

---

## 진행 팁(부모용)

- 각 class의 최종 코드는 반드시 실행해 보고 다음 class로 넘어간다.
- 에러가 났을 때는 "마지막으로 바꾼 5줄"만 먼저 되돌려 확인한다.
- class 4 이후 확장 과제 예시
  - 벽 통과 모드/충돌 모드 토글
  - 난수 시드 고정 후 리플레이 비교
  - 최고점수 파일 저장(`json`)
