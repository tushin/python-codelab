## 개요

- 대상: `pygame`으로 기본 게임 루프를 구현해본 뒤 타일맵 아케이드를 만들고 싶은 학습자
- 방식: class별로 "실행 코드 + 기능 1개 확장" 방식
- 최종 산출물: 미로 이동, 점수, 유령 추적, 생명/재시작이 포함된 팩맨 게임 1개
- 실행 환경: Python 3.10+ / `pip install pygame`

## 목차

1. class 1. 타일맵 + 팩맨 이동
2. class 2. 점 먹기 + 점수 + 벽 충돌 정교화
3. class 3. 유령 이동 + 충돌 + 생명/게임오버
4. class 4. 파워펠릿 + 취약모드 + 폴리싱

---

## class 1. 타일맵 + 팩맨 이동

### 목표

- 문자 기반 타일맵으로 미로를 만든다.
- 화살표 입력으로 팩맨을 이동시킨다.
- 벽 타일은 통과하지 못하도록 막는다.

### 핵심 변수/함수

- `LEVEL`: 미로 문자열 목록
- `TILE`: 타일 픽셀 크기
- `pac_x`, `pac_y`: 팩맨 위치(타일 좌표)
- `can_move()`: 목표 칸 이동 가능 여부 판정

### 단계별 구현

#### 단계 1) 미로 데이터와 기본 창

##### 세부목표
  - 타일맵 데이터를 만들고 화면 크기를 자동 계산한다.
  - 미로 크기와 렌더링 범위를 같은 기준으로 통일한다.

```python
import pygame
import sys

pygame.init()

LEVEL = [
    "###################",
    "#........#........#",
    "#.###.##.#.##.###.#",
    "#.................#",
    "#.###.#.###.#.###.#",
    "#.....#..P..#.....#",
    "###################",
]

TILE = 36
ROWS = len(LEVEL)
COLS = len(LEVEL[0])
WIDTH, HEIGHT = COLS * TILE, ROWS * TILE

screen = pygame.display.set_mode((WIDTH, HEIGHT))
clock = pygame.time.Clock()
```

##### 선언한 변수/함수의 목적
  - `LEVEL`: 미로 타일맵 데이터
  - `TILE`: 타일 1칸 크기
  - `ROWS`, `COLS`: 미로 행/열 크기
  - `screen`: 화면 출력 대상 객체

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `LEVEL = [...]`: 문자열 맵으로 벽/길/시작 위치를 한눈에 관리해 레벨 수정을 쉽게 만든다.
  - `ROWS = len(LEVEL)`: 행 수를 자동 계산해 맵 크기 변경 시 코드 수정을 줄인다.
  - `COLS = len(LEVEL[0])`: 첫 줄 길이를 열 기준으로 사용해 가로 충돌 범위를 고정한다.
  - `WIDTH, HEIGHT = COLS * TILE, ROWS * TILE`: 타일 수를 픽셀 크기로 변환해 창 크기와 맵 범위를 일치시킨다.

#### 단계 2) 시작 위치 찾기

##### 세부목표
  - 맵에서 `P` 문자를 찾아 팩맨 시작 좌표를 결정한다.
  - 하드코딩 좌표 없이 레벨 데이터 기반으로 초기화를 구성한다.

```python
pac_x, pac_y = 1, 1
for r, row in enumerate(LEVEL):
    for c, ch in enumerate(row):
        if ch == "P":
            pac_x, pac_y = c, r
```

##### 선언한 변수/함수의 목적
  - `pac_x`, `pac_y`: 팩맨 현재 타일 좌표
  - `r`, `c`: 탐색 중인 맵 좌표

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `for r, row in enumerate(LEVEL):`: 맵 모든 행을 순회해 시작 위치 문자를 탐색한다.
  - `for c, ch in enumerate(row):`: 각 행의 문자 인덱스를 읽어 열 좌표까지 함께 확보한다.
  - `if ch == "P":`: 시작 마커를 조건으로 잡아 초기 위치를 레벨 데이터에서 추출한다.
  - `pac_x, pac_y = c, r`: 찾은 좌표를 상태값으로 저장해 첫 렌더링부터 정확한 위치에 표시한다.

