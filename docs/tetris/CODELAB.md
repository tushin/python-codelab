## 개요

- 대상: `pygame`으로 처음 블록 퍼즐을 만드는 초급~중급 학습자
- 방식: class별로 "실행 코드 + 기능 1개 확장" 방식
- 최종 산출물: 테트리미노 회전, 줄 삭제, 점수 계산이 가능한 테트리스 게임 1개
- 실행 환경: Python 3.10+ / `pip install pygame`

## 목차

1. class 1. 보드 만들기 + 블록 이동
2. class 2. 회전 + 고정 + 다음 블록
3. class 3. 줄 삭제 + 게임오버 + 재시작
4. class 4. 점수/레벨 + 하드드롭 + 폴리싱

---

## class 1. 보드 만들기 + 블록 이동

### 목표

- 테트리스 보드와 기본 게임 루프를 만든다.
- 블록을 좌우 이동/낙하시키고 바닥 충돌을 처리한다.
- 격자 좌표 기반으로 블록을 렌더링한다.

### 핵심 변수/함수

- `COLS`, `ROWS`, `CELL`: 보드 가로/세로 칸 수와 셀 크기
- `piece`: 현재 블록 정보(`shape`, `x`, `y`, `color`)
- `can_move()`: 이동 가능한 위치인지 검사
- `draw_board()`: 보드와 현재 블록 렌더링

### 단계별 구현

#### 단계 1) 기본 창/보드 상수 만들기

##### 세부목표
  - 초기화와 보드 크기 상수를 연결해 실행 가능한 기본 화면을 만든다.
  - 좌표계를 픽셀보다 격자 중심으로 다루는 기준을 세운다.

```python
import pygame
import sys

pygame.init()

COLS, ROWS = 10, 20
CELL = 28
WIDTH, HEIGHT = COLS * CELL, ROWS * CELL

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Tetris - Class 1")
clock = pygame.time.Clock()
```

##### 선언한 변수/함수의 목적
  - `COLS`: 보드 가로 칸 수
  - `ROWS`: 보드 세로 칸 수
  - `CELL`: 한 칸 픽셀 크기
  - `screen`: 화면 출력 대상 객체
  - `clock`: 프레임 속도 제어 객체

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `COLS, ROWS = 10, 20`: 테트리스 표준에 가까운 보드 비율을 고정해 이동/충돌 기준을 일치시킨다.
  - `CELL = 28`: 셀 크기를 상수로 분리해 렌더링과 좌표 계산에서 같은 배율을 재사용한다.
  - `WIDTH, HEIGHT = COLS * CELL, ROWS * CELL`: 격자 칸 수를 픽셀 크기로 변환해 창 크기와 보드 범위를 정확히 맞춘다.
  - `screen = pygame.display.set_mode((WIDTH, HEIGHT))`: 게임 창 크기를 확정해 그리기 좌표 기준을 만든다.

#### 단계 2) 블록 데이터와 이동 가능 검사

##### 세부목표
  - 현재 블록 상태를 딕셔너리로 관리해 업데이트 흐름을 단순화한다.
  - 벽/바닥 경계 검사 함수를 분리해 입력 처리와 충돌 검사를 분리한다.

```python
I_SHAPE = [(0, 1), (1, 1), (2, 1), (3, 1)]
piece = {"shape": I_SHAPE, "x": 3, "y": 0, "color": (52, 211, 153)}


def can_move(piece, dx, dy):
    for cx, cy in piece["shape"]:
        nx = piece["x"] + cx + dx
        ny = piece["y"] + cy + dy
        if nx < 0 or nx >= COLS or ny >= ROWS:
            return False
    return True
```

##### 선언한 변수/함수의 목적
  - `I_SHAPE`: 일자 블록 상대 좌표 목록
  - `piece`: 현재 조작 중인 블록 상태
  - `can_move()`: 이동 가능한지 판정하는 함수
  - `nx`, `ny`: 이동 후 검사 좌표

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `piece = {"shape": I_SHAPE, "x": 3, "y": 0, "color": ...}`: 블록의 모양/위치/색을 한 구조로 묶어 업데이트와 렌더링을 같은 데이터로 처리한다.
  - `for cx, cy in piece["shape"]:`: 블록을 구성하는 4칸을 모두 순회해 일부 칸만 통과하는 충돌 누락을 막는다.
  - `nx = piece["x"] + cx + dx`: 상대 좌표와 이동량을 합쳐 실제 검사 x좌표를 계산한다.
  - `if nx < 0 or nx >= COLS or ny >= ROWS:`: 좌우 벽과 바닥 경계를 한 번에 차단해 보드 밖 이동을 막는다.

