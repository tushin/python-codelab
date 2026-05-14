## 개요

- 대상: `pygame`으로 첫 러닝 액션 게임을 만들어보는 초급 학습자
- 방식: class별로 "실행 코드 + 기능 1개 확장" 방식
- 최종 산출물: 크롬 공룡게임 스타일의 점프 액션 게임 1개
- 실행 환경: Python 3.10+ / `pip install pygame`

## 목차

1. class 1. 달리기 화면 + 점프
2. class 2. 장애물 생성 + 이동
3. class 3. 충돌 판정 + 게임오버 + 재시작
4. class 4. 점수 + 속도 증가 + 시작/일시정지 폴리싱

---

## class 1. 달리기 화면 + 점프

### 목표

- 게임 창, 바닥, 공룡(플레이어)을 그린다.
- 중력과 점프를 구현한다.
- 공룡이 바닥 아래로 내려가지 않도록 고정한다.

### 핵심 변수/함수

- `dino_y`: 공룡의 현재 Y좌표(위치). 매 프레임 `dino_rect = Rect(DINO_X, int(dino_y), ...)`에 넣어 화면에 그린다.
- `dino_vy`: 공룡의 세로 속도. 점프 시 `JUMP_POWER`를 넣고, 매 프레임 `GRAVITY`를 더해 낙하를 계산한다.
- `on_ground`: `dino_y >= GROUND_Y - DINO_H` 계산 결과(Boolean). `SPACE` 입력 시 점프 가능 여부를 판정한다.
- `GRAVITY`, `JUMP_POWER`: 각각 낙하 가속도와 점프 시작 속도. `dino_vy` 계산식에 직접 사용된다.

### 단계별 구현

#### 단계 1) 기본 창/루프 만들기

##### 세부목표
  - 초기화와 화면 생성 코드를 연결해 실행 가능한 기본 게임 화면을 만든다.
  - 현재 단계의 상태값이 다음 단계 동작으로 어떻게 이어지는지 이해한다.

```python
import pygame
import sys

pygame.init()

WIDTH, HEIGHT = 1000, 320
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Dino Run - Class 1")
clock = pygame.time.Clock()
```

##### 선언한 변수/함수의 목적
  - `WIDTH`: 게임 화면 가로 크기
  - `HEIGHT`: 게임 화면 세로 크기
  - `screen`: 화면 출력 대상 객체
  - `clock`: 프레임 속도 제어 객체

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `screen = pygame.display.set_mode((WIDTH, HEIGHT))`: 게임 창 크기를 확정해 좌표 기준을 만든다.
  - `pygame.init()`: 키보드 입력, 화면 출력, 사운드 같은 pygame 하위 모듈을 한 번에 초기화해 하드웨어와 통신할 준비를 마친다.
  - `WIDTH, HEIGHT = 1000, 320`: 창 크기를 먼저 고정해 좌표계 기준과 UI 배치 범위를 명확히 만든다.
  - `pygame.display.set_caption("Dino Run - Class 1")`: 창 제목을 지정해 디버깅/수업 중 여러 창을 띄워도 현재 실행 게임을 쉽게 구분하게 한다.

#### 단계 2) 공룡과 바닥 상수 정의

##### 세부목표
  - 공룡과 바닥 상수 정의 단계의 코드를 게임 루프에 연결해 실제 동작으로 확인한다.
  - 현재 단계의 상태값이 다음 단계 동작으로 어떻게 이어지는지 이해한다.

```python
GROUND_Y = 250
DINO_W, DINO_H = 44, 52
DINO_X = 90

dino_y = GROUND_Y - DINO_H
dino_vy = 0.0

GRAVITY = 0.72
JUMP_POWER = -13.8
```

