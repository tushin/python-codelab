## 개요

- 대상: `pygame` 기반 게임을 만든 뒤 물리엔진을 처음 붙여보는 학습자
- 방식: class별로 "실행 코드 + 기능 1개 확장" 방식
- 최종 산출물: 낙하, 충돌, 합체, 점수 시스템이 있는 스이카게임 스타일 물리 퍼즐 1개
- 실행 환경: Python 3.10+ / `pip install pygame pymunk`

## 목차

1. class 1. 물리 월드 + 과일 낙하
2. class 2. 충돌 판정 + 같은 과일 합체
3. class 3. 점수 + 게임오버 + 재시작
4. class 4. 다음 과일 미리보기 + 폴리싱

---

## class 1. 물리 월드 + 과일 낙하

### 목표

- `pymunk.Space`로 중력 월드를 만든다.
- 통(컨테이너) 벽을 만들고 과일 공이 튕기며 쌓이게 한다.
- 클릭한 x좌표에서 과일을 떨어뜨린다.

### 핵심 변수/함수

- `space`: 물리 시뮬레이션 월드
- `spawn_fruit()`: 원형 과일 바디 생성
- `fruits`: 현재 과일 바디 목록
- `step_dt`: 물리 스텝 간격

### 단계별 구현

#### 단계 1) 물리 월드와 경계 만들기

##### 세부목표
  - 중력과 고정 벽을 구성해 공이 쌓일 수 있는 기본 환경을 만든다.
  - 렌더링 좌표와 물리 좌표를 같은 축으로 맞춘다.

```python
import pygame
import pymunk

pygame.init()
WIDTH, HEIGHT = 520, 760
screen = pygame.display.set_mode((WIDTH, HEIGHT))
clock = pygame.time.Clock()

space = pymunk.Space()
space.gravity = (0, 980)

left = pymunk.Segment(space.static_body, (80, 120), (80, 700), 6)
right = pymunk.Segment(space.static_body, (440, 120), (440, 700), 6)
floor = pymunk.Segment(space.static_body, (80, 700), (440, 700), 6)
space.add(left, right, floor)
```

##### 선언한 변수/함수의 목적
  - `space`: 물리 시뮬레이션 월드
  - `left`, `right`, `floor`: 컨테이너 경계 세그먼트

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `space = pymunk.Space()`: 물리 바디/충돌/중력 계산을 담당하는 월드를 생성한다.
  - `space.gravity = (0, 980)`: y축 중력을 설정해 과일이 아래로 자연 낙하하게 만든다.
  - `pymunk.Segment(..., 6)`: 컨테이너 벽을 선분 충돌체로 만들어 공이 밖으로 빠지지 않게 막는다.
  - `space.add(left, right, floor)`: 경계 충돌체를 월드에 등록해 시뮬레이션 대상에 포함한다.

#### 단계 2) 과일 생성 함수

##### 세부목표
  - 클릭 위치에서 과일 원형 바디를 생성한다.
  - 질량/마찰/탄성을 설정해 쌓이는 움직임을 만든다.

```python
fruits = []


def spawn_fruit(x, level=0):
    radius = [14, 20, 28, 38, 50][level]
    mass = 1 + level * 0.6
    moment = pymunk.moment_for_circle(mass, 0, radius)
    body = pymunk.Body(mass, moment)
    body.position = (x, 140)
    shape = pymunk.Circle(body, radius)
    shape.friction = 0.55
    shape.elasticity = 0.2
    space.add(body, shape)
    fruits.append({"body": body, "shape": shape, "level": level})
```

##### 선언한 변수/함수의 목적
  - `spawn_fruit()`: 과일 생성 함수
  - `radius`: 레벨별 반지름
  - `mass`: 과일 질량
  - `fruits`: 과일 상태 목록

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `radius = [14, 20, 28, 38, 50][level]`: 과일 단계별 크기를 고정해 합체 성장 규칙을 명확히 만든다.
  - `moment = pymunk.moment_for_circle(...)`: 원형 회전 관성을 계산해 충돌 시 자연스러운 회전/밀림을 만든다.
  - `shape.friction = 0.55`: 마찰 계수를 부여해 과일이 접촉 후 바로 미끄러지지 않고 어느 정도 쌓이게 한다.
  - `fruits.append({...})`: 물리 객체와 레벨 정보를 함께 저장해 렌더링/합체 로직에서 같은 데이터를 참조한다.

