## 개요

- 대상: `pygame` 기초를 끝내고 실제 게임 리소스(스프라이트/폰트)를 붙여보고 싶은 학습자
- 방식: class별로 `한 가지 핵심 기능`을 확실히 완성하고 다음 class에서 누적 확장
- 최종 산출물: 중력 점프, 파이프 충돌, 점수/베스트 점수 UI가 있는 플래피 버드 스타일 게임 1개
- 실행 환경: Python 3.10+ / `pip install pygame pymunk`

## 리소스 준비 체크

- 리소스 다운로드: [FlappyResource.zip](/docs/flappy-physics/resources/FlappyResource.zip)
- 다운로드한 `FlappyResource.zip` 압축을 푼 뒤 아래처럼 배치하면 바로 실행 가능하다.
- 예시:
  - `flappy_final.py`
  - `assets/FlappySprite.png`
  - `assets/FlappyFont.TTF`

## 목차

1. class 1. 스프라이트 시트 + TTF 폰트 로딩
2. class 2. 물리엔진으로 새 점프 구현
3. class 3. 물리 충돌 기반 파이프 장애물
4. class 4. 점수, 재시작, 게임 완성

---

## class 1. 스프라이트 시트 + TTF 폰트 로딩

### 목표

- `FlappySprite.png`에서 필요한 영역을 잘라 스프라이트로 사용한다.
- `FlappyFont.TTF`를 로드해 점수 텍스트를 그린다.
- 배경, 바닥, 새 애니메이션이 움직이는 기본 화면을 만든다.

### 핵심 변수/함수

- `ASSET_DIR`: 리소스 폴더 경로
- `SPRITES`: 스프라이트 좌표 맵
- `cut(rect)`: 스프라이트 시트에서 이미지 자르기
- `font_big`: TTF 폰트 렌더링 객체

### 단계별 구현

#### 단계 1) 리소스 경로와 스프라이트 좌표 정의

##### 세부목표
  - 리소스 파일 경로를 코드에서 고정하지 않고 폴더 기준으로 관리한다.
  - 스프라이트 시트에서 필요한 좌표를 딕셔너리로 관리해 재사용성을 높인다.

```python
from pathlib import Path
import pygame

pygame.init()
WIDTH, HEIGHT = 480, 720
screen = pygame.display.set_mode((WIDTH, HEIGHT))
clock = pygame.time.Clock()

ASSET_DIR = Path(__file__).resolve().parent / "assets"
SPRITE_PATH = ASSET_DIR / "FlappySprite.png"
FONT_PATH = ASSET_DIR / "FlappyFont.TTF"

sprite_sheet = pygame.image.load(str(SPRITE_PATH)).convert_alpha()

SPRITES = {
    "bg": (0, 512, 288, 512),
    "ground": (0, 399, 307, 112),
    "bird_up": (421, 975, 34, 24),
    "bird_mid": (421, 1000, 34, 24),
    "bird_down": (456, 1000, 34, 24),
}
```

##### 선언한 변수/함수의 목적
  - `ASSET_DIR`: 코드 파일 옆 `assets` 폴더를 찾는 기준 경로
  - `SPRITE_PATH`, `FONT_PATH`: 이미지/폰트 파일 위치
  - `SPRITES`: 스프라이트 이름과 `(x, y, w, h)` 좌표 묶음

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `ASSET_DIR = Path(__file__).resolve().parent / "assets"`: 실행 위치가 바뀌어도 코드 파일 기준으로 리소스를 찾게 만들어 경로 오류를 줄인다.
  - `pygame.image.load(...).convert_alpha()`: 투명 채널(RGBA)을 유지한 상태로 로드해 새 스프라이트 가장자리가 깨지지 않게 렌더링한다.
  - `SPRITES = {...}`: 스프라이트 잘라내기 좌표를 하드코딩 함수 호출마다 반복하지 않고 이름 기반으로 일관되게 참조한다.
  - `"bird_up"`, `"bird_mid"`, `"bird_down"`: 날개 프레임을 분리해 이후 애니메이션 인덱스로 바꿔 그릴 수 있게 준비한다.

#### 단계 2) 스프라이트 자르기와 애니메이션 프레임 만들기

##### 세부목표
  - 공통 `cut()` 함수로 스프라이트를 잘라 재사용한다.
  - 새 프레임 배열을 만들고 시간 기반 인덱스로 애니메이션을 만든다.