#### 단계 3) 입력 + 자동 낙하 처리

##### 세부목표
  - 좌우/아래 입력을 블록 상태 변화로 연결한다.
  - 일정 시간마다 블록이 한 칸씩 내려오는 기본 낙하 규칙을 만든다.

```python
DROP_EVENT = pygame.USEREVENT + 1
pygame.time.set_timer(DROP_EVENT, 450)

for event in pygame.event.get():
    if event.type == pygame.KEYDOWN:
        if event.key == pygame.K_LEFT and can_move(piece, -1, 0):
            piece["x"] -= 1
        elif event.key == pygame.K_RIGHT and can_move(piece, 1, 0):
            piece["x"] += 1
        elif event.key == pygame.K_DOWN and can_move(piece, 0, 1):
            piece["y"] += 1
    elif event.type == DROP_EVENT:
        if can_move(piece, 0, 1):
            piece["y"] += 1
```

##### 선언한 변수/함수의 목적
  - `DROP_EVENT`: 자동 낙하 이벤트 ID
  - `piece["x"]`: 현재 블록의 가로 위치
  - `piece["y"]`: 현재 블록의 세로 위치

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `pygame.time.set_timer(DROP_EVENT, 450)`: 프레임과 분리된 시간 기준 낙하 주기를 고정해 게임 템포를 안정화한다.
  - `if event.key == pygame.K_LEFT and can_move(piece, -1, 0):`: 이동 전에 충돌 검사를 먼저 통과한 경우만 위치를 갱신한다.
  - `piece["x"] -= 1`: 왼쪽 입력을 실제 좌표 변화로 반영해 다음 렌더링 프레임에서 블록 위치가 즉시 바뀌게 한다.
  - `if can_move(piece, 0, 1): piece["y"] += 1`: 자동 낙하 시에도 동일한 경계 검사 규칙을 재사용해 바닥 관통을 막는다.

#### 단계 4) 보드/블록 렌더링

##### 세부목표
  - 격자와 블록을 같은 좌표계로 그려 화면과 내부 상태를 일치시킨다.
  - 렌더링 코드를 함수로 분리해 이후 고정 블록/UI 추가를 쉽게 만든다.

```python
def draw_board(surface, piece):
    surface.fill((15, 23, 42))
    for y in range(ROWS):
        for x in range(COLS):
            pygame.draw.rect(surface, (30, 41, 59), (x * CELL, y * CELL, CELL, CELL), 1)

    for cx, cy in piece["shape"]:
        px = (piece["x"] + cx) * CELL
        py = (piece["y"] + cy) * CELL
        pygame.draw.rect(surface, piece["color"], (px, py, CELL, CELL), border_radius=4)
```

##### 선언한 변수/함수의 목적
  - `draw_board()`: 보드와 현재 블록을 그리는 함수
  - `px`, `py`: 블록 셀의 픽셀 좌표

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `for y in range(ROWS): for x in range(COLS):`: 모든 칸을 순회해 격자 라인을 빠짐없이 렌더링한다.
  - `pygame.draw.rect(..., 1)`: 테두리 두께를 1로 지정해 칸 경계를 보이게 하면서 배경 색을 유지한다.
  - `px = (piece["x"] + cx) * CELL`: 격자 좌표를 픽셀 좌표로 변환해 블록 위치를 화면에 정확히 매핑한다.
  - `pygame.draw.rect(surface, piece["color"], (px, py, CELL, CELL), border_radius=4)`: 현재 블록의 색과 위치를 적용해 조작 결과를 즉시 시각화한다.

#### class 1 최종 코드