#### 단계 3) 입력으로 과일 드롭

##### 세부목표
  - 마우스 클릭 위치를 과일 생성 좌표로 연결한다.
  - 벽 밖 생성 입력을 보정한다.

```python
if event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
    mx, _ = pygame.mouse.get_pos()
    drop_x = max(96, min(424, mx))
    spawn_fruit(drop_x, 0)
```

##### 선언한 변수/함수의 목적
  - `mx`: 마우스 x좌표
  - `drop_x`: 보정된 생성 x좌표

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `pygame.mouse.get_pos()`: 현재 클릭 위치를 읽어 과일 생성 입력으로 사용한다.
  - `drop_x = max(96, min(424, mx))`: 컨테이너 벽 안쪽 범위로 좌표를 강제해 벽 밖 스폰을 막는다.
  - `spawn_fruit(drop_x, 0)`: 클릭 이벤트를 실제 물리 바디 생성으로 연결한다.
  - `event.button == 1`: 좌클릭만 드롭 입력으로 허용해 다른 버튼 이벤트와 분리한다.

#### 단계 4) 물리 스텝 + 렌더링

##### 세부목표
  - 고정 시간 간격으로 물리 엔진을 업데이트한다.
  - 과일 원형을 화면에 그려 충돌 결과를 확인한다.

```python
step_dt = 1 / 60
space.step(step_dt)

screen.fill((10, 35, 25))
for item in fruits:
    pos = item["body"].position
    r = int(item["shape"].radius)
    pygame.draw.circle(screen, (253, 224, 71), (int(pos.x), int(pos.y)), r)
```

##### 선언한 변수/함수의 목적
  - `step_dt`: 물리 업데이트 시간 간격
  - `pos`: 과일 현재 위치

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `space.step(step_dt)`: 물리 월드를 한 틱 진행해 중력/충돌/반발 계산을 갱신한다.
  - `for item in fruits:`: 모든 과일을 순회해 현재 물리 위치를 화면에 반영한다.
  - `pos = item["body"].position`: 물리 엔진이 계산한 좌표를 읽어 렌더링 기준으로 사용한다.
  - `pygame.draw.circle(...)`: 과일 반지름과 위치를 원으로 그려 쌓임 상태를 즉시 시각화한다.

#### class 1 최종 코드

```python
# class 1은 위 단계 코드를 한 파일로 합쳐 실행
# 핵심: space 생성, 경계 등록, spawn_fruit(), 클릭 드롭, space.step()
```

---

## class 2. 충돌 판정 + 같은 과일 합체

### 목표

- 같은 레벨 과일끼리 부딪히면 상위 과일로 합체한다.
- 충돌 콜백에서 합체 후보를 수집한다.
- 한 프레임 뒤 안전하게 기존 과일을 제거/생성한다.

### 핵심 변수/함수

- `pending_merge`: 합체 대기 큐
- `fruit_by_shape`: shape -> fruit 매핑
- `on_collision()`: 물리 충돌 콜백

### 단계별 구현

#### 단계 1) shape 매핑 테이블

##### 세부목표
  - 충돌 콜백에서 shape로 과일 정보를 찾는 구조를 만든다.
  - 합체 큐를 위한 상태 저장소를 준비한다.

```python
fruit_by_shape = {}
pending_merge = []

# spawn_fruit() 끝부분
fruit = {"body": body, "shape": shape, "level": level}
fruits.append(fruit)
fruit_by_shape[shape] = fruit
```