```python
def cut(rect):
    x, y, w, h = rect
    return sprite_sheet.subsurface(pygame.Rect(x, y, w, h)).copy()

bg_img = pygame.transform.scale(cut(SPRITES["bg"]), (WIDTH, HEIGHT))
ground_img = cut(SPRITES["ground"])
bird_frames = [
    cut(SPRITES["bird_up"]),
    cut(SPRITES["bird_mid"]),
    cut(SPRITES["bird_down"]),
]
```

##### 선언한 변수/함수의 목적
  - `cut(rect)`: 좌표를 받아 단일 이미지로 잘라내는 유틸 함수
  - `bird_frames`: 새 날개 애니메이션 프레임 목록
  - `bg_img`, `ground_img`: 배경/바닥 렌더링용 Surface

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `sprite_sheet.subsurface(...).copy()`: 원본 시트 일부만 참조하지 않고 복사본을 만들어 이후 변형(회전/스케일)에 안전하게 사용한다.
  - `pygame.transform.scale(...)`: 원본 배경 크기를 게임 해상도에 맞춰 확대해 화면 빈 공간 없이 꽉 채운다.
  - `bird_frames = [...]`: 프레임 순서를 코드에서 고정해 인덱스만 바꾸면 안정적으로 같은 애니메이션 패턴을 반복한다.
  - `cut(SPRITES[...])` 방식: 좌표 숫자를 직접 반복 입력하지 않아 유지보수 시 좌표 수정 범위를 한 곳으로 줄인다.

#### 단계 3) TTF 폰트 로드와 기본 HUD 출력

##### 세부목표
  - TTF 폰트를 로드해 점수 문자열을 렌더링한다.
  - 새 프레임과 점수 텍스트를 같은 루프에서 갱신한다.

```python
font_big = pygame.font.Font(str(FONT_PATH), 54)
font_small = pygame.font.Font(str(FONT_PATH), 24)

frame_timer = 0.0
bird_idx = 0
score_preview = 0
running = True

while running:
    dt = clock.tick(60) / 1000.0
    frame_timer += dt
    if frame_timer >= 0.12:
        frame_timer = 0.0
        bird_idx = (bird_idx + 1) % len(bird_frames)

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    screen.blit(bg_img, (0, 0))
    screen.blit(ground_img, (0, HEIGHT - ground_img.get_height()))
    screen.blit(bird_frames[bird_idx], (160, 260))

    score_surf = font_big.render(str(score_preview), True, (255, 255, 255))
    title_surf = font_small.render("Flappy Physics", True, (255, 245, 190))
    screen.blit(score_surf, (WIDTH // 2 - score_surf.get_width() // 2, 54))
    screen.blit(title_surf, (16, 16))

    pygame.display.flip()

pygame.quit()
```

##### 선언한 변수/함수의 목적
  - `font_big`, `font_small`: 서로 다른 크기의 TTF 렌더링 객체
  - `frame_timer`: 애니메이션 프레임 전환 타이머
  - `bird_idx`: 현재 새 프레임 인덱스

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `pygame.font.Font(str(FONT_PATH), 54)`: 기본 시스템 폰트가 아니라 게임 전용 TTF를 써서 UI 톤을 리소스와 맞춘다.
  - `if frame_timer >= 0.12`: FPS가 달라도 0.12초마다 프레임이 바뀌도록 시간 기반 애니메이션 속도를 고정한다.
  - `font_big.render(str(score_preview), True, (255, 255, 255))`: 숫자를 Surface로 변환해 스프라이트와 같은 방식으로 화면에 배치한다.
  - `screen.blit(..., (WIDTH // 2 - ...))`: 텍스트 폭을 반영해 가운데 정렬을 계산하므로 점수 자릿수가 바뀌어도 중앙에 유지된다.

#### class 1 최종 코드