##### 선언한 변수/함수의 목적
  - `GROUND_Y`: 바닥 Y 좌표
  - `DINO_W`: 공룡 가로 크기
  - `DINO_H`: 공룡 세로 크기
  - `DINO_X`: 공룡 시작 X 좌표
  - `dino_y`: 객체 위치 좌표를 저장한다. 매 프레임 Rect 생성과 그리기에 사용되어 화면 위치가 정해진다.
  - `dino_vy`: 공룡의 세로 속도로, 점프와 낙하 물리 계산의 직접 입력값이 된다.
  - `GRAVITY`: 중력 가속 값
  - `JUMP_POWER`: 점프 시작 속도

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `GROUND_Y = 250`: 바닥 기준 y좌표를 상수화해 착지 판정과 장애물 배치를 동일 기준으로 맞춘다.
  - `DINO_W, DINO_H = 44, 52`: 공룡 충돌 박스 크기를 고정해 렌더링 크기와 충돌 판정 크기가 일치하게 한다.
  - `DINO_X = 90`: 공룡 시작 x좌표를 고정해 장애물 접근 시간과 점프 타이밍 난이도를 안정화한다.
  - `dino_y = GROUND_Y - DINO_H`: 공룡의 초기 y좌표를 바닥선에 맞춰 시작 직후 공중/관통 상태가 생기지 않게 한다.

#### 단계 3) 스페이스 점프 입력

##### 세부목표
  - 입력 이벤트를 상태값 변화와 연결해 사용자의 조작이 게임 동작으로 이어지게 만든다.
  - 학습 목표는 `if/elif` 분기로 입력 조건을 나누고 상태 업데이트 순서를 이해하는 것이다.

```python
if event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE:
    on_ground = dino_y >= GROUND_Y - DINO_H
    if on_ground:
        dino_vy = JUMP_POWER
```

##### 선언한 변수/함수의 목적
  - `on_ground`: 공룡이 착지 상태인지 저장해 점프 가능 여부를 제어한다.
  - `dino_vy`: 공룡의 세로 속도로, 점프와 낙하 물리 계산의 직접 입력값이 된다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE:`: 이벤트 타입을 비교해 입력/업데이트 처리를 분기한다.
  - `if on_ground:`: 지면에 닿아 있을 때만 점프 속도를 적용해 공중 연속 점프를 방지한다.
  - `on_ground = dino_y >= GROUND_Y - DINO_H`: 현재 착지 여부를 불리언으로 고정해 점프 입력 허용/차단을 명확히 제어한다.
  - `dino_vy = JUMP_POWER`: 점프 순간 시작 속도를 설정해 키 입력이 즉시 위로 이동하는 물리 반응으로 이어지게 한다.

#### 단계 4) 중력 + 바닥 고정

##### 세부목표
  - 중력 + 바닥 고정 단계의 코드를 게임 루프에 연결해 실제 동작으로 확인한다.
  - 현재 단계의 상태값이 다음 단계 동작으로 어떻게 이어지는지 이해한다.

```python
dino_vy += GRAVITY
dino_y += dino_vy

if dino_y > GROUND_Y - DINO_H:
    dino_y = GROUND_Y - DINO_H
    dino_vy = 0
```

##### 선언한 변수/함수의 목적
  - `dino_y`: 객체 위치 좌표를 저장한다. 매 프레임 Rect 생성과 그리기에 사용되어 화면 위치가 정해진다.
  - `dino_vy`: 공룡의 세로 속도로, 점프와 낙하 물리 계산의 직접 입력값이 된다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if dino_y > GROUND_Y - DINO_H:`: 바닥선 아래로 내려가려는 순간에만 위치를 보정해 관통을 방지한다.
  - `dino_vy += GRAVITY`: 매 프레임 낙하 가속도를 속도에 누적해, 점프 후 자연스럽게 내려오도록 만든다.
  - `dino_y += dino_vy`: 갱신된 속도를 위치에 반영해 공룡의 실제 화면 위치를 위/아래로 이동시킨다.
  - `dino_y = GROUND_Y - DINO_H`: 공룡의 초기 y좌표를 바닥선에 맞춰 시작 직후 공중/관통 상태가 생기지 않게 한다.

### class 1 최종 코드