#### 단계 3) 이동 가능 판정 함수

##### 세부목표
  - 벽(`#`) 여부를 기준으로 이동 가능 여부를 판단한다.
  - 경계 밖 좌표 접근을 먼저 차단해 인덱스 오류를 막는다.

```python
def can_move(nx, ny):
    if nx < 0 or nx >= COLS or ny < 0 or ny >= ROWS:
        return False
    return LEVEL[ny][nx] != "#"
```

##### 선언한 변수/함수의 목적
  - `can_move()`: 이동 가능 판정 함수
  - `nx`, `ny`: 이동 후보 좌표

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if nx < 0 or nx >= COLS ...`: 맵 경계 밖 좌표를 먼저 차단해 잘못된 인덱스 접근을 예방한다.
  - `LEVEL[ny][nx] != "#"`: 목표 칸이 벽이 아닐 때만 이동을 허용한다.
  - `return False`: 불가능한 이동을 명시적으로 반환해 입력 처리 분기를 단순화한다.
  - `can_move(nx, ny)` 형태로 좌표만 받도록 분리해 팩맨/유령 양쪽 로직에서 재사용 가능하게 만든다.

#### 단계 4) 입력 이동 + 타일 렌더링

##### 세부목표
  - 방향키 입력을 타일 좌표 이동으로 연결한다.
  - 벽/길/팩맨을 화면에 렌더링한다.

```python
if event.type == pygame.KEYDOWN:
    nx, ny = pac_x, pac_y
    if event.key == pygame.K_LEFT:
        nx -= 1
    elif event.key == pygame.K_RIGHT:
        nx += 1
    elif event.key == pygame.K_UP:
        ny -= 1
    elif event.key == pygame.K_DOWN:
        ny += 1
    if can_move(nx, ny):
        pac_x, pac_y = nx, ny
```

##### 선언한 변수/함수의 목적
  - `nx`, `ny`: 이동 계산용 임시 좌표

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `nx, ny = pac_x, pac_y`: 원본 좌표를 복사해 후보 좌표를 먼저 계산한다.
  - `if event.key == pygame.K_LEFT:`: 입력 방향마다 좌표 변화를 분기해 의도한 한 칸 이동을 만든다.
  - `if can_move(nx, ny):`: 후보 칸이 이동 가능할 때만 실제 상태를 갱신해 벽 관통을 막는다.
  - `pac_x, pac_y = nx, ny`: 승인된 이동만 반영해 렌더링과 충돌 기준이 항상 일치하게 유지된다.

#### class 1 최종 코드

```python
import pygame
import sys

pygame.init()

LEVEL = [
    "###################",
    "#........#........#",
    "#.###.##.#.##.###.#",
    "#.................#",
    "#.###.#.###.#.###.#",
    "#.....#..P..#.....#",
    "###################",
]

TILE = 36
ROWS = len(LEVEL)
COLS = len(LEVEL[0])
WIDTH, HEIGHT = COLS * TILE, ROWS * TILE

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Pac-Man - Class 1")
clock = pygame.time.Clock()

pac_x, pac_y = 1, 1
for r, row in enumerate(LEVEL):
    for c, ch in enumerate(row):
        if ch == "P":
            pac_x, pac_y = c, r


def can_move(nx, ny):
    if nx < 0 or nx >= COLS or ny < 0 or ny >= ROWS:
        return False
    return LEVEL[ny][nx] != "#"