```python
import pygame
import sys

pygame.init()

COLS, ROWS = 10, 20
CELL = 28
WIDTH, HEIGHT = COLS * CELL, ROWS * CELL

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Tetris - Class 1")
clock = pygame.time.Clock()

I_SHAPE = [(0, 1), (1, 1), (2, 1), (3, 1)]
piece = {"shape": I_SHAPE, "x": 3, "y": 0, "color": (52, 211, 153)}

DROP_EVENT = pygame.USEREVENT + 1
pygame.time.set_timer(DROP_EVENT, 450)


def can_move(piece, dx, dy):
    for cx, cy in piece["shape"]:
        nx = piece["x"] + cx + dx
        ny = piece["y"] + cy + dy
        if nx < 0 or nx >= COLS or ny >= ROWS:
            return False
    return True


def draw_board(surface, piece):
    surface.fill((15, 23, 42))
    for y in range(ROWS):
        for x in range(COLS):
            pygame.draw.rect(surface, (30, 41, 59), (x * CELL, y * CELL, CELL, CELL), 1)

    for cx, cy in piece["shape"]:
        px = (piece["x"] + cx) * CELL
        py = (piece["y"] + cy) * CELL
        pygame.draw.rect(surface, piece["color"], (px, py, CELL, CELL), border_radius=4)


while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_LEFT and can_move(piece, -1, 0):
                piece["x"] -= 1
            elif event.key == pygame.K_RIGHT and can_move(piece, 1, 0):
                piece["x"] += 1
            elif event.key == pygame.K_DOWN and can_move(piece, 0, 1):
                piece["y"] += 1

        elif event.type == DROP_EVENT:
            if can_move(piece, 0, 1):
                piece["y"] += 1

    draw_board(screen, piece)
    pygame.display.flip()
    clock.tick(60)
```

---

## class 2. 회전 + 고정 + 다음 블록

### 목표

- 블록 회전 규칙을 추가한다.
- 블록이 바닥/기존 블록에 닿으면 보드에 고정한다.
- 다음 블록을 생성해 게임 흐름을 이어간다.

### 핵심 변수/함수

- `board`: 고정된 블록 색상 정보를 담는 2차원 리스트
- `rotate_shape()`: 블록 상대 좌표를 회전한 새 모양 반환
- `lock_piece()`: 현재 블록을 보드에 고정
- `spawn_piece()`: 새 블록 생성

### 단계별 구현

#### 단계 1) 보드 배열과 블록 세트 추가

##### 세부목표
  - 고정된 블록을 저장할 보드 자료구조를 만든다.
  - 여러 종류 블록을 랜덤 생성할 준비를 한다.

```python
import random

SHAPES = [
    [(0, 1), (1, 1), (2, 1), (3, 1)],
    [(0, 0), (0, 1), (1, 1), (2, 1)],
    [(2, 0), (0, 1), (1, 1), (2, 1)],
    [(1, 0), (2, 0), (1, 1), (2, 1)],
]
board = [[None for _ in range(COLS)] for _ in range(ROWS)]
```

##### 선언한 변수/함수의 목적
  - `SHAPES`: 블록 모양 상대 좌표 목록
  - `board`: 고정 블록 상태 배열

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `SHAPES = [...]`: 블록 모양을 좌표 목록으로 통일해 회전/충돌/렌더링 로직이 같은 형식을 쓰게 한다.
  - `board = [[None ...] for _ in range(ROWS)]`: 빈 칸을 `None`으로 초기화해 충돌 검사와 줄 삭제 조건을 단순화한다.
  - `for _ in range(COLS)`: 한 행의 칸 개수를 고정해 x좌표 접근 범위를 보드 크기와 일치시킨다.
  - `for _ in range(ROWS)`: 행 수를 고정해 y축 이동과 바닥 충돌 기준을 명확히 만든다.

#### 단계 2) 회전 함수와 회전 충돌 검사

##### 세부목표
  - 블록 좌표를 회전한 결과를 계산한다.
  - 회전 후 벽/바닥/기존 블록 충돌 여부를 검사해 불가능한 회전을 막는다.

```python
def rotate_shape(shape):
    return [(-y, x) for x, y in shape]


def can_place(shape, px, py):
    for cx, cy in shape:
        nx, ny = px + cx, py + cy
        if nx < 0 or nx >= COLS or ny >= ROWS:
            return False
        if ny >= 0 and board[ny][nx] is not None:
            return False
    return True
```