```python
# class 1 최종 파일 예시: flappy_class1.py
# 실행 전: 같은 폴더에 assets/FlappySprite.png, assets/FlappyFont.TTF 배치

from pathlib import Path
import pygame

pygame.init()
WIDTH, HEIGHT = 480, 720
screen = pygame.display.set_mode((WIDTH, HEIGHT))
clock = pygame.time.Clock()

ASSET_DIR = Path(__file__).resolve().parent / "assets"
sprite_sheet = pygame.image.load(str(ASSET_DIR / "FlappySprite.png")).convert_alpha()
font_big = pygame.font.Font(str(ASSET_DIR / "FlappyFont.TTF"), 54)
font_small = pygame.font.Font(str(ASSET_DIR / "FlappyFont.TTF"), 24)

SPRITES = {
    "bg": (0, 512, 288, 512),
    "ground": (0, 399, 307, 112),
    "bird_up": (421, 975, 34, 24),
    "bird_mid": (421, 1000, 34, 24),
    "bird_down": (456, 1000, 34, 24),
}


def cut(rect):
    x, y, w, h = rect
    return sprite_sheet.subsurface(pygame.Rect(x, y, w, h)).copy()


bg_img = pygame.transform.scale(cut(SPRITES["bg"]), (WIDTH, HEIGHT))
ground_img = cut(SPRITES["ground"])
bird_frames = [cut(SPRITES["bird_up"]), cut(SPRITES["bird_mid"]), cut(SPRITES["bird_down"])]

frame_timer = 0.0
bird_idx = 0
running = True
while running:
    dt = clock.tick(60) / 1000.0
    frame_timer += dt
    if frame_timer >= 0.12:
        frame_timer = 0.0
        bird_idx = (bird_idx + 1) % len(bird_frames)

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    screen.blit(bg_img, (0, 0))
    screen.blit(ground_img, (0, HEIGHT - ground_img.get_height()))
    screen.blit(bird_frames[bird_idx], (160, 260))

    score_surf = font_big.render("0", True, (255, 255, 255))
    label_surf = font_small.render("Class 1 - Asset Load", True, (255, 245, 190))
    screen.blit(score_surf, (WIDTH // 2 - score_surf.get_width() // 2, 54))
    screen.blit(label_surf, (16, 16))

    pygame.display.flip()

pygame.quit()
```

---

## class 2. 물리엔진으로 새 점프 구현

### 목표

- `pymunk.Space` 중력 월드를 만들고 새를 동적 바디로 등록한다.
- 클릭 입력으로 새의 속도를 순간 변경해 점프를 만든다.
- 물리 위치와 스프라이트 렌더링 위치를 동기화한다.

### 핵심 변수/함수

- `space`: 물리 시뮬레이션 월드
- `bird_body`, `bird_shape`: 새 물리 바디/충돌체
- `flap()`: 클릭 시 점프 속도 적용 함수
- `space.step(step_dt)`: 고정 시간 물리 업데이트

### 단계별 구현

#### 단계 1) 물리 월드와 바닥 충돌체 추가

##### 세부목표
  - 새가 중력으로 떨어지도록 월드를 만든다.
  - 바닥 세그먼트를 추가해 화면 밖으로 무한 낙하하지 않게 막는다.

```python
import pymunk

space = pymunk.Space()
space.gravity = (0, 1750)

floor_y = HEIGHT - ground_img.get_height() + 8
floor = pymunk.Segment(space.static_body, (0, floor_y), (WIDTH, floor_y), 4)
floor.friction = 0.9
floor.elasticity = 0.0
space.add(floor)
```

##### 선언한 변수/함수의 목적
  - `space`: 물리 계산을 담당하는 공간
  - `floor_y`: 바닥 충돌선 y좌표
  - `floor`: 고정 바닥 충돌체

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `space = pymunk.Space()`: 충돌/중력/속도 갱신을 전담하는 엔진 인스턴스를 생성한다.
  - `space.gravity = (0, 1750)`: y축 가속도를 크게 설정해 플래피 버드 특유의 빠른 낙하 감각을 만든다.
  - `pymunk.Segment(space.static_body, ...)`: 움직이지 않는 고정 벽/바닥을 간단한 선분으로 만들어 성능과 안정성을 확보한다.
  - `space.add(floor)`: 충돌체를 월드에 등록해야만 `space.step()` 호출 때 실제 충돌 계산에 포함된다.

#### 단계 2) 새 동적 바디 생성과 점프 함수

##### 세부목표
  - 새를 물리 바디로 만들어 중력 영향을 받게 한다.
  - 클릭 시 위쪽 속도를 강제로 넣어 점프를 구현한다.

```python
bird_mass = 1.0
bird_radius = 12
bird_moment = pymunk.moment_for_circle(bird_mass, 0, bird_radius)

bird_body = pymunk.Body(bird_mass, bird_moment)
bird_body.position = (WIDTH * 0.32, HEIGHT * 0.45)
bird_shape = pymunk.Circle(bird_body, bird_radius)
bird_shape.friction = 0.4
bird_shape.elasticity = 0.0
space.add(bird_body, bird_shape)


def flap():
    bird_body.velocity = (0, -420)
```

