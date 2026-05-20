## 개요

- 대상: `pygame`으로 처음 아케이드 물리 반사를 구현하는 초급 학습자
- 방식: class별로 "실행 코드 + 기능 1개 확장" 방식
- 최종 산출물: 패들/공/벽돌/라이프/점수 루프가 완성된 벽돌깨기 게임 1개
- 실행 환경: Python 3.10+ / `pip install pygame`

## 목차

1. class 1. 패들 이동 + 공 반사
2. class 2. 벽돌 생성 + 충돌 삭제
3. class 3. 라이프 + 게임오버 + 재시작
4. class 4. 점수/스테이지 + 파워업 폴리싱

---

## class 1. 패들 이동 + 공 반사

### 목표

- 패들을 좌우로 움직인다.
- 공을 벽과 패들에 반사시킨다.
- 바닥으로 떨어지면 라운드를 리셋한다.

### 핵심 변수/함수

- `paddle`: 패들 사각형(`Rect`)
- `ball`: 공 사각형(`Rect`)
- `vx`, `vy`: 공 속도 벡터
- `reset_round()`: 공/패들 초기 위치 복구

### 단계별 구현

#### 단계 1) 기본 창 + 객체 초기화

##### 세부목표
  - 게임 창, 패들, 공 상태를 만들어 실행 가능한 시작 화면을 만든다.
  - `Rect` 중심으로 위치를 관리하는 기준을 익힌다.

```python
import pygame
import sys

pygame.init()
WIDTH, HEIGHT = 760, 520
screen = pygame.display.set_mode((WIDTH, HEIGHT))
clock = pygame.time.Clock()

paddle = pygame.Rect(WIDTH // 2 - 60, HEIGHT - 42, 120, 16)
ball = pygame.Rect(WIDTH // 2 - 8, HEIGHT // 2, 16, 16)
vx, vy = 4, -4
```

##### 선언한 변수/함수의 목적
  - `paddle`: 패들 충돌/렌더링 사각형
  - `ball`: 공 충돌/렌더링 사각형
  - `vx`, `vy`: 공 이동 속도

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `paddle = pygame.Rect(...)`: 패들의 위치와 크기를 한 객체로 묶어 이동과 충돌 판정을 단순화한다.
  - `ball = pygame.Rect(...)`: 공의 좌표/크기를 `Rect`로 관리해 벽/패들과 같은 방식으로 충돌 검사를 재사용한다.
  - `vx, vy = 4, -4`: 시작 속도를 대각선으로 설정해 좌우/상하 반사 규칙을 모두 초반에 확인할 수 있다.
  - `WIDTH, HEIGHT = 760, 520`: 게임 영역 크기를 고정해 경계 충돌 기준을 명확히 만든다.

#### 단계 2) 패들 입력 처리

##### 세부목표
  - 키 입력을 패들 좌표 변화로 연결한다.
  - 화면 밖으로 나가지 않도록 경계 보정을 넣는다.

```python
keys = pygame.key.get_pressed()
if keys[pygame.K_LEFT]:
    paddle.x -= 7
if keys[pygame.K_RIGHT]:
    paddle.x += 7
paddle.x = max(0, min(WIDTH - paddle.width, paddle.x))
```

##### 선언한 변수/함수의 목적
  - `keys`: 현재 키 입력 상태 배열
  - `paddle.x`: 패들의 현재 x좌표

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `keys = pygame.key.get_pressed()`: 한 프레임에서 누른 키 상태를 읽어 연속 이동 입력을 처리한다.
  - `paddle.x -= 7`: 왼쪽 입력을 즉시 좌표 변화로 반영해 패들 반응성을 높인다.
  - `paddle.x += 7`: 오른쪽 입력을 동일 기준으로 처리해 좌우 이동 속도를 균형 있게 유지한다.
  - `paddle.x = max(0, min(...))`: 패들이 벽 밖으로 빠져나가는 것을 경계 값으로 강제 차단한다.

#### 단계 3) 공 이동 + 벽/패들 반사

##### 세부목표
  - 공 이동 업데이트를 매 프레임 적용한다.
  - 벽과 패들 충돌 시 진행 방향을 반전한다.

```python
ball.x += vx
ball.y += vy

if ball.left <= 0 or ball.right >= WIDTH:
    vx *= -1
if ball.top <= 0:
    vy *= -1
if ball.colliderect(paddle) and vy > 0:
    vy *= -1
```