##### 선언한 변수/함수의 목적
  - `fruit_by_shape`: 충돌 shape 역참조 테이블
  - `pending_merge`: 합체 예약 목록

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `fruit_by_shape = {}`: 콜백에서 shape로 즉시 원본 과일 정보를 찾기 위한 해시 테이블을 만든다.
  - `pending_merge = []`: 충돌 시점엔 즉시 제거하지 않고 예약 처리할 큐를 분리한다.
  - `fruit_by_shape[shape] = fruit`: 물리 shape와 게임 상태를 연결해 충돌 이벤트 데이터를 게임 로직으로 변환한다.
  - `fruits.append(fruit)`: 렌더링/점수/합체 로직이 같은 객체를 참조하도록 목록에 등록한다.

#### 단계 2) 충돌 콜백 등록

##### 세부목표
  - 과일끼리 충돌할 때 같은 레벨 여부를 검사한다.
  - 합체 대상 좌표를 큐에 넣는다.

```python
FRUIT_COLLISION = 7

# spawn_fruit() 안
shape.collision_type = FRUIT_COLLISION


def on_collision(arbiter, space_, data):
    s1, s2 = arbiter.shapes
    f1 = fruit_by_shape.get(s1)
    f2 = fruit_by_shape.get(s2)
    if not f1 or not f2:
        return True
    if f1["level"] != f2["level"] or f1["level"] >= 4:
        return True
    mx = (f1["body"].position.x + f2["body"].position.x) * 0.5
    my = (f1["body"].position.y + f2["body"].position.y) * 0.5
    pending_merge.append((f1, f2, int(mx), int(my), f1["level"] + 1))
    return True

handler = space.add_collision_handler(FRUIT_COLLISION, FRUIT_COLLISION)
handler.begin = on_collision
```

##### 선언한 변수/함수의 목적
  - `FRUIT_COLLISION`: 과일 충돌 타입 ID
  - `on_collision()`: 충돌 콜백 함수
  - `mx`, `my`: 합체 생성 좌표

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `shape.collision_type = FRUIT_COLLISION`: 과일 shape를 같은 그룹으로 분류해 전용 충돌 콜백을 연결한다.
  - `f1 = fruit_by_shape.get(s1)`: 물리 shape를 게임 객체로 역참조해 레벨/점수 로직을 적용 가능하게 만든다.
  - `if f1["level"] != f2["level"] ...`: 같은 레벨일 때만 합체를 허용해 규칙을 명확히 유지한다.
  - `pending_merge.append((...))`: 콜백 시점엔 예약만 하고 실제 삭제/생성은 메인 루프에서 안전하게 처리한다.

#### 단계 3) 합체 큐 처리

##### 세부목표
  - 예약된 합체를 프레임 루프에서 실행한다.
  - 기존 과일 2개를 제거하고 상위 과일 1개를 생성한다.

```python
def remove_fruit(fruit):
    if fruit in fruits:
        fruits.remove(fruit)
    fruit_by_shape.pop(fruit["shape"], None)
    space.remove(fruit["shape"], fruit["body"])


def apply_merges():
    while pending_merge:
        f1, f2, mx, my, next_level = pending_merge.pop(0)
        if f1 not in fruits or f2 not in fruits:
            continue
        remove_fruit(f1)
        remove_fruit(f2)
        spawn_fruit(mx, next_level)
```

##### 선언한 변수/함수의 목적
  - `remove_fruit()`: 과일 제거 함수
  - `apply_merges()`: 합체 예약 처리 함수

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `fruit_by_shape.pop(..., None)`: 매핑 테이블에서도 제거해 이후 충돌 콜백에서 유령 참조가 생기지 않게 한다.
  - `space.remove(fruit["shape"], fruit["body"])`: 물리 월드에서 객체를 제거해 실제 충돌 계산 대상에서 제외한다.
  - `if f1 not in fruits or f2 not in fruits:`: 이미 처리된 중복 합체 이벤트를 건너뛰어 이중 제거 오류를 막는다.
  - `spawn_fruit(mx, next_level)`: 두 과일 중심 좌표에서 상위 레벨 과일을 생성해 합체 결과를 자연스럽게 연결한다.

#### 단계 4) 합체 후 흔들림 안정화

##### 세부목표
  - 연속 합체 시 과도한 떨림을 줄인다.
  - 충돌 직후 속도를 일부 감쇠해 안정적인 쌓임을 만든다.