##### 선언한 변수/함수의 목적
  - `bird_mass`, `bird_moment`: 새의 질량/관성 값
  - `bird_body`: 위치와 속도를 가진 동적 바디
  - `flap()`: 점프 속도 주입 함수

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `pymunk.moment_for_circle(...)`: 원형 바디 관성을 계산해 물리 엔진이 회전/충돌을 안정적으로 처리할 수 있게 한다.
  - `bird_body.position = (...)`: 새 시작 위치를 정해 게임 시작 직후 플레이어가 제어 가능한 높이에 배치한다.
  - `space.add(bird_body, bird_shape)`: 바디와 충돌체를 함께 등록해야 물리 이동과 충돌 판정이 동시에 작동한다.
  - `bird_body.velocity = (0, -420)`: 클릭 순간 현재 낙하 속도를 덮어써 위로 반동을 주기 때문에 탭 점프 감각이 명확해진다.

#### 단계 3) 물리 스텝과 렌더링 동기화

##### 세부목표
  - 프레임마다 고정 시간 간격으로 물리를 진행한다.
  - 물리 좌표를 스프라이트 좌표로 변환해 새를 그린다.

```python
step_dt = 1 / 60

for event in pygame.event.get():
    if event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
        flap()

space.step(step_dt)

bird_frame = bird_frames[bird_idx]
bx = int(bird_body.position.x - bird_frame.get_width() // 2)
by = int(bird_body.position.y - bird_frame.get_height() // 2)
screen.blit(bird_frame, (bx, by))
```

##### 선언한 변수/함수의 목적
  - `step_dt`: 물리 업데이트 간격
  - `bx`, `by`: 새 스프라이트 좌상단 좌표

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `step_dt = 1 / 60`: 물리 스텝을 고정해 PC 성능이 달라도 비슷한 점프 궤적을 유지한다.
  - `if event.type == pygame.MOUSEBUTTONDOWN ...`: 입력 이벤트를 직접 점프 함수와 연결해 조작 반응 지연을 줄인다.
  - `space.step(step_dt)`: 중력과 속도 적분을 진행해 새 위치를 매 프레임 갱신한다.
  - `bird_body.position` 기반 렌더링: 눈에 보이는 위치를 물리 상태에서 읽으므로 충돌체와 그래픽이 따로 놀지 않는다.

#### class 2 최종 코드

```python
# class 2는 class 1 코드에 pymunk 초기화와 bird_body/flap/space.step을 합친 형태
# 핵심: 점프를 좌표 직접 변경이 아니라 velocity 변경으로 구현한다.
```

---

## class 3. 물리 충돌 기반 파이프 장애물

### 목표

- 파이프를 스프라이트로 렌더링하고 충돌체를 물리 월드에 등록한다.
- 새와 파이프가 부딪히면 게임오버 상태로 전환한다.
- 파이프를 일정 속도로 왼쪽으로 이동시켜 장애물 루프를 만든다.

### 핵심 변수/함수

- `pipes`: 파이프 데이터 목록
- `spawn_pipe(x)`: 파이프 1세트 생성
- `PIPE_COLLISION`: 파이프 충돌 타입 ID
- `space.on_collision(...)`: 충돌 콜백 등록

### 단계별 구현

#### 단계 1) 파이프 스프라이트와 렌더 데이터 준비

##### 세부목표
  - 파이프 본체 스프라이트를 잘라 상/하 파이프 모두 그릴 수 있게 준비한다.
  - 파이프 갭 범위를 정해 게임 난이도를 제어한다.

```python
import random

pipe_img = cut((289, 754, 52, 270))
pipe_img_top = pygame.transform.flip(pipe_img, False, True)

PIPE_SPEED = 150
PIPE_GAP = 180
PIPE_INTERVAL = 1.55
pipe_timer = 0.0
pipes = []
```

##### 선언한 변수/함수의 목적
  - `pipe_img`, `pipe_img_top`: 하단/상단 파이프 렌더 이미지
  - `PIPE_GAP`: 위아래 파이프 사이 통과 간격
  - `pipes`: 현재 파이프 상태 목록

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `cut((289, 754, 52, 270))`: 스프라이트 시트에서 파이프 본체를 정확히 잘라 장애물 그래픽으로 재사용한다.
  - `pygame.transform.flip(..., False, True)`: 별도 상단 이미지가 없어도 세로 반전으로 위 파이프를 만들 수 있다.
  - `PIPE_GAP = 180`: 통과 가능 영역 높이를 상수화해 난이도 조정 시 한 값만 바꾸면 되게 만든다.
  - `pipes = []`: 렌더링, 이동, 충돌체 제거를 같은 리스트 루프로 처리하기 위한 상태 컨테이너다.