while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

        if event.type == pygame.KEYDOWN:
            nx, ny = pac_x, pac_y
            if event.key == pygame.K_LEFT:
                nx -= 1
            elif event.key == pygame.K_RIGHT:
                nx += 1
            elif event.key == pygame.K_UP:
                ny -= 1
            elif event.key == pygame.K_DOWN:
                ny += 1
            if can_move(nx, ny):
                pac_x, pac_y = nx, ny

    screen.fill((2, 6, 23))
    for y, row in enumerate(LEVEL):
        for x, ch in enumerate(row):
            if ch == "#":
                pygame.draw.rect(screen, (30, 64, 175), (x * TILE, y * TILE, TILE, TILE), border_radius=6)
            else:
                pygame.draw.rect(screen, (15, 23, 42), (x * TILE, y * TILE, TILE, TILE))

    cx = pac_x * TILE + TILE // 2
    cy = pac_y * TILE + TILE // 2
    pygame.draw.circle(screen, (250, 204, 21), (cx, cy), TILE // 2 - 4)

    pygame.display.flip()
    clock.tick(60)
```

---

## class 2. 점 먹기 + 점수 + 벽 충돌 정교화

### 목표

- 길 타일의 점(`.`)을 먹으면 점수를 올린다.
- 점을 먹은 칸은 빈 칸으로 변경한다.
- 남은 점이 0이면 클리어 상태로 전환한다.

### 핵심 변수/함수

- `board`: 수정 가능한 맵 배열(문자 2차원 리스트)
- `score`: 누적 점수
- `dots_left`: 남은 점 개수
- `eat_dot()`: 현재 칸 점 먹기 처리

### 단계별 구현

#### 단계 1) 수정 가능한 맵으로 변환

##### 세부목표
  - 문자열 불변 문제를 피하기 위해 리스트 맵으로 변환한다.
  - 점 개수를 미리 세어 클리어 조건 계산에 사용한다.

```python
board = [list(row.replace("P", " ")) for row in LEVEL]
dots_left = sum(row.count(".") for row in board)
score = 0
```

##### 선언한 변수/함수의 목적
  - `board`: 게임 중 수정되는 맵 데이터
  - `dots_left`: 남은 점 개수
  - `score`: 누적 점수

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `list(row.replace("P", " "))`: 시작 마커를 빈칸으로 바꿔 이동 로직과 점수 로직 충돌을 막는다.
  - `board = [...]`: 문자열 대신 문자 리스트를 사용해 칸 단위 수정(`board[y][x] = " "`)이 가능해진다.
  - `sum(row.count(".") for row in board)`: 남은 점을 초기 계산해 클리어 조건 판정을 단순화한다.
  - `score = 0`: 점수 상태를 명확히 초기화해 라운드 진행에 따라 누적되게 만든다.

#### 단계 2) 점 먹기 함수

##### 세부목표
  - 팩맨 현재 칸이 점이면 점수와 남은 점을 갱신한다.
  - 같은 칸 재방문으로 중복 점수 획득이 되지 않게 막는다.

```python
def eat_dot(px, py):
    global score, dots_left
    if board[py][px] == ".":
        board[py][px] = " "
        score += 10
        dots_left -= 1
```

##### 선언한 변수/함수의 목적
  - `eat_dot()`: 점 먹기 처리 함수
  - `score`, `dots_left`: 진행 상태값

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if board[py][px] == ".":`: 현재 칸이 점일 때만 먹기 로직을 실행해 중복 처리 오류를 막는다.
  - `board[py][px] = " "`: 먹은 칸을 빈칸으로 바꿔 다음 방문 시 점수가 다시 오르지 않게 만든다.
  - `score += 10`: 점 1개당 점수를 증가시켜 진행 보상을 즉시 반영한다.
  - `dots_left -= 1`: 남은 점 수를 줄여 클리어 조건(`dots_left == 0`)과 동기화한다.

#### 단계 3) 이동 후 점 먹기 연결

##### 세부목표
  - 실제 이동이 성공한 경우에만 점 먹기 함수를 호출한다.
  - 벽 앞 입력에서는 점수/점 상태가 바뀌지 않게 유지한다.

```python
if can_move(nx, ny):
    pac_x, pac_y = nx, ny
    eat_dot(pac_x, pac_y)
```

##### 선언한 변수/함수의 목적
  - `pac_x`, `pac_y`: 현재 팩맨 좌표

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if can_move(nx, ny):`: 이동 허용 여부를 먼저 확정해 실패 입력에서 상태 오염을 막는다.
  - `pac_x, pac_y = nx, ny`: 이동 성공 시 현재 좌표를 갱신한다.
  - `eat_dot(pac_x, pac_y)`: 새 위치 칸을 즉시 검사해 점수 반영 지연을 없앤다.
  - 이동과 점수 처리를 같은 분기에서 연결해 화면 위치와 점수 변화 타이밍을 맞춘다.

#### 단계 4) 클리어/점수 HUD

##### 세부목표
  - 점수와 남은 점 수를 HUD로 표시한다.
  - 점을 전부 먹으면 클리어 문구를 출력한다.

```python
hud = small.render(f"Score {score}  Dots {dots_left}", True, (226, 232, 240))
screen.blit(hud, (10, 8))
if dots_left == 0:
    clear = font.render("Stage Clear!", True, (134, 239, 172))
```

##### 선언한 변수/함수의 목적
  - `hud`: 상태 HUD 텍스트
  - `clear`: 클리어 안내 텍스트

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `f"Score {score}  Dots {dots_left}"`: 핵심 진행 정보를 한 줄로 묶어 현재 목표 상태를 즉시 확인하게 한다.
  - `screen.blit(hud, (10, 8))`: 상단 고정 배치로 플레이 중에도 쉽게 읽히게 만든다.
  - `if dots_left == 0:`: 남은 점이 0일 때만 클리어 상태를 표시해 조건 판정을 명확히 한다.
  - `font.render("Stage Clear!", ...)`: 성공 상태를 시각적으로 강조해 다음 단계 전환 신호를 준다.

#### class 2 최종 코드

```python
# class 1 코드에 board/eat_dot/score/dots_left/HUD를 통합해 실행
# 핵심: 이동 성공 후 점 먹기 처리, dots_left==0 클리어 표시
```

---

## class 3. 유령 이동 + 충돌 + 생명/게임오버

### 목표

- 유령이 랜덤/추적 규칙으로 이동한다.
- 팩맨과 유령이 같은 칸이면 생명을 깎는다.
- 생명이 0이면 게임오버 처리한다.

### 핵심 변수/함수

- `ghosts`: 유령 상태 목록(`x`, `y`, `dir`)
- `lives`: 남은 생명
- `move_ghost()`: 유령 이동 함수
- `reset_positions()`: 팩맨/유령 위치 초기화

### 단계별 구현

#### 단계 1) 유령 상태 추가

##### 세부목표
  - 유령 여러 개를 타일 좌표로 배치한다.
  - 이동 방향 상태를 함께 저장한다.

```python
ghosts = [
    {"x": 9, "y": 1, "dir": (1, 0)},
    {"x": 17, "y": 5, "dir": (-1, 0)},
]
lives = 3
game_over = False
```

##### 선언한 변수/함수의 목적
  - `ghosts`: 유령 목록
  - `lives`: 남은 생명
  - `game_over`: 종료 상태 플래그

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `ghosts = [...]`: 유령 좌표와 방향을 딕셔너리로 관리해 이동 규칙을 개별 적용할 수 있게 만든다.
  - `"dir": (1, 0)`: 직전 진행 방향을 저장해 가능한 경우 같은 방향 유지 이동이 가능해진다.
  - `lives = 3`: 충돌 즉시 종료가 아닌 다회 기회 구조를 만든다.
  - `game_over = False`: 기본 상태를 진행 가능으로 두고 충돌 조건에서만 종료로 전환한다.

#### 단계 2) 유령 이동 함수

##### 세부목표
  - 벽이 아닌 방향 후보를 골라 유령을 이동시킨다.
  - 단순 추적 성향을 넣어 플레이 긴장감을 만든다.

```python
import random


def move_ghost(g):
    candidates = []
    for dx, dy in [(-1, 0), (1, 0), (0, -1), (0, 1)]:
        nx, ny = g["x"] + dx, g["y"] + dy
        if can_move(nx, ny):
            candidates.append((dx, dy))

    if not candidates:
        return

    candidates.sort(key=lambda d: abs((g["x"] + d[0]) - pac_x) + abs((g["y"] + d[1]) - pac_y))
    pick = candidates[0] if random.random() < 0.65 else random.choice(candidates)
    g["x"] += pick[0]
    g["y"] += pick[1]
```

##### 선언한 변수/함수의 목적
  - `move_ghost()`: 유령 이동 제어 함수
  - `candidates`: 이동 가능한 방향 목록
  - `pick`: 최종 선택 방향

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if can_move(nx, ny): candidates.append(...)`: 벽을 통과하지 않는 방향만 후보로 남긴다.
  - `candidates.sort(key=...)`: 팩맨까지의 맨해튼 거리를 기준으로 추적 우선순위를 계산한다.
  - `random.random() < 0.65`: 항상 추적만 하지 않고 일부 랜덤성을 섞어 패턴 고정을 줄인다.
  - `g["x"] += pick[0]`: 선택 방향을 실제 좌표로 반영해 유령 이동 상태를 갱신한다.

#### 단계 3) 충돌 판정 + 생명 감소

##### 세부목표
  - 팩맨과 유령 좌표가 겹치면 생명을 줄인다.
  - 생명이 남으면 위치 리셋, 없으면 게임오버 처리한다.

```python
def reset_positions():
    global pac_x, pac_y
    pac_x, pac_y = 9, 5
    ghosts[0]["x"], ghosts[0]["y"] = 9, 1
    ghosts[1]["x"], ghosts[1]["y"] = 17, 5

for g in ghosts:
    if g["x"] == pac_x and g["y"] == pac_y:
        lives -= 1
        if lives <= 0:
            game_over = True
        else:
            reset_positions()
        break
```

##### 선언한 변수/함수의 목적
  - `reset_positions()`: 위치 초기화 함수
  - `lives`: 생명 상태값

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if g["x"] == pac_x and g["y"] == pac_y:`: 타일 좌표가 완전히 같을 때 충돌로 판정해 규칙을 단순화한다.
  - `lives -= 1`: 충돌 결과를 생명 감소로 누적해 게임 지속 여부를 판단한다.
  - `if lives <= 0: game_over = True`: 남은 생명이 없으면 종료 상태로 전환한다.
  - `reset_positions()`: 생명이 남아 있으면 즉시 초기 위치로 복귀해 다음 시도를 시작한다.

#### 단계 4) 생명/게임오버 HUD

##### 세부목표
  - 점수와 생명을 함께 표시한다.
  - 종료 상태에서 재시작 안내를 출력한다.

```python
hud = small.render(f"Score {score}  Lives {lives}", True, (226, 232, 240))
screen.blit(hud, (10, 8))
if game_over:
    over = font.render("Game Over", True, (254, 202, 202))
```

##### 선언한 변수/함수의 목적
  - `hud`: 진행 정보 HUD
  - `over`: 종료 안내 텍스트

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `f"Score {score}  Lives {lives}"`: 점수와 생명 정보를 한 줄로 묶어 현재 위험도를 바로 파악하게 한다.
  - `screen.blit(hud, (10, 8))`: 상단 고정 HUD로 이동 중에도 상태 확인이 가능하게 한다.
  - `if game_over:`: 종료 상태에서만 오버레이 메시지를 렌더링해 플레이 화면과 구분한다.
  - `font.render("Game Over", ...)`: 실패 상태를 큰 텍스트로 강조해 즉시 인지하도록 만든다.

#### class 3 최종 코드

```python
# class 2 코드에 ghosts/move_ghost/lives/game_over/reset_positions를 통합해 실행
# 핵심: 유령 이동 후 좌표 충돌 판정, 생명 감소 및 종료 처리
```

---

## class 4. 파워펠릿 + 취약모드 + 폴리싱

### 목표

- 파워펠릿(`o`)을 먹으면 일정 시간 유령이 취약해진다.
- 취약 유령을 먹으면 점수를 추가한다.
- 최종 실행 가능한 팩맨 게임 코드를 완성한다.

### 핵심 변수/함수

- `fright_until`: 취약모드 종료 시각
- `is_frightened()`: 취약 상태 판정 함수
- `bonus_chain`: 연속 유령 점수 배수 단계

### 단계별 구현

#### 단계 1) 파워펠릿 처리

##### 세부목표
  - 파워펠릿 칸을 먹으면 취약모드를 시작한다.
  - 취약 시간 동안 유령 처리 규칙을 바꾼다.

```python
fright_until = 0
bonus_chain = 0

if board[pac_y][pac_x] == "o":
    board[pac_y][pac_x] = " "
    fright_until = pygame.time.get_ticks() + 7000
    bonus_chain = 0


def is_frightened(now):
    return now < fright_until
```

##### 선언한 변수/함수의 목적
  - `fright_until`: 취약 종료 시각
  - `bonus_chain`: 연속 유령 보너스 단계
  - `is_frightened()`: 취약 상태 판정 함수

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `fright_until = ... + 7000`: 파워펠릿 효과 지속 시간을 절대 시각으로 저장해 프레임 차이 영향을 줄인다.
  - `board[pac_y][pac_x] = " "`: 먹은 파워펠릿을 제거해 중복 활성화를 막는다.
  - `bonus_chain = 0`: 새 취약 구간 시작 시 연속 보너스를 초기화한다.
  - `return now < fright_until`: 현재 시각이 종료 시각 이전인지 비교해 취약 여부를 빠르게 판정한다.

#### 단계 2) 취약 유령 충돌 처리

##### 세부목표
  - 취약 상태에서는 유령을 먹고 점수를 얻는다.
  - 일반 상태에서는 기존처럼 생명을 잃는다.

```python
now = pygame.time.get_ticks()
for g in ghosts:
    if g["x"] == pac_x and g["y"] == pac_y:
        if is_frightened(now):
            score += 200 * (2 ** bonus_chain)
            bonus_chain = min(3, bonus_chain + 1)
            g["x"], g["y"] = 9, 1
        else:
            lives -= 1
```

##### 선언한 변수/함수의 목적
  - `now`: 현재 시각
  - `bonus_chain`: 연속 보너스 단계

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if is_frightened(now):`: 취약모드 여부에 따라 충돌 결과를 점수 획득/피해로 분기한다.
  - `score += 200 * (2 ** bonus_chain)`: 같은 취약 구간에서 연속으로 유령을 먹을수록 보너스를 키운다.
  - `bonus_chain = min(3, bonus_chain + 1)`: 보너스 단계 상한을 둬 점수 폭주를 제한한다.
  - `g["x"], g["y"] = 9, 1`: 먹힌 유령을 스폰 위치로 되돌려 즉시 화면에서 분리한다.

#### 단계 3) 유령 렌더링 색상 분기

##### 세부목표
  - 취약모드에서 유령 색상을 바꿔 상태를 시각화한다.
  - 플레이어가 위험/기회 상태를 즉시 구분하게 한다.

```python
ghost_color = (96, 165, 250) if is_frightened(now) else (248, 113, 113)
for g in ghosts:
    gx = g["x"] * TILE + TILE // 2
    gy = g["y"] * TILE + TILE // 2
    pygame.draw.circle(screen, ghost_color, (gx, gy), TILE // 2 - 6)
```

##### 선언한 변수/함수의 목적
  - `ghost_color`: 현재 유령 색상
  - `gx`, `gy`: 유령 픽셀 좌표

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `ghost_color = ... if ... else ...`: 취약 상태일 때 색을 바꿔 유령 처치 가능 구간을 명확히 표시한다.
  - `for g in ghosts:`: 모든 유령에 같은 시각 규칙을 적용해 상태 인지를 일관되게 만든다.
  - `gx = g["x"] * TILE + TILE // 2`: 타일 좌표를 원 중심 픽셀 좌표로 변환해 정확히 가운데에 렌더링한다.
  - `pygame.draw.circle(...)`: 유령을 원형으로 간단히 표현해 팩맨과 충돌 범위를 직관적으로 맞춘다.

#### 단계 4) 최종 루프 정리

##### 세부목표
  - 이동/점수/유령/취약모드/HUD를 하나의 루프로 통합한다.
  - 학습용으로 바로 실행 가능한 최종 버전을 완성한다.

```python
# 루프 순서
# 1) 입력 처리
# 2) 팩맨 이동 + 점/파워펠릿 처리
# 3) 유령 이동
# 4) 충돌 판정(취약/일반)
# 5) 렌더링 + HUD
```

##### 선언한 변수/함수의 목적
  - `loop order`: 업데이트 단계 순서 규칙

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - 입력을 먼저 처리해 프레임 지연 없이 팩맨 이동 의도를 반영한다.
  - 이동 후 점/파워펠릿을 처리해 좌표와 점수 변화 타이밍을 맞춘다.
  - 유령 이동 후 충돌 판정을 수행해 같은 프레임 위치 기준으로 결과를 확정한다.
  - 마지막에 렌더링/HUD를 그려 갱신된 상태값이 화면에 즉시 반영되게 만든다.

#### class 4 최종 코드

```python
import pygame
import random
import sys

pygame.init()

LEVEL = [
    "###################",
    "#o.......#.......o#",
    "#.###.##.#.##.###.#",
    "#.................#",
    "#.###.#.###.#.###.#",
    "#.....#..P..#.....#",
    "#.................#",
    "#.###.##.#.##.###.#",
    "#o.......#.......o#",
    "###################",
]

TILE = 36
ROWS = len(LEVEL)
COLS = len(LEVEL[0])
WIDTH, HEIGHT = COLS * TILE, ROWS * TILE

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Pac-Man - Final")
clock = pygame.time.Clock()
font = pygame.font.SysFont("arial", 44, bold=True)
small = pygame.font.SysFont("arial", 24)

board = [list(row.replace("P", " ")) for row in LEVEL]
score = 0
lives = 3
game_over = False
fright_until = 0
bonus_chain = 0

pac_x, pac_y = 1, 1
for r, row in enumerate(LEVEL):
    for c, ch in enumerate(row):
        if ch == "P":
            pac_x, pac_y = c, r

start_pac = (pac_x, pac_y)

ghosts = [
    {"x": 9, "y": 1, "dir": (1, 0)},
    {"x": 17, "y": 5, "dir": (-1, 0)},
]
start_ghosts = [(9, 1), (17, 5)]


def can_move(nx, ny):
    if nx < 0 or nx >= COLS or ny < 0 or ny >= ROWS:
        return False
    return board[ny][nx] != "#"


def is_frightened(now):
    return now < fright_until


def move_ghost(g):
    candidates = []
    for dx, dy in [(-1, 0), (1, 0), (0, -1), (0, 1)]:
        nx, ny = g["x"] + dx, g["y"] + dy
        if can_move(nx, ny):
            candidates.append((dx, dy))

    if not candidates:
        return

    candidates.sort(key=lambda d: abs((g["x"] + d[0]) - pac_x) + abs((g["y"] + d[1]) - pac_y))
    pick = candidates[0] if random.random() < 0.65 else random.choice(candidates)
    g["x"] += pick[0]
    g["y"] += pick[1]


def reset_positions():
    global pac_x, pac_y
    pac_x, pac_y = start_pac
    for i, g in enumerate(ghosts):
        g["x"], g["y"] = start_ghosts[i]


def reset_game():
    global board, score, lives, game_over, fright_until, bonus_chain
    board = [list(row.replace("P", " ")) for row in LEVEL]
    score = 0
    lives = 3
    game_over = False
    fright_until = 0
    bonus_chain = 0
    reset_positions()


while True:
    now = pygame.time.get_ticks()

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_q:
                pygame.quit()
                sys.exit()
            if event.key == pygame.K_r and game_over:
                reset_game()

            if not game_over:
                nx, ny = pac_x, pac_y
                if event.key == pygame.K_LEFT:
                    nx -= 1
                elif event.key == pygame.K_RIGHT:
                    nx += 1
                elif event.key == pygame.K_UP:
                    ny -= 1
                elif event.key == pygame.K_DOWN:
                    ny += 1

                if can_move(nx, ny):
                    pac_x, pac_y = nx, ny

                    if board[pac_y][pac_x] == ".":
                        board[pac_y][pac_x] = " "
                        score += 10
                    elif board[pac_y][pac_x] == "o":
                        board[pac_y][pac_x] = " "
                        score += 50
                        fright_until = now + 7000
                        bonus_chain = 0

    if not game_over:
        for g in ghosts:
            move_ghost(g)

        for g in ghosts:
            if g["x"] == pac_x and g["y"] == pac_y:
                if is_frightened(now):
                    score += 200 * (2 ** bonus_chain)
                    bonus_chain = min(3, bonus_chain + 1)
                    g["x"], g["y"] = start_ghosts[0]
                else:
                    lives -= 1
                    if lives <= 0:
                        game_over = True
                    else:
                        reset_positions()
                    break

    screen.fill((2, 6, 23))

    for y in range(ROWS):
        for x in range(COLS):
            ch = board[y][x]
            rect = pygame.Rect(x * TILE, y * TILE, TILE, TILE)
            if ch == "#":
                pygame.draw.rect(screen, (30, 64, 175), rect, border_radius=6)
            else:
                pygame.draw.rect(screen, (15, 23, 42), rect)
                if ch == ".":
                    pygame.draw.circle(screen, (241, 245, 249), rect.center, 3)
                elif ch == "o":
                    pygame.draw.circle(screen, (250, 204, 21), rect.center, 7)

    pac_pos = (pac_x * TILE + TILE // 2, pac_y * TILE + TILE // 2)
    pygame.draw.circle(screen, (250, 204, 21), pac_pos, TILE // 2 - 4)

    ghost_color = (96, 165, 250) if is_frightened(now) else (248, 113, 113)
    for g in ghosts:
        gx = g["x"] * TILE + TILE // 2
        gy = g["y"] * TILE + TILE // 2
        pygame.draw.circle(screen, ghost_color, (gx, gy), TILE // 2 - 6)

    dots_left = sum(row.count(".") + row.count("o") for row in board)
    hud = small.render(f"Score {score}  Lives {lives}  Dots {dots_left}", True, (226, 232, 240))
    screen.blit(hud, (10, 8))

    if dots_left == 0 and not game_over:
        clear = font.render("Stage Clear!", True, (134, 239, 172))
        screen.blit(clear, (WIDTH // 2 - clear.get_width() // 2, HEIGHT // 2 - 20))

    if game_over:
        over = font.render("Game Over", True, (254, 202, 202))
        hint = small.render("R: Restart   Q: Quit", True, (254, 226, 226))
        screen.blit(over, (WIDTH // 2 - over.get_width() // 2, HEIGHT // 2 - 30))
        screen.blit(hint, (WIDTH // 2 - hint.get_width() // 2, HEIGHT // 2 + 20))

    pygame.display.flip()
    clock.tick(8)
```