```python
import pygame
import sys

pygame.init()

WIDTH, HEIGHT = 1000, 320
GROUND_Y = 250

BG = (247, 247, 247)
LINE = (70, 70, 70)
DINO_COLOR = (32, 32, 32)

DINO_W, DINO_H = 44, 52
DINO_X = 90
GRAVITY = 0.72
JUMP_POWER = -13.8

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Dino Run - Class 1")
clock = pygame.time.Clock()

dino_y = GROUND_Y - DINO_H
dino_vy = 0.0

while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        elif event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE:
            on_ground = dino_y >= GROUND_Y - DINO_H
            if on_ground:
                dino_vy = JUMP_POWER

    dino_vy += GRAVITY
    dino_y += dino_vy

    if dino_y > GROUND_Y - DINO_H:
        dino_y = GROUND_Y - DINO_H
        dino_vy = 0

    screen.fill(BG)
    pygame.draw.line(screen, LINE, (0, GROUND_Y), (WIDTH, GROUND_Y), 3)
    dino_rect = pygame.Rect(DINO_X, int(dino_y), DINO_W, DINO_H)
    pygame.draw.rect(screen, DINO_COLOR, dino_rect, border_radius=4)

    pygame.display.flip()
    clock.tick(60)
```

---

## class 2. 장애물 생성 + 이동

### 목표

- 일정 시간마다 선인장(장애물)을 생성한다.
- 장애물을 왼쪽으로 이동시키고 화면 밖 객체를 삭제한다.

### 핵심 변수/함수

- `obstacles`: `pygame.Rect` 장애물 목록. 이동 루프(`ob.x -= world_speed`), 충돌 루프(`dino_rect.colliderect(ob)`), 렌더링 루프(`draw.rect`)에서 공통으로 사용한다.
- `SPAWN_EVENT`: `set_timer`로 주기 발생하는 이벤트 타입. 이벤트 루프에서 이 값과 비교해 `spawn_obstacle()` 호출 여부를 결정한다.
- `world_speed`: 장애물이 왼쪽으로 이동하는 픽셀 속도. 값이 커질수록 난이도가 올라간다.
- `spawn_obstacle()`: 장애물 크기(`w`, `h`)를 정하고, 시작 위치 `WIDTH + 20`에서 `obstacles` 리스트에 추가한다.

### 단계별 구현

#### 단계 1) 장애물 리스트/속도 변수

##### 세부목표
  - 장애물 리스트/속도 변수 단계의 코드를 게임 루프에 연결해 실제 동작으로 확인한다.
  - 현재 단계의 상태값이 다음 단계 동작으로 어떻게 이어지는지 이해한다.

```python
obstacles = []
world_speed = 7
```

##### 선언한 변수/함수의 목적
  - `obstacles`: 게임 객체 목록을 저장한다. 루프에서 순회하며 이동·충돌 판정·렌더링에 함께 사용한다.
  - `world_speed`: 장애물 좌측 이동 속도로 값이 커질수록 체감 난이도가 올라간다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `obstacles = []`: 장애물 목록을 비워 시작해 이전 판 데이터가 새 게임에 섞이지 않게 한다.
  - `world_speed = 7`: 기본 이동 속도를 정해 장애물 이동 체감 난이도를 일정하게 시작한다.

#### 단계 2) 생성 타이머 등록

##### 세부목표
  - 타이머/시간 기반 로직으로 업데이트 주기를 제어해 프레임과 게임 규칙을 분리한다.
  - 학습 목표는 `set_timer`·`get_ticks`로 시간 기반 동작을 제어하는 방법을 익히는 것이다.

```python
SPAWN_EVENT = pygame.USEREVENT + 1
pygame.time.set_timer(SPAWN_EVENT, 1100)
```

##### 선언한 변수/함수의 목적
  - `SPAWN_EVENT`: 장애물 생성 이벤트 ID

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `pygame.time.set_timer(SPAWN_EVENT, 1100)`: 일정 주기 이벤트를 등록해 업데이트 박자를 고정한다.
  - `SPAWN_EVENT = pygame.USEREVENT + 1`: 사용자 이벤트 번호를 분리해 일반 입력 이벤트와 장애물 생성 이벤트를 안전하게 구분한다.

#### 단계 3) 장애물 생성 함수

##### 세부목표
  - 반복되는 규칙을 함수로 분리해 가독성과 유지보수성을 높인다.
  - 학습 목표는 `def`/매개변수/반환값 기준으로 코드를 읽고 기능 단위로 나누는 것이다.

```python
import random

def spawn_obstacle():
    h = random.choice([36, 44, 58])
    w = random.choice([18, 22, 26])
    rect = pygame.Rect(WIDTH + 20, GROUND_Y - h, w, h)
    obstacles.append(rect)
```