#### 단계 2) 파이프 충돌체 생성과 콜백 등록

##### 세부목표
  - 파이프마다 상단/하단 충돌체를 만들어 물리 월드에 넣는다.
  - 새와 파이프가 충돌하면 즉시 게임오버 플래그를 세운다.

```python
BIRD_COLLISION = 1
PIPE_COLLISION = 2
bird_shape.collision_type = BIRD_COLLISION


def make_box(body, cx, cy, w, h):
    hw, hh = w / 2, h / 2
    verts = [(cx - hw, cy - hh), (cx + hw, cy - hh), (cx + hw, cy + hh), (cx - hw, cy + hh)]
    return pymunk.Poly(body, verts)


def spawn_pipe(x_pos):
    gap_y = random.randint(220, HEIGHT - 250)
    body = pymunk.Body(body_type=pymunk.Body.KINEMATIC)
    body.position = (x_pos, 0)

    upper_h = gap_y - PIPE_GAP // 2
    lower_y = gap_y + PIPE_GAP // 2
    lower_h = floor_y - lower_y

    upper_shape = make_box(body, 26, upper_h * 0.5, 52, upper_h)
    lower_shape = make_box(body, 26, lower_y + lower_h * 0.5, 52, lower_h)
    upper_shape.collision_type = PIPE_COLLISION
    lower_shape.collision_type = PIPE_COLLISION

    space.add(body, upper_shape, lower_shape)
    pipes.append({"x": x_pos, "gap_y": gap_y, "body": body, "upper": upper_shape, "lower": lower_shape, "scored": False})


nonlocal_game = {"game_over": False}


def on_hit_pipe(arbiter, space_, data):
    nonlocal_game["game_over"] = True
    return True


space.on_collision(BIRD_COLLISION, PIPE_COLLISION, begin=on_hit_pipe)
```

##### 선언한 변수/함수의 목적
  - `make_box()`: 파이프용 사각형 충돌체 생성 함수
  - `spawn_pipe()`: 파이프 세트 생성 함수
  - `nonlocal_game`: 콜백에서 수정할 게임 상태 컨테이너

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `body = pymunk.Body(body_type=pymunk.Body.KINEMATIC)`: 파이프를 외부 코드로 이동시키되 충돌 계산은 정상 동작하게 만든다.
  - `pymunk.Poly(body, verts)`: `create_box()` 대신 꼭짓점 기반 폴리를 써서 상단/하단 박스를 서로 다른 오프셋 위치에 배치한다.
  - `upper_shape.collision_type = PIPE_COLLISION`: 새와 파이프 충돌 조합만 별도로 콜백 처리할 수 있게 분류한다.
  - `space.on_collision(..., begin=on_hit_pipe)`: 접촉 시작 순간 콜백을 실행해 게임오버 반응을 프레임 지연 없이 처리한다.

#### 단계 3) 파이프 이동, 제거, 렌더링

##### 세부목표
  - 일정 간격으로 새 파이프를 생성한다.
  - 화면 밖으로 나간 파이프와 충돌체를 정리한다.

```python
pipe_timer += dt
if pipe_timer >= PIPE_INTERVAL:
    pipe_timer = 0.0
    spawn_pipe(WIDTH + 40)

for pipe in pipes:
    pipe["x"] -= PIPE_SPEED * dt
    pipe["body"].position = (pipe["x"], 0)

alive = []
for pipe in pipes:
    if pipe["x"] + pipe_img.get_width() > -20:
        alive.append(pipe)
    else:
        space.remove(pipe["upper"], pipe["lower"], pipe["body"])
pipes = alive

for pipe in pipes:
    top_y = pipe["gap_y"] - PIPE_GAP // 2 - pipe_img.get_height()
    bottom_y = pipe["gap_y"] + PIPE_GAP // 2
    screen.blit(pipe_img_top, (int(pipe["x"]), int(top_y)))
    screen.blit(pipe_img, (int(pipe["x"]), int(bottom_y)))
```

##### 선언한 변수/함수의 목적
  - `pipe_timer`: 파이프 생성 타이머
  - `alive`: 화면에 남길 파이프 임시 목록
  - `pipe["body"].position`: 렌더 x와 물리 충돌체 위치 동기화

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `spawn_pipe(WIDTH + 40)`: 화면 오른쪽 바깥에서 생성해 자연스럽게 슬라이드 인되게 만든다.
  - `pipe["body"].position = (pipe["x"], 0)`: 충돌체를 매 프레임 같은 x로 이동시켜 스프라이트와 충돌 위치가 어긋나지 않게 한다.
  - `space.remove(...)`: 화면에서 사라진 파이프 충돌체를 즉시 제거해 보이지 않는 충돌과 성능 누수를 막는다.
  - `screen.blit(pipe_img_top...)`: 동일 스프라이트를 반전해 상단/하단 파이프를 모두 렌더링해 리소스 사용량을 줄인다.