```python
# spawn_fruit() 끝부분
body.velocity = (body.velocity.x * 0.85, body.velocity.y * 0.85)
```

##### 선언한 변수/함수의 목적
  - `body.velocity`: 과일 속도 벡터

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `body.velocity.x * 0.85`: x속도를 감쇠해 합체 직후 과일이 과하게 튀는 현상을 줄인다.
  - `body.velocity.y * 0.85`: y속도도 함께 줄여 합체 후 통 바닥에서 빠르게 안정화되게 만든다.
  - 감쇠 계수를 적용해 물리엔진의 자연 반발은 유지하되 시각적 노이즈를 줄인다.
  - 합체 직후에만 속도를 조정해 일반 낙하 감각은 크게 해치지 않는다.

#### class 2 최종 코드

```python
# class 1 코드에 fruit_by_shape, on_collision, apply_merges()를 추가해 실행
# 핵심: 같은 레벨 충돌 시 pending_merge에 예약하고 루프에서 안전 처리
```

---

## class 3. 점수 + 게임오버 + 재시작

### 목표

- 합체 시 점수를 증가시킨다.
- 과일이 상단 경계선을 오래 넘으면 게임오버 처리한다.
- `R` 키로 게임을 재시작한다.

### 핵심 변수/함수

- `score`: 누적 점수
- `danger_frames`: 위험선 초과 프레임 카운트
- `game_over`: 종료 상태

### 단계별 구현

#### 단계 1) 점수 계산

##### 세부목표
  - 합체 레벨에 따라 점수를 차등 증가시킨다.
  - HUD에 즉시 반영 가능한 누적값을 만든다.

```python
score = 0

# apply_merges() 안
score += [0, 10, 30, 80, 200, 500][next_level]
```

##### 선언한 변수/함수의 목적
  - `score`: 누적 점수
  - `next_level`: 합체 후 생성 레벨

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `score = 0`: 점수를 명시적으로 초기화해 재시작 시 상태 오염을 막는다.
  - `[0, 10, 30, 80, 200, 500][next_level]`: 레벨별 점수 테이블로 성장 난이도 대비 보상을 차등화한다.
  - `score += ...`: 합체 이벤트를 즉시 점수 변화로 연결해 플레이 피드백을 강화한다.
  - `next_level` 기반 계산으로 큰 과일 합체일수록 높은 보상을 주는 규칙을 만든다.

#### 단계 2) 위험선 게임오버

##### 세부목표
  - 상단에 과일이 오래 머무르면 종료하도록 규칙을 만든다.
  - 순간 스치기 오판정을 피하기 위해 프레임 누적 기준을 둔다.

```python
danger_frames = 0
game_over = False

if any(item["body"].position.y < 150 for item in fruits):
    danger_frames += 1
else:
    danger_frames = 0

if danger_frames > 180:
    game_over = True
```

##### 선언한 변수/함수의 목적
  - `danger_frames`: 위험 상태 누적 프레임
  - `game_over`: 종료 상태

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `any(item["body"].position.y < 150 for item in fruits)`: 위험선 위에 있는 과일이 하나라도 있는지 빠르게 판정한다.
  - `danger_frames += 1`: 위험 상태가 지속된 시간을 프레임 단위로 누적한다.
  - `danger_frames = 0`: 위험선에서 내려오면 누적값을 초기화해 일시 스침을 실패로 보지 않게 한다.
  - `if danger_frames > 180:`: 약 3초(60fps 기준) 이상 위험 상태일 때만 게임오버를 확정한다.

#### 단계 3) 재시작 함수

##### 세부목표
  - 물리 월드 과일을 전부 제거한다.
  - 점수/상태값을 초기화한다.

```python
def reset_game():
    global score, game_over, danger_frames
    for fruit in fruits[:]:
        remove_fruit(fruit)
    score = 0
    game_over = False
    danger_frames = 0
```

##### 선언한 변수/함수의 목적
  - `reset_game()`: 전체 상태 초기화 함수

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `for fruit in fruits[:]:`: 복사본 순회로 제거 중 리스트 변경 오류를 피한다.
  - `remove_fruit(fruit)`: 렌더링 목록/매핑/물리 월드에서 동시에 제거해 상태 불일치를 막는다.
  - `score = 0`: 점수를 초기화해 새 게임의 보상 누적을 독립적으로 시작한다.
  - `game_over = False`: 종료 플래그를 해제해 드롭/물리 업데이트 루프를 다시 활성화한다.