##### 선언한 변수/함수의 목적
  - `rotate_shape()`: 회전된 좌표 목록 반환 함수
  - `can_place()`: 특정 모양/위치를 배치 가능한지 검사
  - `nx`, `ny`: 회전 후 검사 좌표

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `return [(-y, x) for x, y in shape]`: 좌표 변환으로 90도 회전을 계산해 별도 이미지 없이 회전 규칙을 구현한다.
  - `if nx < 0 or nx >= COLS or ny >= ROWS:`: 회전 결과가 벽/바닥을 넘는 경우를 즉시 차단한다.
  - `if ny >= 0 and board[ny][nx] is not None:`: 이미 고정된 블록이 있는 칸으로의 회전을 금지한다.
  - `can_place(shape, px, py)`: 이동/회전/생성 검증을 하나의 함수로 통합해 규칙 불일치를 줄인다.

#### 단계 3) 고정 + 새 블록 생성

##### 세부목표
  - 낙하 불가능할 때 현재 블록을 보드에 고정한다.
  - 새 블록을 생성해 플레이를 지속한다.

```python
def spawn_piece():
    shape = random.choice(SHAPES)
    return {"shape": shape, "x": 3, "y": 0, "color": (96, 165, 250)}


def lock_piece(piece):
    for cx, cy in piece["shape"]:
        bx, by = piece["x"] + cx, piece["y"] + cy
        if by >= 0:
            board[by][bx] = piece["color"]
```

##### 선언한 변수/함수의 목적
  - `spawn_piece()`: 새 블록 생성 함수
  - `lock_piece()`: 현재 블록 고정 함수
  - `bx`, `by`: 보드 기록 좌표

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `shape = random.choice(SHAPES)`: 블록 종류를 랜덤 선택해 매 판 입력 패턴을 다양화한다.
  - `return {"shape": shape, "x": 3, "y": 0, ...}`: 생성 위치를 상단 중앙 근처로 고정해 시작 직후 충돌 가능성을 줄인다.
  - `for cx, cy in piece["shape"]:`: 블록의 4칸을 모두 보드에 기록해 부분 고정 누락을 막는다.
  - `board[by][bx] = piece["color"]`: 고정된 칸에 색을 저장해 이후 충돌 검사와 렌더링에서 동일 데이터를 공유한다.

#### 단계 4) 회전/낙하 이벤트 연결

##### 세부목표
  - `UP` 키 회전을 입력 루프에 연결한다.
  - 낙하 불가능 시 고정 후 새 블록 생성 흐름을 완성한다.

```python
if event.type == pygame.KEYDOWN and event.key == pygame.K_UP:
    rotated = rotate_shape(piece["shape"])
    if can_place(rotated, piece["x"], piece["y"]):
        piece["shape"] = rotated

if event.type == DROP_EVENT:
    if can_place(piece["shape"], piece["x"], piece["y"] + 1):
        piece["y"] += 1
    else:
        lock_piece(piece)
        piece = spawn_piece()
```

##### 선언한 변수/함수의 목적
  - `rotated`: 회전 결과 좌표 목록
  - `piece["shape"]`: 현재 블록 모양 데이터

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `rotated = rotate_shape(piece["shape"])`: 원본 모양을 즉시 덮어쓰지 않고 임시 회전 결과를 먼저 계산해 검증 단계를 확보한다.
  - `if can_place(rotated, piece["x"], piece["y"]):`: 회전 후 충돌이 없을 때만 실제 모양을 교체한다.
  - `lock_piece(piece)`: 더 내려갈 수 없는 시점에 현재 블록을 보드 상태로 확정해 다음 블록과의 충돌 기준을 만든다.
  - `piece = spawn_piece()`: 고정 직후 새 블록을 생성해 게임 루프가 끊기지 않게 이어준다.

#### class 2 최종 코드

```python
# class 1 코드에 아래 로직을 합쳐 실행
# 핵심: board, rotate_shape, can_place, lock_piece, spawn_piece 추가
# 실제 수업에서는 class 1 최종 코드 위에 함수들을 붙이고
# 입력 루프에 UP 회전/DROP 고정 분기를 연결해 완성한다.
```