#### class 3 최종 코드

```python
# class 3은 class 2에 파이프 생성/이동/충돌 콜백을 통합한 상태
# 핵심: 파이프는 kinematic body로 이동시키고, bird는 dynamic body로 중력/점프를 유지한다.
```

---

## class 4. 점수, 재시작, 게임 완성

### 목표

- 파이프 통과 시 점수를 올리고 TTF 폰트로 HUD를 갱신한다.
- 게임오버 화면과 재시작 입력을 추가한다.
- 클래스 1~3 요소를 하나의 완성 게임 루프로 정리한다.

### 핵심 변수/함수

- `score`, `best_score`: 현재 점수/최고 점수
- `reset_game()`: 물리 객체와 게임 상태 초기화
- `game_state`: `"ready"`, `"play"`, `"over"` 상태 문자열

### 단계별 구현

#### 단계 1) 점수 조건과 HUD 업데이트

##### 세부목표
  - 파이프를 통과한 시점을 감지해 점수를 1씩 증가시킨다.
  - 점수 텍스트를 매 프레임 렌더링해 즉시 반영한다.

```python
score = 0
best_score = 0

for pipe in pipes:
    if (not pipe["scored"]) and (pipe["x"] + 52 < bird_body.position.x):
        pipe["scored"] = True
        score += 1

score_surf = font_big.render(str(score), True, (255, 255, 255))
best_surf = font_small.render(f"BEST {best_score}", True, (255, 245, 190))
screen.blit(score_surf, (WIDTH // 2 - score_surf.get_width() // 2, 52))
screen.blit(best_surf, (16, 16))
```

##### 선언한 변수/함수의 목적
  - `score`: 현재 라운드 점수
  - `best_score`: 세션 내 최고 기록
  - `pipe["scored"]`: 중복 득점 방지 플래그

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `pipe["x"] + 52 < bird_body.position.x`: 파이프 오른쪽 끝이 새 중심보다 왼쪽으로 지나간 순간을 통과 이벤트로 정의한다.
  - `pipe["scored"] = True`: 이미 점수 처리한 파이프를 표시해 한 파이프에서 점수가 여러 번 오르는 버그를 막는다.
  - `font_big.render(str(score), ...)`: 점수 숫자를 매 프레임 새로 렌더링해 즉시 HUD에 반영한다.
  - `screen.blit(best_surf, (16, 16))`: 베스트 점수를 상단 고정 위치에 표시해 학습자가 상태 변화를 빠르게 확인할 수 있다.

#### 단계 2) 게임오버와 재시작 상태 머신

##### 세부목표
  - 충돌 시 `over` 상태로 전환하고 입력을 잠근다.
  - 스페이스 키로 현재 라운드를 초기화한다.

```python
game_state = "ready"


def reset_game():
    global score, pipe_timer, pipes
    score = 0
    pipe_timer = 0.0
    for pipe in pipes:
        space.remove(pipe["upper"], pipe["lower"], pipe["body"])
    pipes = []
    bird_body.position = (WIDTH * 0.32, HEIGHT * 0.45)
    bird_body.velocity = (0, 0)

if nonlocal_game["game_over"]:
    game_state = "over"
    best_score = max(best_score, score)

if event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE and game_state == "over":
    nonlocal_game["game_over"] = False
    game_state = "ready"
    reset_game()
```

##### 선언한 변수/함수의 목적
  - `game_state`: 입력/업데이트 분기 기준 상태값
  - `reset_game()`: 라운드 재시작용 초기화 루틴
  - `nonlocal_game["game_over"]`: 충돌 콜백 결과 공유 값

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `game_state = "over"`: 게임 상태를 명시적으로 바꿔 이동/입력/생성 로직을 한 번에 제어할 수 있게 한다.
  - `best_score = max(best_score, score)`: 라운드 종료 시점에만 최고 점수를 갱신해 계산 타이밍을 단순화한다.
  - `space.remove(pipe[...])`: 재시작 때 이전 라운드 충돌체를 모두 지워 보이지 않는 파이프 충돌을 제거한다.
  - `bird_body.velocity = (0, 0)`: 낙하 속도를 초기화해 재시작 직후 즉시 바닥으로 추락하는 현상을 막는다.