##### 선언한 변수/함수의 목적
  - `ball.x`, `ball.y`: 공의 현재 위치
  - `vx`, `vy`: 반사 시 부호가 바뀌는 속도 값

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `ball.x += vx`: x축 속도를 누적해 프레임마다 공 위치를 갱신한다.
  - `if ball.left <= 0 or ball.right >= WIDTH:`: 좌우 벽 충돌을 검사해 x속도 부호를 뒤집는다.
  - `if ball.top <= 0:`: 상단 벽 충돌에서 y속도를 반전해 공이 아래로 되돌아오게 한다.
  - `if ball.colliderect(paddle) and vy > 0:`: 패들 위에서 내려오는 공만 반사해 중복 반전 버그를 줄인다.

#### 단계 4) 바닥 판정 + 라운드 리셋

##### 세부목표
  - 공이 바닥 아래로 내려가면 위치를 초기화한다.
  - 게임 흐름을 끊지 않고 즉시 다음 시도를 시작한다.

```python
def reset_round():
    paddle.x = WIDTH // 2 - paddle.width // 2
    ball.x = WIDTH // 2 - ball.width // 2
    ball.y = HEIGHT // 2
    return 4, -4

if ball.top > HEIGHT:
    vx, vy = reset_round()
```

##### 선언한 변수/함수의 목적
  - `reset_round()`: 라운드 초기화 함수
  - `ball.top`: 공의 상단 y좌표

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `paddle.x = WIDTH // 2 - paddle.width // 2`: 패들을 중앙으로 보내 다음 시도를 같은 조건에서 시작하게 한다.
  - `ball.x = WIDTH // 2 - ball.width // 2`: 공 x위치를 중앙으로 맞춰 출발 방향 비교를 쉽게 만든다.
  - `return 4, -4`: 속도 벡터를 함수 반환값으로 돌려줘 초기화 규칙을 한 곳에서 관리한다.
  - `if ball.top > HEIGHT:`: 공이 바닥 아래로 완전히 내려갔을 때만 리셋해 정상 반사 구간을 보존한다.

#### class 1 최종 코드

```python
import pygame
import sys

pygame.init()
WIDTH, HEIGHT = 760, 520
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Breakout - Class 1")
clock = pygame.time.Clock()

paddle = pygame.Rect(WIDTH // 2 - 60, HEIGHT - 42, 120, 16)
ball = pygame.Rect(WIDTH // 2 - 8, HEIGHT // 2, 16, 16)
vx, vy = 4, -4


def reset_round():
    paddle.x = WIDTH // 2 - paddle.width // 2
    ball.x = WIDTH // 2 - ball.width // 2
    ball.y = HEIGHT // 2
    return 4, -4


while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

    keys = pygame.key.get_pressed()
    if keys[pygame.K_LEFT]:
        paddle.x -= 7
    if keys[pygame.K_RIGHT]:
        paddle.x += 7
    paddle.x = max(0, min(WIDTH - paddle.width, paddle.x))

    ball.x += vx
    ball.y += vy

    if ball.left <= 0 or ball.right >= WIDTH:
        vx *= -1
    if ball.top <= 0:
        vy *= -1
    if ball.colliderect(paddle) and vy > 0:
        vy *= -1

    if ball.top > HEIGHT:
        vx, vy = reset_round()

    screen.fill((15, 23, 42))
    pygame.draw.rect(screen, (56, 189, 248), paddle, border_radius=8)
    pygame.draw.ellipse(screen, (241, 245, 249), ball)
    pygame.display.flip()
    clock.tick(60)
```

---

## class 2. 벽돌 생성 + 충돌 삭제

### 목표

- 벽돌 그리드를 만든다.
- 공이 벽돌에 닿으면 벽돌을 제거한다.
- 벽돌 충돌 방향에 맞게 속도를 반전한다.

### 핵심 변수/함수

- `bricks`: 벽돌 `Rect` 리스트
- `make_bricks()`: 벽돌 초기 생성 함수
- `hit_index`: 충돌한 벽돌 인덱스

### 단계별 구현

#### 단계 1) 벽돌 목록 생성

##### 세부목표
  - 반복문으로 벽돌 그리드를 자동 생성한다.
  - 추후 스테이지 확장 가능한 자료구조를 만든다.

```python
def make_bricks():
    items = []
    for r in range(5):
        for c in range(10):
            rect = pygame.Rect(24 + c * 71, 40 + r * 28, 66, 22)
            items.append(rect)
    return items

bricks = make_bricks()
```