##### 선언한 변수/함수의 목적
  - `spawn_obstacle()`: 장애물을 생성해 목록에 추가하는 함수
  - `h`: 장애물 높이를 랜덤으로 뽑아 패턴 다양성을 만든다.
  - `w`: 장애물 너비를 랜덤으로 뽑아 점프 타이밍 난이도를 다양화한다.
  - `rect`: 충돌/렌더링용 사각형 객체

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `def spawn_obstacle():`: `spawn_obstacle()` 함수 정의를 시작해 관련 로직을 기능 단위로 분리한다.
  - `h = random.choice([36, 44, 58])`: 무작위 값을 생성해 좌표/패턴 다양성을 만든다.
  - `w = random.choice([18, 22, 26])`: 무작위 값을 생성해 좌표/패턴 다양성을 만든다.
  - `rect = pygame.Rect(WIDTH + 20, GROUND_Y - h, w, h)`: 사각형 좌표/크기를 `Rect`로 묶어 렌더링과 충돌 판정에서 같은 기준 데이터를 재사용하게 한다.

#### 단계 4) 이벤트에서 생성 + 매 프레임 이동

##### 세부목표
  - 입력 이벤트를 상태값 변화와 연결해 사용자의 조작이 게임 동작으로 이어지게 만든다.
  - 학습 목표는 `if/elif` 분기로 입력 조건을 나누고 상태 업데이트 순서를 이해하는 것이다.

```python
if event.type == SPAWN_EVENT:
    spawn_obstacle()
```

```python
for ob in obstacles:
    ob.x -= world_speed

obstacles = [ob for ob in obstacles if ob.right > -10]
```

##### 선언한 변수/함수의 목적
  - `obstacles`: 게임 객체 목록을 저장한다. 루프에서 순회하며 이동·충돌 판정·렌더링에 함께 사용한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if event.type == SPAWN_EVENT:`: 이벤트 타입을 비교해 입력/업데이트 처리를 분기한다.
  - `for ob in obstacles:`: 장애물 목록 전체를 순회해 이동·충돌·렌더링 규칙을 동일하게 적용한다.
  - `spawn_obstacle()`: 새 장애물을 목록에 추가해 다음 프레임부터 이동/충돌 판정 대상이 생기게 한다.
  - `ob.x -= world_speed`: 각 장애물의 x좌표를 줄여 화면 오른쪽에서 왼쪽으로 이동하는 효과를 만든다.

---

## class 3. 충돌 판정 + 게임오버 + 재시작

### 목표

- 공룡과 장애물이 겹치면 게임오버 상태로 전환한다.
- `R` 키로 즉시 재시작한다.

### 핵심 변수/함수

- `game_state`: 현재 게임 상태 문자열. 업데이트/입력 분기에서 `RUNNING`인지 확인해 로직 실행 범위를 제어한다.
- `dino_rect`: 현재 프레임 공룡 충돌 박스. 장애물 `ob`와 `colliderect`로 충돌 여부를 계산한다.
- `reset_game()`: `dino_y`, `dino_vy`, `obstacles`, `game_state`를 초기값으로 되돌려 즉시 재시작 가능하게 만든다.

### 단계별 구현

#### 단계 1) 상태 변수와 리셋 함수

##### 세부목표
  - 반복되는 규칙을 함수로 분리해 가독성과 유지보수성을 높인다.
  - 학습 목표는 `def`/매개변수/반환값 기준으로 코드를 읽고 기능 단위로 나누는 것이다.

```python
RUNNING = "running"
GAME_OVER = "game_over"
game_state = RUNNING

def reset_game():
    global dino_y, dino_vy, obstacles, game_state
    dino_y = GROUND_Y - DINO_H
    dino_vy = 0
    obstacles = []
    game_state = RUNNING