#### 단계 4) 점수/종료 HUD

##### 세부목표
  - 점수와 종료 안내를 화면에 표시한다.
  - 플레이 중과 종료 상태를 명확히 구분한다.

```python
hud = small.render(f"Score: {score}", True, (236, 253, 245))
screen.blit(hud, (16, 14))
if game_over:
    msg = title.render("Game Over", True, (254, 226, 226))
    hint = small.render("Press R to Restart", True, (254, 202, 202))
```

##### 선언한 변수/함수의 목적
  - `hud`: 점수 HUD 텍스트
  - `msg`, `hint`: 종료 안내 텍스트

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `small.render(f"Score: {score}", ...)`: 현재 점수를 문자열로 렌더링해 합체 보상이 즉시 보이게 한다.
  - `screen.blit(hud, (16, 14))`: HUD를 좌상단에 고정 배치해 지속적으로 확인 가능하게 만든다.
  - `if game_over:`: 종료 상태에서만 안내 문구를 렌더링해 정상 플레이 화면과 구분한다.
  - `hint = small.render("Press R to Restart", ...)`: 사용자가 다음 행동을 즉시 알 수 있게 재시작 입력을 명시한다.

#### class 3 최종 코드

```python
# class 2 코드에 score, danger_frames, game_over, reset_game()를 통합해 실행
# 핵심: 합체 점수 증가 + 위험선 지속 초과 시 게임오버 + R 재시작
```

---

## class 4. 다음 과일 미리보기 + 폴리싱

### 목표

- 다음 드롭 과일을 미리 보여준다.
- 드롭 쿨다운을 넣어 과다 입력을 막는다.
- 최종 실행 가능한 스이카 물리엔진 코드를 완성한다.

### 핵심 변수/함수

- `next_level`: 다음 과일 레벨
- `drop_cooldown`: 드롭 간격 제어
- `draw_preview()`: 다음 과일 미리보기 UI

### 단계별 구현

#### 단계 1) 다음 과일 상태

##### 세부목표
  - 드롭 전에 다음 과일 레벨을 미리 정한다.
  - 드롭 후 다음 값을 갱신한다.

```python
next_level = 0

# 드롭 시
spawn_fruit(drop_x, next_level)
next_level = min(2, random.randint(0, 2))
```

##### 선언한 변수/함수의 목적
  - `next_level`: 다음 생성될 과일 레벨

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `next_level = 0`: 시작 과일을 고정해 초반 난이도 급등을 막는다.
  - `spawn_fruit(drop_x, next_level)`: 미리보기 상태를 실제 드롭 결과로 직접 연결한다.
  - `random.randint(0, 2)`: 작은 과일 중심 랜덤으로 초반 플레이 안정성을 유지한다.
  - `next_level = min(2, ...)`: 생성 가능한 초기 레벨 상한을 제한해 게임 템포를 조절한다.

#### 단계 2) 드롭 쿨다운

##### 세부목표
  - 연속 클릭으로 과도한 생성이 일어나지 않게 한다.
  - 일정 프레임이 지난 뒤에만 다음 드롭을 허용한다.

```python
drop_cooldown = 0

if drop_cooldown > 0:
    drop_cooldown -= 1

if event.type == pygame.MOUSEBUTTONDOWN and drop_cooldown == 0 and not game_over:
    spawn_fruit(drop_x, next_level)
    drop_cooldown = 14
```