##### 선언한 변수/함수의 목적
  - `make_bricks()`: 벽돌 초기화 함수
  - `items`: 생성 중인 벽돌 리스트
  - `bricks`: 게임 중 살아있는 벽돌 목록

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `for r in range(5):`: 행 단위 반복으로 벽돌 층 수를 고정한다.
  - `for c in range(10):`: 열 반복으로 한 줄 벽돌 수를 고정해 보드 폭을 균일하게 채운다.
  - `pygame.Rect(24 + c * 71, 40 + r * 28, 66, 22)`: 간격 포함 좌표 계산으로 벽돌 사이 시각적 여백을 만든다.
  - `bricks = make_bricks()`: 생성 함수를 분리해 재시작/스테이지 교체 시 같은 규칙을 재사용한다.

#### 단계 2) 벽돌 충돌 처리

##### 세부목표
  - 공과 벽돌 충돌을 검사해 맞은 벽돌을 제거한다.
  - 벽돌 충돌 시 공의 y속도를 반전한다.

```python
hit_index = ball.collidelist(bricks)
if hit_index != -1:
    bricks.pop(hit_index)
    vy *= -1
```

##### 선언한 변수/함수의 목적
  - `hit_index`: 충돌한 벽돌 인덱스
  - `bricks.pop()`: 충돌 벽돌 제거 연산

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `hit_index = ball.collidelist(bricks)`: 공과 충돌한 첫 벽돌 인덱스를 한 번에 찾는다.
  - `if hit_index != -1:`: 충돌이 실제 발생한 프레임에서만 제거/반사 처리를 실행한다.
  - `bricks.pop(hit_index)`: 맞은 벽돌을 목록에서 삭제해 다음 프레임부터 렌더링/충돌 대상에서 제외한다.
  - `vy *= -1`: 공 진행 방향을 반전해 벽돌 타격 반사 감각을 만든다.

#### 단계 3) 벽돌 렌더링

##### 세부목표
  - 벽돌 리스트를 순회해 화면에 그린다.
  - 충돌 후 제거 결과가 화면에 즉시 반영되게 한다.

```python
for brick in bricks:
    pygame.draw.rect(screen, (99, 102, 241), brick, border_radius=6)
```

##### 선언한 변수/함수의 목적
  - `brick`: 현재 렌더링 대상 벽돌

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `for brick in bricks:`: 살아있는 벽돌만 순회해 상태와 렌더링을 일치시킨다.
  - `pygame.draw.rect(...)`: 각 벽돌을 같은 크기/색상 규칙으로 그려 가독성을 유지한다.
  - `bricks` 길이가 줄면 반복 횟수도 자동으로 줄어 삭제 결과가 즉시 화면에 반영된다.
  - `border_radius=6`을 적용해 모서리를 둥글게 만들어 충돌 위치를 시각적으로 구분하기 쉽게 한다.

#### 단계 4) 클리어 조건 추가

##### 세부목표
  - 모든 벽돌을 깨면 새 라운드를 시작한다.
  - 반복 플레이가 가능한 기본 루프를 만든다.

```python
if not bricks:
    bricks = make_bricks()
    vx, vy = reset_round()
```

##### 선언한 변수/함수의 목적
  - `bricks`: 남은 벽돌 상태 목록

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if not bricks:`: 벽돌 리스트가 비었는지 검사해 클리어 시점을 정확히 판정한다.
  - `bricks = make_bricks()`: 새 벽돌을 재생성해 다음 라운드를 즉시 시작한다.
  - `vx, vy = reset_round()`: 공/패들 위치와 속도를 초기화해 라운드 시작 조건을 통일한다.
  - 클리어 처리와 초기화를 같은 분기에서 실행해 빈 화면 정지 상태를 방지한다.

#### class 2 최종 코드

```python
# class 1 코드에 make_bricks(), collidelist(), 클리어 분기를 추가해 실행
# 핵심: bricks 리스트를 생성하고 공 충돌 시 pop + vy 반전 처리
```

---

## class 3. 라이프 + 게임오버 + 재시작

### 목표

- 공이 바닥으로 떨어질 때 라이프를 감소시킨다.
- 라이프가 0이면 게임오버 처리한다.
- `R` 키로 재시작한다.

### 핵심 변수/함수

- `lives`: 남은 기회 수
- `game_over`: 종료 상태
- `reset_game()`: 전체 상태 초기화

### 단계별 구현

#### 단계 1) 라이프 상태 추가

##### 세부목표
  - 바닥 실패를 점수형 상태로 관리한다.
  - 즉시 종료가 아닌 다회 시도 구조를 만든다.

```python
lives = 3
game_over = False