#### 단계 3) 준비/플레이/오버 화면 메시지 마무리

##### 세부목표
  - 상태별 안내 문구를 TTF로 렌더링해 학습 UX를 개선한다.
  - 완성 루프에서 `ready -> play -> over` 전환을 분명히 만든다.

```python
if game_state == "ready":
    msg = font_small.render("CLICK TO START", True, (255, 245, 190))
    screen.blit(msg, (WIDTH // 2 - msg.get_width() // 2, HEIGHT // 2 - 40))

if game_state == "over":
    over = font_big.render("GAME OVER", True, (255, 232, 140))
    restart = font_small.render("PRESS SPACE TO RESTART", True, (255, 245, 190))
    screen.blit(over, (WIDTH // 2 - over.get_width() // 2, HEIGHT // 2 - 68))
    screen.blit(restart, (WIDTH // 2 - restart.get_width() // 2, HEIGHT // 2 + 8))
```

##### 선언한 변수/함수의 목적
  - `msg`, `over`, `restart`: 상태 안내 텍스트 Surface

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if game_state == "ready"`: 시작 전 상태에서만 안내 문구를 보여 입력 규칙을 사용자에게 즉시 전달한다.
  - `font_big.render("GAME OVER", ...)`: 종료 순간 큰 타이틀을 띄워 상태 변화가 시각적으로 명확해진다.
  - `PRESS SPACE TO RESTART` 문구: 재시작 키를 화면에 직접 제시해 사용자가 도움말 없이 다음 행동을 수행하게 한다.
  - `WIDTH // 2 - surf.get_width() // 2`: 안내 문구를 모두 중앙 정렬해 해상도가 달라도 UI 배치 균형을 유지한다.

#### class 4 최종 코드