---

## class 3. 줄 삭제 + 게임오버 + 재시작

### 목표

- 가득 찬 줄을 지우고 위 줄을 내린다.
- 새 블록 생성 불가능 시 게임오버 처리한다.
- `R` 키 재시작을 구현한다.

### 핵심 변수/함수

- `clear_lines()`: 꽉 찬 행 삭제 후 빈 행 추가
- `game_over`: 게임 종료 상태
- `reset_game()`: 보드/블록/상태 초기화

### 단계별 구현

#### 단계 1) 줄 삭제 함수

##### 세부목표
  - 보드에서 꽉 찬 줄을 찾아 제거한다.
  - 제거된 줄 수를 반환해 점수/레벨 계산에 연결 가능한 구조를 만든다.

```python
def clear_lines():
    global board
    kept = [row for row in board if any(cell is None for cell in row)]
    cleared = ROWS - len(kept)
    board = [[None for _ in range(COLS)] for _ in range(cleared)] + kept
    return cleared
```

##### 선언한 변수/함수의 목적
  - `clear_lines()`: 줄 삭제 처리 함수
  - `kept`: 삭제 후 남길 행 목록
  - `cleared`: 삭제된 줄 수

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `kept = [row for row in board if any(cell is None for cell in row)]`: 빈 칸이 하나라도 있는 행만 남겨 꽉 찬 줄을 제거한다.
  - `cleared = ROWS - len(kept)`: 원래 행 수와 남은 행 수 차이로 삭제 줄 수를 계산한다.
  - `[[None ...] for _ in range(cleared)] + kept`: 삭제된 줄 수만큼 위에 빈 행을 보충해 보드 높이를 유지한다.
  - `return cleared`: 삭제 결과를 반환해 이후 점수 증가 로직에서 재사용할 수 있게 한다.

#### 단계 2) 게임오버 판정

##### 세부목표
  - 새 블록 생성 위치가 이미 막힌 경우 즉시 게임오버로 전환한다.
  - 게임오버 상태에서는 블록 업데이트를 중단한다.

```python
game_over = False

piece = spawn_piece()
if not can_place(piece["shape"], piece["x"], piece["y"]):
    game_over = True
```

##### 선언한 변수/함수의 목적
  - `game_over`: 종료 상태 플래그
  - `piece`: 현재 블록 상태

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `game_over = False`: 기본 상태를 진행 가능으로 두고 충돌 조건에서만 종료로 전환한다.
  - `piece = spawn_piece()`: 고정 직후 다음 블록을 생성해 배치 가능 여부를 즉시 검사한다.
  - `if not can_place(...):`: 시작 위치조차 비어있지 않으면 더 이상 진행 불가로 판단한다.
  - `game_over = True`: 종료 상태를 켜 입력/낙하 업데이트를 차단하고 안내 메시지 렌더링으로 분기한다.

#### 단계 3) 재시작 함수

##### 세부목표
  - 보드와 상태값을 한 번에 초기화한다.
  - `R` 키 입력으로 재시작 루틴을 호출한다.

```python
def reset_game():
    global board, piece, game_over
    board = [[None for _ in range(COLS)] for _ in range(ROWS)]
    piece = spawn_piece()
    game_over = False

if event.type == pygame.KEYDOWN and event.key == pygame.K_r:
    reset_game()
```

##### 선언한 변수/함수의 목적
  - `reset_game()`: 게임 초기화 함수
  - `board`: 고정 블록 상태 배열
  - `game_over`: 종료 상태 플래그

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `board = [[None ...] for _ in range(ROWS)]`: 보드를 완전히 비워 이전 판의 고정 블록 잔존을 없앤다.
  - `piece = spawn_piece()`: 재시작 직후 조작 가능한 블록을 즉시 만들어 게임 흐름을 이어간다.
  - `game_over = False`: 종료 상태를 해제해 낙하/입력 업데이트 분기가 다시 동작하도록 복구한다.
  - `if event.type == pygame.KEYDOWN and event.key == pygame.K_r:`: 재시작 입력을 명확히 분리해 다른 키 입력과 충돌하지 않게 한다.