```

##### 선언한 변수/함수의 목적
  - `reset_game()`: 게임 상태를 초기값으로 되돌리는 함수
  - `RUNNING`: 게임 진행 상태 문자열
  - `GAME_OVER`: 게임오버 상태 문자열
  - `game_state`: ready/running/paused/game_over 중 현재 상태를 저장해 전체 루프 분기를 제어한다.
  - `dino_y`: 객체 위치 좌표를 저장한다. 매 프레임 Rect 생성과 그리기에 사용되어 화면 위치가 정해진다.
  - `dino_vy`: 공룡의 세로 속도로, 점프와 낙하 물리 계산의 직접 입력값이 된다.
  - `obstacles`: 게임 객체 목록을 저장한다. 루프에서 순회하며 이동·충돌 판정·렌더링에 함께 사용한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `def reset_game():`: `reset_game()` 함수 정의를 시작해 관련 로직을 기능 단위로 분리한다.
  - `RUNNING = "running"`: 진행 상태 문자열을 상수화해 오타 없이 상태 전환 분기를 재사용하게 한다.
  - `GAME_OVER = "game_over"`: 종료 상태 문자열을 상수화해 충돌 후 입력/업데이트 제한 분기를 일관되게 처리한다.
  - `game_state = RUNNING`: 초기 상태를 `RUNNING`으로 지정해 시작 직후 업데이트 루프가 바로 동작하게 한다.

#### 단계 2) 충돌 검사

##### 세부목표
  - 충돌/경계 판정을 구현해 실패·성공 상태 전환 조건을 정확히 만든다.
  - 현재 단계의 상태값이 다음 단계 동작으로 어떻게 이어지는지 이해한다.

```python
if game_state == RUNNING:
    dino_rect = pygame.Rect(DINO_X, int(dino_y), DINO_W, DINO_H)
    for ob in obstacles:
        if dino_rect.colliderect(ob):
            game_state = GAME_OVER
            break
```

##### 선언한 변수/함수의 목적
  - `dino_rect`: 충돌/렌더링용 사각형 객체
  - `game_state`: ready/running/paused/game_over 중 현재 상태를 저장해 전체 루프 분기를 제어한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if dino_rect.colliderect(ob):`: 사각형 충돌 여부를 검사해 상태 전환 조건으로 사용한다.
  - `if game_state == RUNNING:`: 실행 상태일 때만 이동/충돌/점수 갱신을 진행해 일시정지·종료 상태와 로직을 분리한다.
  - `for ob in obstacles:`: 장애물 목록 전체를 순회해 이동·충돌·렌더링 규칙을 동일하게 적용한다.
  - `dino_rect = pygame.Rect(DINO_X, int(dino_y), DINO_W, DINO_H)`: 현재 프레임의 공룡 충돌 박스를 만들어 장애물과의 `colliderect` 판정을 정확히 수행한다.

#### 단계 3) `R` 키 재시작

##### 세부목표
  - 입력 이벤트를 상태값 변화와 연결해 사용자의 조작이 게임 동작으로 이어지게 만든다.
  - 학습 목표는 `if/elif` 분기로 입력 조건을 나누고 상태 업데이트 순서를 이해하는 것이다.

```python
if event.type == pygame.KEYDOWN and event.key == pygame.K_r:
    reset_game()
```