```python
# class 4 최종 파일 예시: flappy_final.py
# 준비: 같은 폴더에 assets/FlappySprite.png, assets/FlappyFont.TTF

from pathlib import Path
import random
import pygame
import pymunk

pygame.init()
WIDTH, HEIGHT = 480, 720
screen = pygame.display.set_mode((WIDTH, HEIGHT))
clock = pygame.time.Clock()

ASSET_DIR = Path(__file__).resolve().parent / "assets"
sprite_sheet = pygame.image.load(str(ASSET_DIR / "FlappySprite.png")).convert_alpha()
font_big = pygame.font.Font(str(ASSET_DIR / "FlappyFont.TTF"), 48)
font_small = pygame.font.Font(str(ASSET_DIR / "FlappyFont.TTF"), 24)

SPRITES = {
    "bg": (0, 512, 288, 512),
    "ground": (0, 399, 307, 112),
    "bird_up": (421, 975, 34, 24),
    "bird_mid": (421, 1000, 34, 24),
    "bird_down": (456, 1000, 34, 24),
    "pipe": (289, 754, 52, 270),
}


def cut(rect):
    x, y, w, h = rect
    return sprite_sheet.subsurface(pygame.Rect(x, y, w, h)).copy()


bg_img = pygame.transform.scale(cut(SPRITES["bg"]), (WIDTH, HEIGHT))
ground_img = cut(SPRITES["ground"])
pipe_img = cut(SPRITES["pipe"])
pipe_img_top = pygame.transform.flip(pipe_img, False, True)
bird_frames = [cut(SPRITES["bird_up"]), cut(SPRITES["bird_mid"]), cut(SPRITES["bird_down"])]

space = pymunk.Space()
space.gravity = (0, 1750)

floor_y = HEIGHT - ground_img.get_height() + 8
floor = pymunk.Segment(space.static_body, (0, floor_y), (WIDTH, floor_y), 4)
space.add(floor)

bird_mass = 1.0
bird_radius = 12
bird_moment = pymunk.moment_for_circle(bird_mass, 0, bird_radius)
bird_body = pymunk.Body(bird_mass, bird_moment)
bird_body.position = (WIDTH * 0.32, HEIGHT * 0.45)
bird_shape = pymunk.Circle(bird_body, bird_radius)
space.add(bird_body, bird_shape)

BIRD_COLLISION = 1
PIPE_COLLISION = 2
bird_shape.collision_type = BIRD_COLLISION

PIPE_SPEED = 150
PIPE_GAP = 180
PIPE_INTERVAL = 1.55

pipes = []
pipe_timer = 0.0
frame_timer = 0.0
bird_idx = 0
score = 0
best_score = 0
state = "ready"
flags = {"game_over": False}


def flap():
    bird_body.velocity = (0, -420)


def make_box(body, cx, cy, w, h):
    hw, hh = w / 2, h / 2
    verts = [(cx - hw, cy - hh), (cx + hw, cy - hh), (cx + hw, cy + hh), (cx - hw, cy + hh)]
    return pymunk.Poly(body, verts)


def spawn_pipe(x_pos):
    gap_y = random.randint(220, HEIGHT - 250)
    body = pymunk.Body(body_type=pymunk.Body.KINEMATIC)
    body.position = (x_pos, 0)

    upper_h = gap_y - PIPE_GAP // 2
    lower_y = gap_y + PIPE_GAP // 2
    lower_h = floor_y - lower_y

    upper = make_box(body, 26, upper_h * 0.5, 52, upper_h)
    lower = make_box(body, 26, lower_y + lower_h * 0.5, 52, lower_h)
    upper.collision_type = PIPE_COLLISION
    lower.collision_type = PIPE_COLLISION

    space.add(body, upper, lower)
    pipes.append({"x": x_pos, "gap_y": gap_y, "body": body, "upper": upper, "lower": lower, "scored": False})


def reset_game():
    global pipes, pipe_timer, score, state
    score = 0
    pipe_timer = 0.0
    for p in pipes:
        space.remove(p["upper"], p["lower"], p["body"])
    pipes = []
    bird_body.position = (WIDTH * 0.32, HEIGHT * 0.45)
    bird_body.velocity = (0, 0)
    flags["game_over"] = False
    state = "ready"


def on_hit_pipe(arbiter, space_, data):
    flags["game_over"] = True
    return True


space.on_collision(BIRD_COLLISION, PIPE_COLLISION, begin=on_hit_pipe)

running = True
while running:
    dt = clock.tick(60) / 1000.0

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        if event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
            if state in ("ready", "play"):
                state = "play"
                flap()
        if event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE and state == "over":
            reset_game()

    frame_timer += dt
    if frame_timer >= 0.11:
        frame_timer = 0.0
        bird_idx = (bird_idx + 1) % len(bird_frames)

    if state == "play":
        pipe_timer += dt
        if pipe_timer >= PIPE_INTERVAL:
            pipe_timer = 0.0
            spawn_pipe(WIDTH + 40)

        for p in pipes:
            p["x"] -= PIPE_SPEED * dt
            p["body"].position = (p["x"], 0)
            if (not p["scored"]) and (p["x"] + 52 < bird_body.position.x):
                p["scored"] = True
                score += 1

        alive = []
        for p in pipes:
            if p["x"] + pipe_img.get_width() > -20:
                alive.append(p)
            else:
                space.remove(p["upper"], p["lower"], p["body"])
        pipes = alive

        space.step(1 / 60)

    if flags["game_over"] and state != "over":
        state = "over"
        best_score = max(best_score, score)

    screen.blit(bg_img, (0, 0))

    for p in pipes:
        top_y = p["gap_y"] - PIPE_GAP // 2 - pipe_img.get_height()
        bottom_y = p["gap_y"] + PIPE_GAP // 2
        screen.blit(pipe_img_top, (int(p["x"]), int(top_y)))
        screen.blit(pipe_img, (int(p["x"]), int(bottom_y)))

    bird_img = bird_frames[bird_idx]
    bx = int(bird_body.position.x - bird_img.get_width() // 2)
    by = int(bird_body.position.y - bird_img.get_height() // 2)
    screen.blit(bird_img, (bx, by))

    screen.blit(ground_img, (0, HEIGHT - ground_img.get_height()))

    score_surf = font_big.render(str(score), True, (255, 255, 255))
    best_surf = font_small.render(f"BEST {best_score}", True, (255, 245, 190))
    screen.blit(score_surf, (WIDTH // 2 - score_surf.get_width() // 2, 52))
    screen.blit(best_surf, (16, 16))

    if state == "ready":
        msg = font_small.render("CLICK TO START", True, (255, 245, 190))
        screen.blit(msg, (WIDTH // 2 - msg.get_width() // 2, HEIGHT // 2 - 40))

    if state == "over":
        over = font_big.render("GAME OVER", True, (255, 232, 140))
        restart = font_small.render("PRESS SPACE TO RESTART", True, (255, 245, 190))
        screen.blit(over, (WIDTH // 2 - over.get_width() // 2, HEIGHT // 2 - 68))
        screen.blit(restart, (WIDTH // 2 - restart.get_width() // 2, HEIGHT // 2 + 8))

    pygame.display.flip()

pygame.quit()
```