#### 단계 4) 게임오버 메시지 렌더링

##### 세부목표
  - 종료 상태를 화면으로 명확히 전달한다.
  - 사용자에게 재시작 키를 안내한다.

```python
if game_over:
    text = font.render("Game Over", True, (248, 250, 252))
    hint = small_font.render("Press R to Restart", True, (148, 163, 184))
    screen.blit(text, (20, 240))
    screen.blit(hint, (20, 280))
```

##### 선언한 변수/함수의 목적
  - `text`: 게임오버 제목 텍스트 Surface
  - `hint`: 재시작 안내 텍스트 Surface

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if game_over:`: 종료 상태일 때만 오버레이 텍스트를 그려 플레이 중 UI와 구분한다.
  - `font.render("Game Over", ...)`: 문자열을 렌더링 가능한 Surface로 변환한다.
  - `screen.blit(text, (20, 240))`: 제목 문구를 고정 위치에 배치해 사용자가 즉시 상태를 인지하게 한다.
  - `screen.blit(hint, (20, 280))`: 재시작 힌트를 함께 출력해 다음 행동을 명확히 안내한다.

#### class 3 최종 코드

```python
# class 2 코드에 clear_lines(), game_over, reset_game, 오버레이 렌더링을 통합해 실행
# 핵심 동작: lock_piece() 이후 clear_lines() 호출, spawn 실패 시 game_over=True
```

---

## class 4. 점수/레벨 + 하드드롭 + 폴리싱

### 목표

- 줄 삭제 수에 따라 점수와 레벨을 올린다.
- 스페이스 하드드롭을 구현한다.
- 최종 실행 가능한 테트리스 코드를 완성한다.

### 핵심 변수/함수

- `score`, `level`, `lines_total`: 진행 상태 점수 데이터
- `drop_delay`: 자동 낙하 간격(ms)
- `hard_drop()`: 바닥 또는 충돌 직전까지 즉시 낙하

### 단계별 구현

#### 단계 1) 점수/레벨 상태 추가

##### 세부목표
  - 줄 삭제 결과를 점수와 레벨 변화로 연결한다.
  - 레벨에 따라 자동 낙하 속도가 빨라지게 만든다.

```python
score = 0
level = 1
lines_total = 0

def apply_scoring(cleared):
    global score, level, lines_total
    table = {1: 100, 2: 300, 3: 500, 4: 800}
    score += table.get(cleared, 0) * level
    lines_total += cleared
    level = 1 + lines_total // 10
```

##### 선언한 변수/함수의 목적
  - `apply_scoring()`: 줄 삭제 점수 계산 함수
  - `score`: 누적 점수
  - `level`: 현재 난이도 레벨
  - `lines_total`: 누적 삭제 줄 수

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `table = {1: 100, 2: 300, 3: 500, 4: 800}`: 동시 삭제 줄 수별 보상 점수를 명확한 규칙으로 고정한다.
  - `score += table.get(cleared, 0) * level`: 삭제 줄 수와 레벨을 곱해 난이도 상승에 맞는 점수 보상을 준다.
  - `lines_total += cleared`: 누적 삭제 줄 수를 유지해 레벨 상승 조건을 안정적으로 계산한다.
  - `level = 1 + lines_total // 10`: 10줄 단위로 레벨을 올려 플레이 시간이 길수록 템포가 빨라지게 만든다.

#### 단계 2) 하드드롭 구현

##### 세부목표
  - 스페이스 키로 블록을 즉시 바닥까지 내린다.
  - 하드드롭 후 고정/줄삭제/다음 블록 생성까지 한 번에 처리한다.

```python
def hard_drop(piece):
    while can_place(piece["shape"], piece["x"], piece["y"] + 1):
        piece["y"] += 1

if event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE:
    hard_drop(piece)
    lock_piece(piece)
    cleared = clear_lines()
    apply_scoring(cleared)
    piece = spawn_piece()
```