##### 선언한 변수/함수의 목적
  - `drop_cooldown`: 드롭 간격 프레임 카운트

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if drop_cooldown > 0:`: 쿨다운 활성 상태에서만 카운트를 줄여 음수로 내려가는 것을 방지한다.
  - `drop_cooldown -= 1`: 프레임마다 간격을 줄여 일정 시간 뒤 재드롭이 가능하게 만든다.
  - `drop_cooldown == 0 and not game_over`: 드롭 가능 조건과 종료 상태를 함께 검사해 잘못된 입력을 차단한다.
  - `drop_cooldown = 14`: 약 0.23초(60fps 기준) 간격을 둬 조작은 빠르되 과밀 생성은 막는다.

#### 단계 3) 미리보기 렌더링

##### 세부목표
  - 다음 과일 정보를 UI로 표시한다.
  - 현재 게임 상태 판단을 돕는다.

```python
def draw_preview(surface, level):
    radius = [14, 20, 28, 38, 50][level]
    pygame.draw.circle(surface, (187, 247, 208), (470, 76), radius)
```

##### 선언한 변수/함수의 목적
  - `draw_preview()`: 다음 과일 미리보기 함수
  - `radius`: 레벨별 표시 반지름

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `radius = [14, 20, 28, 38, 50][level]`: 실제 드롭 크기와 같은 규칙을 UI에도 적용해 예측 정확도를 높인다.
  - `pygame.draw.circle(...)`: 미리보기 원을 별도 위치에 렌더링해 플레이 영역과 구분한다.
  - `level` 기반 렌더링으로 드롭 전에 합체 전략을 세울 수 있게 돕는다.
  - 고정 좌표 `(470, 76)`를 사용해 HUD처럼 항상 같은 위치에서 읽히게 한다.

#### 단계 4) 최종 루프 정리

##### 세부목표
  - 입력, 합체, 점수, 게임오버, UI를 하나의 루프로 통합한다.
  - 실습에서 바로 실행 가능한 최종 버전을 완성한다.

```python
# 루프 순서
# 1) 이벤트 입력
# 2) 물리 step
# 3) 합체 큐 처리
# 4) 위험선/게임오버 판정
# 5) HUD + 미리보기 렌더링
```

##### 선언한 변수/함수의 목적
  - `pending_merge`: 합체 예약 큐

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `event 입력`을 먼저 처리해 드롭/재시작 같은 사용자 조작이 프레임 지연 없이 반영되게 한다.
  - `space.step(...)` 이후 `apply_merges()`를 호출해 충돌 결과 기반 합체가 같은 프레임 흐름에서 정리되게 한다.
  - `위험선 판정`을 렌더링 전 수행해 HUD와 게임오버 문구가 최신 상태를 보여주게 한다.
  - `HUD + 미리보기`를 마지막에 그려 업데이트된 점수/다음 과일 정보를 정확히 표시한다.

#### class 4 최종 코드

```python
import pygame
import pymunk
import random
import sys

pygame.init()
WIDTH, HEIGHT = 520, 760
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Suika Physics - Final")
clock = pygame.time.Clock()
small = pygame.font.SysFont("arial", 22)
title = pygame.font.SysFont("arial", 42, bold=True)

space = pymunk.Space()
space.gravity = (0, 980)

wall_color = (134, 239, 172)
fruit_colors = [(254, 202, 202), (253, 186, 116), (253, 224, 71), (134, 239, 172), (147, 197, 253)]
radii = [14, 20, 28, 38, 50]

left = pymunk.Segment(space.static_body, (80, 120), (80, 700), 6)
right = pymunk.Segment(space.static_body, (440, 120), (440, 700), 6)
floor = pymunk.Segment(space.static_body, (80, 700), (440, 700), 6)
for seg in (left, right, floor):
    seg.elasticity = 0.15
    seg.friction = 0.7
space.add(left, right, floor)

FRUIT_COLLISION = 7
fruits = []
fruit_by_shape = {}
pending_merge = []
score = 0
danger_frames = 0
game_over = False
next_level = 0
drop_cooldown = 0


def spawn_fruit(x, level=0, y=140):
    radius = radii[level]
    mass = 1 + level * 0.6
    moment = pymunk.moment_for_circle(mass, 0, radius)
    body = pymunk.Body(mass, moment)
    body.position = (x, y)
    shape = pymunk.Circle(body, radius)
    shape.friction = 0.55
    shape.elasticity = 0.2
    shape.collision_type = FRUIT_COLLISION
    space.add(body, shape)
    fruit = {"body": body, "shape": shape, "level": level}
    fruits.append(fruit)
    fruit_by_shape[shape] = fruit
    return fruit