##### 선언한 변수/함수의 목적
  - `state`: 초기화 함수 반환 상태를 잠깐 담아 여러 상태 변수를 한 번에 갱신한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if event.type == pygame.KEYDOWN and event.key == pygame.K_r:`: 이벤트 타입을 비교해 입력/업데이트 처리를 분기한다.
  - `reset_game()`: 위치·속도·장애물·상태를 초기값으로 되돌려 재시작 직후 플레이 가능한 상태를 즉시 복원한다.

---

## class 4. 점수 + 속도 증가 + 시작/일시정지 폴리싱

### 목표

- 시간 기반 점수를 올린다.
- 점수에 따라 이동 속도를 증가시킨다.
- `ENTER` 시작, `P` 일시정지, `R` 재시작을 구현한다.

### 핵심 변수/함수

- `READY`, `RUNNING`, `PAUSED`, `GAME_OVER`: 키 입력(`ENTER`, `P`, `R`)에 따라 전환되는 상태값. 상태별로 업데이트/안내문 출력 로직을 분리한다.
- `dt`: `clock.tick(60)`의 프레임 시간(ms). `score += dt * 0.01` 계산에 사용해 프레임 수와 무관하게 점수가 오르게 만든다.
- `score`: 누적 생존 점수. HUD의 `SCORE` 텍스트 출력과 난이도 계산(`int(score // 120)`)에 동시에 사용한다.
- `world_speed = 7 + min(8, int(score // 120))`: 점수 구간별 장애물 이동속도. 실제 이동식 `ob.x -= world_speed`에 들어간다.

### 단계별 구현

#### 단계 1) 상태 확장

##### 세부목표
  - 상태 확장 단계의 코드를 게임 루프에 연결해 실제 동작으로 확인한다.
  - 현재 단계의 상태값이 다음 단계 동작으로 어떻게 이어지는지 이해한다.

```python
READY = "ready"
RUNNING = "running"
PAUSED = "paused"
GAME_OVER = "game_over"
```

##### 선언한 변수/함수의 목적
  - `READY`: 시작 대기 상태 문자열
  - `RUNNING`: 게임 진행 상태 문자열
  - `PAUSED`: 일시정지 상태 문자열
  - `GAME_OVER`: 게임오버 상태 문자열

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `READY = "ready"`: 시작 대기 상태를 별도 상수로 두어 안내 화면과 실제 플레이 루프를 분리한다.
  - `RUNNING = "running"`: 진행 상태 문자열을 상수화해 오타 없이 상태 전환 분기를 재사용하게 한다.
  - `PAUSED = "paused"`: 일시정지 상태 상수를 두어 업데이트 중단과 재개 입력을 명확히 분기한다.
  - `GAME_OVER = "game_over"`: 종료 상태 문자열을 상수화해 충돌 후 입력/업데이트 제한 분기를 일관되게 처리한다.

#### 단계 2) 점수 누적

##### 세부목표
  - 점수 누적 단계의 코드를 게임 루프에 연결해 실제 동작으로 확인한다.
  - 현재 단계의 상태값이 다음 단계 동작으로 어떻게 이어지는지 이해한다.

```python
if game_state == RUNNING:
    score += dt * 0.01
```

##### 선언한 변수/함수의 목적
  - `state`: 초기화 함수 반환 상태를 잠깐 담아 여러 상태 변수를 한 번에 갱신한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if game_state == RUNNING:`: 실행 상태일 때만 이동/충돌/점수 갱신을 진행해 일시정지·종료 상태와 로직을 분리한다.
  - `score += dt * 0.01`: 프레임 시간(`dt`) 기반으로 점수를 누적해 FPS 차이가 있어도 같은 시간 대비 비슷하게 점수가 오른다.

#### 단계 3) 속도 증가

##### 세부목표
  - 속도 증가 단계의 코드를 게임 루프에 연결해 실제 동작으로 확인한다.
  - 현재 단계의 상태값이 다음 단계 동작으로 어떻게 이어지는지 이해한다.

```python
world_speed = 7 + min(8, int(score // 120))
```

##### 선언한 변수/함수의 목적
  - `world_speed`: 장애물 좌측 이동 속도로 값이 커질수록 체감 난이도가 올라간다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `world_speed = 7 + min(8, int(score // 120))`: 기본 이동 속도를 정해 장애물 이동 체감 난이도를 일정하게 시작한다.

#### 단계 4) 시작/일시정지 키

##### 세부목표
  - 시작/일시정지 키 단계의 코드를 게임 루프에 연결해 실제 동작으로 확인한다.
  - 현재 단계의 상태값이 다음 단계 동작으로 어떻게 이어지는지 이해한다.

```python
if event.key == pygame.K_RETURN and game_state == READY:
    game_state = RUNNING
elif event.key == pygame.K_p and game_state == RUNNING:
    game_state = PAUSED
elif event.key == pygame.K_p and game_state == PAUSED:
    game_state = RUNNING
```

##### 선언한 변수/함수의 목적
  - `game_state`: ready/running/paused/game_over 중 현재 상태를 저장해 전체 루프 분기를 제어한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if event.key == pygame.K_RETURN and game_state == READY:`: 대기 상태에서 Enter를 눌렀을 때만 게임을 시작 상태로 전환한다.
  - `game_state = RUNNING`: 초기 상태를 `RUNNING`으로 지정해 시작 직후 업데이트 루프가 바로 동작하게 한다.
  - `elif event.key == pygame.K_p and game_state == RUNNING:`: 플레이 중 P 입력에서만 일시정지로 전환해 상태 토글 기준을 명확히 한다.
  - `game_state = PAUSED`: 초기 상태를 `RUNNING`으로 지정해 시작 직후 업데이트 루프가 바로 동작하게 한다.

### class 4 최종 코드

```python
import random
import sys
import pygame

pygame.init()

WIDTH, HEIGHT = 1000, 320
GROUND_Y = 250

BG = (247, 247, 247)
LINE = (75, 75, 75)
DINO_COLOR = (28, 28, 28)
CACTUS = (36, 140, 74)
TEXT = (45, 45, 45)

DINO_W, DINO_H = 44, 52
DINO_X = 90
GRAVITY = 0.72
JUMP_POWER = -13.8

READY = "ready"
RUNNING = "running"
PAUSED = "paused"
GAME_OVER = "game_over"

SPAWN_EVENT = pygame.USEREVENT + 1

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Dino Run Codelab - Final")
clock = pygame.time.Clock()
font = pygame.font.SysFont("arial", 28, bold=True)
small_font = pygame.font.SysFont("arial", 20)

def make_dino_rect(y_value):
    return pygame.Rect(DINO_X, int(y_value), DINO_W, DINO_H)

def spawn_obstacle(obstacles):
    h = random.choice([36, 44, 58])
    w = random.choice([18, 22, 26])
    obstacles.append(pygame.Rect(WIDTH + 20, GROUND_Y - h, w, h))

def reset_game():
    return GROUND_Y - DINO_H, 0.0, [], 0.0

dino_y, dino_vy, obstacles, score = reset_game()
game_state = READY
pygame.time.set_timer(SPAWN_EVENT, 1050)

while True:
    dt = clock.tick(60)

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_RETURN and game_state == READY:
                game_state = RUNNING
            elif event.key == pygame.K_p and game_state == RUNNING:
                game_state = PAUSED
            elif event.key == pygame.K_p and game_state == PAUSED:
                game_state = RUNNING
            elif event.key == pygame.K_r and game_state in (GAME_OVER, READY):
                dino_y, dino_vy, obstacles, score = reset_game()
                game_state = RUNNING
            elif event.key == pygame.K_SPACE and game_state == RUNNING:
                on_ground = dino_y >= GROUND_Y - DINO_H
                if on_ground:
                    dino_vy = JUMP_POWER

        if event.type == SPAWN_EVENT and game_state == RUNNING:
            spawn_obstacle(obstacles)

    if game_state == RUNNING:
        dino_vy += GRAVITY
        dino_y += dino_vy
        if dino_y > GROUND_Y - DINO_H:
            dino_y = GROUND_Y - DINO_H
            dino_vy = 0

        world_speed = 7 + min(8, int(score // 120))
        for ob in obstacles:
            ob.x -= world_speed
        obstacles = [ob for ob in obstacles if ob.right > -10]

        dino_rect = make_dino_rect(dino_y)
        for ob in obstacles:
            if dino_rect.colliderect(ob):
                game_state = GAME_OVER
                break

        score += dt * 0.01

    screen.fill(BG)
    pygame.draw.line(screen, LINE, (0, GROUND_Y), (WIDTH, GROUND_Y), 3)

    for ob in obstacles:
        pygame.draw.rect(screen, CACTUS, ob, border_radius=3)

    dino_rect = make_dino_rect(dino_y)
    pygame.draw.rect(screen, DINO_COLOR, dino_rect, border_radius=4)
    pygame.draw.rect(screen, (255, 255, 255), (dino_rect.x + 30, dino_rect.y + 10, 8, 8))

    score_surface = font.render(f"SCORE {int(score):05d}", True, TEXT)
    screen.blit(score_surface, (WIDTH - score_surface.get_width() - 20, 20))

    if game_state == READY:
        msg = "ENTER: 시작  |  SPACE: 점프"
        txt = small_font.render(msg, True, TEXT)
        screen.blit(txt, (WIDTH // 2 - txt.get_width() // 2, 110))
    elif game_state == PAUSED:
        txt = small_font.render("일시정지 (P: 계속)", True, TEXT)
        screen.blit(txt, (WIDTH // 2 - txt.get_width() // 2, 110))
    elif game_state == GAME_OVER:
        over = font.render("GAME OVER", True, (190, 40, 40))
        retry = small_font.render("R: 재시작", True, TEXT)
        screen.blit(over, (WIDTH // 2 - over.get_width() // 2, 94))
        screen.blit(retry, (WIDTH // 2 - retry.get_width() // 2, 132))

    pygame.display.flip()
```