##### 선언한 변수/함수의 목적
  - `hard_drop()`: 즉시 낙하 함수
  - `cleared`: 이번 고정에서 삭제된 줄 수

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `while can_place(..., piece["y"] + 1):`: 더 이상 못 내려갈 때까지 반복해 하드드롭을 정확히 구현한다.
  - `piece["y"] += 1`: 한 칸씩 누적 이동해 충돌 직전 위치를 안전하게 찾는다.
  - `cleared = clear_lines()`: 하드드롭으로 고정된 결과를 줄 삭제 로직에 즉시 반영한다.
  - `apply_scoring(cleared)`: 삭제된 줄 수를 점수/레벨 변화로 연결해 플레이 보상을 즉시 갱신한다.

#### 단계 3) 레벨 기반 낙하 속도 조절

##### 세부목표
  - 레벨이 오를수록 자동 낙하 간격을 줄인다.
  - 타이머를 동적으로 갱신해 체감 난이도를 상승시킨다.

```python
def update_drop_timer():
    drop_delay = max(90, 460 - (level - 1) * 30)
    pygame.time.set_timer(DROP_EVENT, drop_delay)
```

##### 선언한 변수/함수의 목적
  - `update_drop_timer()`: 레벨별 낙하 속도 갱신 함수
  - `drop_delay`: 현재 자동 낙하 간격

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `drop_delay = max(90, 460 - (level - 1) * 30)`: 레벨 상승에 따라 간격을 줄이되 최소값을 둬 과속을 방지한다.
  - `pygame.time.set_timer(DROP_EVENT, drop_delay)`: 타이머 간격을 즉시 갱신해 레벨 변화가 실시간 속도에 반영되게 한다.
  - `460 - (level - 1) * 30`: 초반은 천천히, 후반은 빠르게 증가하는 난이도 곡선을 만든다.
  - `max(90, ...)`: 하한을 고정해 고레벨에서도 반응 불가능한 속도로 무너지는 것을 막는다.

#### 단계 4) HUD 렌더링

##### 세부목표
  - 점수/레벨/줄 수를 화면에 출력한다.
  - 상태값과 시각 정보가 항상 동기화되게 한다.

```python
hud = small_font.render(f"Score {score}  Level {level}  Lines {lines_total}", True, (226, 232, 240))
screen.blit(hud, (8, 8))
```

##### 선언한 변수/함수의 목적
  - `hud`: 진행 정보 텍스트 Surface

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `f"Score {score}  Level {level}  Lines {lines_total}"`: 핵심 상태값을 한 줄로 합쳐 플레이 상황을 즉시 읽을 수 있게 만든다.
  - `small_font.render(..., True, ...)`: 텍스트 안티앨리어싱을 적용해 작은 HUD 글자가 깨지지 않게 렌더링한다.
  - `screen.blit(hud, (8, 8))`: 좌상단 고정 위치에 HUD를 배치해 보드 시야를 크게 가리지 않게 한다.
  - `score/level/lines_total`을 매 프레임 렌더링해 상태 변경이 화면에 즉시 반영되도록 유지한다.

#### class 4 최종 코드