if ball.top > HEIGHT:
    lives -= 1
    if lives <= 0:
        game_over = True
    else:
        vx, vy = reset_round()
```

##### 선언한 변수/함수의 목적
  - `lives`: 남은 기회 수
  - `game_over`: 종료 상태 플래그

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `lives = 3`: 시작 기회를 3회로 설정해 실패-재도전 학습 흐름을 만든다.
  - `lives -= 1`: 바닥 실패를 누적 카운트로 기록해 종료 조건 판단에 사용한다.
  - `if lives <= 0:`: 남은 기회가 없을 때만 전체 게임을 종료 상태로 전환한다.
  - `vx, vy = reset_round()`: 라이프가 남아 있으면 위치와 속도만 초기화해 즉시 재도전한다.

#### 단계 2) 재시작 함수

##### 세부목표
  - 게임 상태를 한 번에 초기화하는 함수를 만든다.
  - 입력으로 재시작을 실행한다.

```python
def reset_game():
    global bricks, lives, game_over
    bricks = make_bricks()
    lives = 3
    game_over = False
    return reset_round()

if event.type == pygame.KEYDOWN and event.key == pygame.K_r and game_over:
    vx, vy = reset_game()
```

##### 선언한 변수/함수의 목적
  - `reset_game()`: 전체 상태 초기화 함수
  - `bricks`: 벽돌 상태 목록

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `bricks = make_bricks()`: 벽돌 상태를 새로 만들어 이전 판 진행도를 완전히 초기화한다.
  - `lives = 3`: 남은 기회를 기본값으로 되돌려 새 게임 규칙을 복구한다.
  - `game_over = False`: 종료 플래그를 해제해 업데이트 루프가 다시 동작하도록 만든다.
  - `if event.type == ... and game_over:`: 게임오버 상태에서만 재시작을 허용해 진행 중 오입력을 방지한다.

#### 단계 3) 게임오버 메시지

##### 세부목표
  - 종료 상태를 화면으로 명확히 전달한다.
  - 재시작 방법을 안내한다.

```python
if game_over:
    over = font.render("Game Over", True, (248, 250, 252))
    hint = small.render("Press R to Restart", True, (148, 163, 184))
    screen.blit(over, (WIDTH // 2 - over.get_width() // 2, 250))
    screen.blit(hint, (WIDTH // 2 - hint.get_width() // 2, 288))
```

##### 선언한 변수/함수의 목적
  - `over`: 종료 제목 텍스트
  - `hint`: 재시작 안내 텍스트

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if game_over:`: 종료 상태일 때만 오버레이를 렌더링해 플레이 중 UI와 구분한다.
  - `font.render("Game Over", ...)`: 상태 문자열을 강조 텍스트로 렌더링한다.
  - `screen.blit(over, ...)`: 중앙 정렬 배치로 시선을 즉시 모아 종료 상태 인지를 빠르게 만든다.
  - `screen.blit(hint, ...)`: 재시작 키 안내를 함께 표시해 사용자 동선을 끊지 않는다.

#### 단계 4) 라이프 HUD

##### 세부목표
  - 현재 남은 라이프를 항상 표시한다.
  - 실패 이후 상태 변화를 바로 확인하게 한다.

```python
life_text = small.render(f"Lives: {lives}", True, (226, 232, 240))
screen.blit(life_text, (12, 10))
```

##### 선언한 변수/함수의 목적
  - `life_text`: 라이프 표시 HUD 텍스트

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `f"Lives: {lives}"`: 상태값을 문자열로 합쳐 현재 남은 기회를 직관적으로 보여준다.
  - `small.render(...)`: 작은 글씨 크기로 HUD를 만들어 플레이 화면 가림을 최소화한다.
  - `screen.blit(life_text, (12, 10))`: 좌상단 고정 위치에 배치해 프레임마다 같은 위치에서 읽게 한다.
  - 라이프 값이 변할 때마다 다시 렌더링해 실패 직후 상태 변화가 즉시 반영되게 한다.

#### class 3 최종 코드

```python
# class 2 코드에 lives/game_over/reset_game/HUD를 통합해 실행
# 핵심: 바닥 실패 시 lives 감소, 0이면 game_over=True, R로 전체 상태 초기화
```

---

## class 4. 점수/스테이지 + 파워업 폴리싱

### 목표

- 벽돌 파괴 점수와 스테이지를 추가한다.
- 간단한 파워업(패들 길이 확장)을 넣는다.
- 최종 실행 가능한 벽돌깨기 코드를 완성한다.

### 핵심 변수/함수

- `score`: 누적 점수
- `stage`: 현재 스테이지
- `power_timer`: 패들 확장 남은 프레임

### 단계별 구현

#### 단계 1) 점수/스테이지

##### 세부목표
  - 벽돌 파괴를 점수 증가로 연결한다.
  - 클리어 시 스테이지를 올리고 공 속도를 높인다.

```python
score = 0
stage = 1

if hit_index != -1:
    bricks.pop(hit_index)
    vy *= -1
    score += 100 * stage

if not bricks:
    stage += 1
    bricks = make_bricks()
    vx, vy = (4 + stage, -4 - stage)
```

##### 선언한 변수/함수의 목적
  - `score`: 누적 점수
  - `stage`: 난이도 단계

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `score += 100 * stage`: 스테이지가 높을수록 같은 벽돌에도 더 높은 점수를 주어 난이도 상승 보상을 만든다.
  - `if not bricks:`: 벽돌 전부 파괴 시점을 스테이지 전환 트리거로 사용한다.
  - `stage += 1`: 클리어 횟수를 단계값으로 누적해 이후 속도/점수 계산의 공통 기준으로 쓴다.
  - `vx, vy = (4 + stage, -4 - stage)`: 스테이지 상승에 맞춰 공 속도를 올려 템포를 점진적으로 높인다.

#### 단계 2) 파워업 드롭 + 패들 확장

##### 세부목표
  - 확률적으로 떨어지는 파워업 아이템을 만든다.
  - 패들이 아이템을 받으면 일정 시간 길이를 늘린다.

```python
import random

power = None
power_timer = 0

if hit_index != -1 and power is None and random.random() < 0.12:
    power = pygame.Rect(ball.centerx - 10, ball.centery - 10, 20, 20)

if power is not None:
    power.y += 4
    if power.colliderect(paddle):
        paddle.width = 170
        power_timer = 480
        power = None
```

##### 선언한 변수/함수의 목적
  - `power`: 떨어지는 파워업 아이템 Rect
  - `power_timer`: 확장 유지 프레임 카운트

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `random.random() < 0.12`: 벽돌 파괴 중 일부만 아이템을 생성해 매번 같은 패턴이 되지 않게 만든다.
  - `power = pygame.Rect(...)`: 아이템 위치를 충돌 지점 근처에서 생성해 인과관계를 눈으로 확인하게 한다.
  - `if power.colliderect(paddle):`: 패들이 아이템과 닿은 순간에만 효과를 발동한다.
  - `paddle.width = 170`: 패들 길이를 늘려 반사 난이도를 일시적으로 낮추는 보상 효과를 만든다.

#### 단계 3) 파워업 만료 처리

##### 세부목표
  - 확장 효과가 영구 적용되지 않게 만료 타이머를 둔다.
  - 만료 시 기본 패들 크기로 복귀한다.

```python
if power_timer > 0:
    power_timer -= 1
    if power_timer == 0:
        paddle.width = 120
```

##### 선언한 변수/함수의 목적
  - `power_timer`: 효과 남은 시간
  - `paddle.width`: 패들 크기 상태

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if power_timer > 0:`: 활성화된 효과가 있을 때만 만료 카운트를 진행한다.
  - `power_timer -= 1`: 프레임 기준으로 남은 시간을 줄여 자연스럽게 효과가 끝나게 만든다.
  - `if power_timer == 0:`: 정확히 만료되는 시점에만 복귀 로직을 실행한다.
  - `paddle.width = 120`: 기본 패들 크기로 되돌려 난이도를 원래 상태로 복구한다.

#### 단계 4) HUD 폴리싱

##### 세부목표
  - 점수/스테이지/라이프를 동시에 표시한다.
  - 파워업 활성 상태도 텍스트로 안내한다.

```python
hud = small.render(f"Score {score}  Stage {stage}  Lives {lives}", True, (226, 232, 240))
screen.blit(hud, (12, 10))
if power_timer > 0:
    tip = small.render("Power: Long Paddle", True, (250, 204, 21))
    screen.blit(tip, (12, 32))
```

##### 선언한 변수/함수의 목적
  - `hud`: 기본 진행 정보 텍스트
  - `tip`: 파워업 활성 안내 텍스트

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `f"Score {score}  Stage {stage}  Lives {lives}"`: 핵심 진행 상태를 한 줄로 묶어 판단 속도를 높인다.
  - `screen.blit(hud, (12, 10))`: HUD를 좌상단 고정 위치에 배치해 게임 시야를 크게 가리지 않게 한다.
  - `if power_timer > 0:`: 파워업이 실제 활성일 때만 안내 문구를 표시해 정보 노이즈를 줄인다.
  - `tip = small.render("Power: Long Paddle", ...)`: 효과 이름을 직접 출력해 현재 보정 상태를 명확히 전달한다.

#### class 4 최종 코드

```python
import pygame
import random
import sys

pygame.init()
WIDTH, HEIGHT = 760, 520
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Breakout - Final")
clock = pygame.time.Clock()
small = pygame.font.SysFont("arial", 20)
font = pygame.font.SysFont("arial", 38, bold=True)

paddle = pygame.Rect(WIDTH // 2 - 60, HEIGHT - 42, 120, 16)
ball = pygame.Rect(WIDTH // 2 - 8, HEIGHT // 2, 16, 16)
vx, vy = 4, -4

score = 0
stage = 1
lives = 3
game_over = False
power = None
power_timer = 0


def make_bricks():
    items = []
    for r in range(5):
        for c in range(10):
            rect = pygame.Rect(24 + c * 71, 40 + r * 28, 66, 22)
            items.append(rect)
    return items


def reset_round():
    paddle.x = WIDTH // 2 - paddle.width // 2
    ball.x = WIDTH // 2 - ball.width // 2
    ball.y = HEIGHT // 2
    return 4 + stage - 1, -(4 + stage - 1)


def reset_game():
    global score, stage, lives, game_over, power, power_timer, bricks
    score = 0
    stage = 1
    lives = 3
    game_over = False
    power = None
    power_timer = 0
    paddle.width = 120
    bricks = make_bricks()
    return reset_round()


bricks = make_bricks()

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
                vx, vy = reset_game()

    keys = pygame.key.get_pressed()
    if not game_over:
        if keys[pygame.K_LEFT]:
            paddle.x -= 8
        if keys[pygame.K_RIGHT]:
            paddle.x += 8
    paddle.x = max(0, min(WIDTH - paddle.width, paddle.x))

    if not game_over:
        ball.x += vx
        ball.y += vy

        if ball.left <= 0 or ball.right >= WIDTH:
            vx *= -1
        if ball.top <= 0:
            vy *= -1
        if ball.colliderect(paddle) and vy > 0:
            vy *= -1

        hit_index = ball.collidelist(bricks)
        if hit_index != -1:
            bricks.pop(hit_index)
            vy *= -1
            score += 100 * stage
            if power is None and random.random() < 0.12:
                power = pygame.Rect(ball.centerx - 10, ball.centery - 10, 20, 20)

        if not bricks:
            stage += 1
            bricks = make_bricks()
            vx, vy = reset_round()

        if ball.top > HEIGHT:
            lives -= 1
            if lives <= 0:
                game_over = True
            else:
                paddle.width = 120
                power_timer = 0
                power = None
                vx, vy = reset_round()

        if power is not None:
            power.y += 4
            if power.colliderect(paddle):
                paddle.width = 170
                power_timer = 480
                power = None
            elif power.top > HEIGHT:
                power = None

        if power_timer > 0:
            power_timer -= 1
            if power_timer == 0:
                paddle.width = 120

    screen.fill((15, 23, 42))

    for brick in bricks:
        pygame.draw.rect(screen, (99, 102, 241), brick, border_radius=6)

    pygame.draw.rect(screen, (56, 189, 248), paddle, border_radius=8)
    pygame.draw.ellipse(screen, (241, 245, 249), ball)

    if power is not None:
        pygame.draw.ellipse(screen, (250, 204, 21), power)

    hud = small.render(f"Score {score}  Stage {stage}  Lives {lives}", True, (226, 232, 240))
    screen.blit(hud, (12, 10))
    if power_timer > 0:
        tip = small.render("Power: Long Paddle", True, (250, 204, 21))
        screen.blit(tip, (12, 32))

    if game_over:
        over = font.render("Game Over", True, (248, 250, 252))
        hint = small.render("Press R to Restart", True, (148, 163, 184))
        screen.blit(over, (WIDTH // 2 - over.get_width() // 2, 250))
        screen.blit(hint, (WIDTH // 2 - hint.get_width() // 2, 288))

    pygame.display.flip()
    clock.tick(60)
```