def remove_fruit(fruit):
    if fruit in fruits:
        fruits.remove(fruit)
    fruit_by_shape.pop(fruit["shape"], None)
    if fruit["shape"] in space.shapes and fruit["body"] in space.bodies:
        space.remove(fruit["shape"], fruit["body"])


def on_collision(arbiter, space_, data):
    s1, s2 = arbiter.shapes
    f1 = fruit_by_shape.get(s1)
    f2 = fruit_by_shape.get(s2)
    if not f1 or not f2:
        return True
    if f1["level"] != f2["level"] or f1["level"] >= 4:
        return True
    mx = (f1["body"].position.x + f2["body"].position.x) * 0.5
    my = (f1["body"].position.y + f2["body"].position.y) * 0.5
    pending_merge.append((f1, f2, int(mx), int(my), f1["level"] + 1))
    return True


def apply_merges():
    global score
    while pending_merge:
        f1, f2, mx, my, next_lv = pending_merge.pop(0)
        if f1 not in fruits or f2 not in fruits:
            continue
        remove_fruit(f1)
        remove_fruit(f2)
        newborn = spawn_fruit(mx, next_lv, my)
        newborn["body"].velocity = (newborn["body"].velocity.x * 0.85, newborn["body"].velocity.y * 0.85)
        score += [0, 10, 30, 80, 200, 500][next_lv]


def draw_preview(surface, level):
    radius = radii[level]
    pygame.draw.circle(surface, (187, 247, 208), (470, 76), radius)


def reset_game():
    global score, game_over, danger_frames, next_level, drop_cooldown
    for fruit in fruits[:]:
        remove_fruit(fruit)
    pending_merge.clear()
    score = 0
    game_over = False
    danger_frames = 0
    next_level = 0
    drop_cooldown = 0


handler = space.add_collision_handler(FRUIT_COLLISION, FRUIT_COLLISION)
handler.begin = on_collision

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
                reset_game()

        if event.type == pygame.MOUSEBUTTONDOWN and event.button == 1 and drop_cooldown == 0 and not game_over:
            mx, _ = pygame.mouse.get_pos()
            drop_x = max(96, min(424, mx))
            spawn_fruit(drop_x, next_level)
            next_level = min(2, random.randint(0, 2))
            drop_cooldown = 14

    if drop_cooldown > 0:
        drop_cooldown -= 1

    if not game_over:
        space.step(1 / 60)
        apply_merges()

        if any(item["body"].position.y < 150 for item in fruits):
            danger_frames += 1
        else:
            danger_frames = max(0, danger_frames - 2)

        if danger_frames > 180:
            game_over = True

    screen.fill((10, 35, 25))

    pygame.draw.line(screen, wall_color, (80, 120), (80, 700), 6)
    pygame.draw.line(screen, wall_color, (440, 120), (440, 700), 6)
    pygame.draw.line(screen, wall_color, (80, 700), (440, 700), 6)
    pygame.draw.line(screen, (251, 191, 36), (80, 150), (440, 150), 2)

    for item in fruits:
        pos = item["body"].position
        lv = item["level"]
        r = int(item["shape"].radius)
        color = fruit_colors[lv]
        pygame.draw.circle(screen, color, (int(pos.x), int(pos.y)), r)
        pygame.draw.circle(screen, (20, 35, 28), (int(pos.x), int(pos.y)), r, 2)

    hud = small.render(f"Score: {score}", True, (236, 253, 245))
    screen.blit(hud, (16, 14))
    sub = small.render("Next", True, (220, 252, 231))
    screen.blit(sub, (444, 24))
    draw_preview(screen, next_level)

    if game_over:
        msg = title.render("Game Over", True, (254, 226, 226))
        hint = small.render("Press R to Restart", True, (254, 202, 202))
        screen.blit(msg, (WIDTH // 2 - msg.get_width() // 2, 280))
        screen.blit(hint, (WIDTH // 2 - hint.get_width() // 2, 332))

    pygame.display.flip()
    clock.tick(60)
```