```python
import pygame
import random
import sys

pygame.init()

COLS, ROWS = 10, 20
CELL = 28
WIDTH, HEIGHT = COLS * CELL, ROWS * CELL

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Tetris - Final")
clock = pygame.time.Clock()
small_font = pygame.font.SysFont("arial", 18)
font = pygame.font.SysFont("arial", 32, bold=True)

DROP_EVENT = pygame.USEREVENT + 1

SHAPES = [
    [(0, 1), (1, 1), (2, 1), (3, 1)],
    [(0, 0), (0, 1), (1, 1), (2, 1)],
    [(2, 0), (0, 1), (1, 1), (2, 1)],
    [(1, 0), (2, 0), (1, 1), (2, 1)],
    [(1, 0), (2, 0), (0, 1), (1, 1)],
    [(0, 0), (1, 0), (1, 1), (2, 1)],
    [(1, 0), (0, 1), (1, 1), (2, 1)],
]
COLORS = [(52, 211, 153), (96, 165, 250), (167, 139, 250), (248, 113, 113), (250, 204, 21)]

board = [[None for _ in range(COLS)] for _ in range(ROWS)]
score = 0
level = 1
lines_total = 0
game_over = False


def spawn_piece():
    return {
        "shape": random.choice(SHAPES),
        "x": 3,
        "y": 0,
        "color": random.choice(COLORS),
    }


def rotate_shape(shape):
    return [(-y, x) for x, y in shape]


def can_place(shape, px, py):
    for cx, cy in shape:
        nx, ny = px + cx, py + cy
        if nx < 0 or nx >= COLS or ny >= ROWS:
            return False
        if ny >= 0 and board[ny][nx] is not None:
            return False
    return True


def lock_piece(piece):
    for cx, cy in piece["shape"]:
        bx, by = piece["x"] + cx, piece["y"] + cy
        if by >= 0:
            board[by][bx] = piece["color"]


def clear_lines():
    global board
    kept = [row for row in board if any(cell is None for cell in row)]
    cleared = ROWS - len(kept)
    board = [[None for _ in range(COLS)] for _ in range(cleared)] + kept
    return cleared


def apply_scoring(cleared):
    global score, level, lines_total
    table = {1: 100, 2: 300, 3: 500, 4: 800}
    score += table.get(cleared, 0) * level
    lines_total += cleared
    level = 1 + lines_total // 10


def update_drop_timer():
    delay = max(90, 460 - (level - 1) * 30)
    pygame.time.set_timer(DROP_EVENT, delay)


def hard_drop(piece):
    while can_place(piece["shape"], piece["x"], piece["y"] + 1):
        piece["y"] += 1


def draw(piece):
    screen.fill((15, 23, 42))
    for y in range(ROWS):
        for x in range(COLS):
            pygame.draw.rect(screen, (30, 41, 59), (x * CELL, y * CELL, CELL, CELL), 1)
            if board[y][x] is not None:
                pygame.draw.rect(screen, board[y][x], (x * CELL + 1, y * CELL + 1, CELL - 2, CELL - 2), border_radius=4)

    for cx, cy in piece["shape"]:
        px = (piece["x"] + cx) * CELL
        py = (piece["y"] + cy) * CELL
        pygame.draw.rect(screen, piece["color"], (px + 1, py + 1, CELL - 2, CELL - 2), border_radius=4)

    hud = small_font.render(f"Score {score}  Level {level}  Lines {lines_total}", True, (226, 232, 240))
    screen.blit(hud, (8, 8))

    if game_over:
        t1 = font.render("Game Over", True, (248, 250, 252))
        t2 = small_font.render("R: Restart   Q: Quit", True, (148, 163, 184))
        screen.blit(t1, (20, 240))
        screen.blit(t2, (20, 280))


piece = spawn_piece()
update_drop_timer()

while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_q:
                pygame.quit()
                sys.exit()
            if event.key == pygame.K_r and game_over:
                board = [[None for _ in range(COLS)] for _ in range(ROWS)]
                score = 0
                level = 1
                lines_total = 0
                game_over = False
                piece = spawn_piece()
                update_drop_timer()
            if game_over:
                continue

            if event.key == pygame.K_LEFT and can_place(piece["shape"], piece["x"] - 1, piece["y"]):
                piece["x"] -= 1
            elif event.key == pygame.K_RIGHT and can_place(piece["shape"], piece["x"] + 1, piece["y"]):
                piece["x"] += 1
            elif event.key == pygame.K_DOWN and can_place(piece["shape"], piece["x"], piece["y"] + 1):
                piece["y"] += 1
            elif event.key == pygame.K_UP:
                rotated = rotate_shape(piece["shape"])
                if can_place(rotated, piece["x"], piece["y"]):
                    piece["shape"] = rotated
            elif event.key == pygame.K_SPACE:
                hard_drop(piece)
                lock_piece(piece)
                cleared = clear_lines()
                apply_scoring(cleared)
                update_drop_timer()
                piece = spawn_piece()
                if not can_place(piece["shape"], piece["x"], piece["y"]):
                    game_over = True

        if event.type == DROP_EVENT and not game_over:
            if can_place(piece["shape"], piece["x"], piece["y"] + 1):
                piece["y"] += 1
            else:
                lock_piece(piece)
                cleared = clear_lines()
                apply_scoring(cleared)
                update_drop_timer()
                piece = spawn_piece()
                if not can_place(piece["shape"], piece["x"], piece["y"]):
                    game_over = True

    draw(piece)
    pygame.display.flip()
    clock.tick(60)
```
