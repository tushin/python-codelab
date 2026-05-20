## 개요

- 대상: `pygame`으로 입력 이벤트를 다뤄본 뒤 사운드 인터랙션을 처음 붙이는 학습자
- 방식: class별로 "실행 코드 + 기능 1개 확장" 방식
- 최종 산출물: 버튼 점등 + 효과음 + 시퀀스 기억 판정이 가능한 Simon 게임 1개
- 실행 환경: Python 3.10+ / `pip install pygame`

## 목차

1. class 1. 버튼 보드 + 클릭 입력
2. class 2. 시퀀스 재생 + 사용자 입력 판정
3. class 3. 라운드 확장 + 실패 처리 + 재시작
4. class 4. 버튼 사운드 + UI 폴리싱

---

## class 1. 버튼 보드 + 클릭 입력

### 목표

- 네 개 버튼(빨강/초록/파랑/노랑) UI를 만든다.
- 마우스 클릭으로 어떤 버튼을 눌렀는지 판정한다.
- 눌린 버튼을 짧게 하이라이트한다.

### 핵심 변수/함수

- `pads`: 버튼 데이터 목록(`id`, `rect`, `base`, `light`)
- `active_pad`: 현재 하이라이트 대상 ID
- `active_until`: 하이라이트 종료 시각(ms)
- `pad_at()`: 클릭 좌표의 버튼 ID 탐색 함수

### 단계별 구현

#### 단계 1) 기본 창/버튼 데이터 만들기

##### 세부목표
  - 게임 창과 버튼 배치를 구성해 실행 가능한 기본 화면을 만든다.
  - 버튼 상태를 리스트 자료구조로 통일해 렌더링/입력에 재사용한다.

```python
import pygame
import sys

pygame.init()
WIDTH, HEIGHT = 640, 640
screen = pygame.display.set_mode((WIDTH, HEIGHT))
clock = pygame.time.Clock()

pads = [
    {"id": 0, "rect": pygame.Rect(120, 120, 180, 180), "base": (185, 28, 28), "light": (248, 113, 113)},
    {"id": 1, "rect": pygame.Rect(340, 120, 180, 180), "base": (22, 163, 74), "light": (74, 222, 128)},
    {"id": 2, "rect": pygame.Rect(120, 340, 180, 180), "base": (37, 99, 235), "light": (96, 165, 250)},
    {"id": 3, "rect": pygame.Rect(340, 340, 180, 180), "base": (202, 138, 4), "light": (250, 204, 21)},
]
```

##### 선언한 변수/함수의 목적
  - `pads`: 버튼 UI/색상/ID 데이터 목록
  - `screen`: 화면 출력 대상 객체
  - `clock`: 프레임 속도 제어 객체

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `pads = [...]`: 버튼별 속성을 한 자료구조로 묶어 입력 판정과 렌더링에서 같은 데이터를 공유한다.
  - `"rect": pygame.Rect(...)`: 버튼 클릭 영역을 명확히 정의해 충돌 판정 기준을 고정한다.
  - `"base"`, `"light"`: 기본색/점등색을 분리해 상태 변화가 화면에 즉시 보이게 한다.
  - `WIDTH, HEIGHT = 640, 640`: 정사각형 캔버스로 네 버튼을 균형 있게 배치할 공간을 만든다.

#### 단계 2) 좌표 -> 버튼 ID 판정 함수

##### 세부목표
  - 클릭 좌표를 버튼 데이터와 매칭하는 함수를 만든다.
  - 이벤트 루프에서는 함수 결과만 사용해 가독성을 높인다.

```python
def pad_at(pos):
    for pad in pads:
        if pad["rect"].collidepoint(pos):
            return pad["id"]
    return None
```

##### 선언한 변수/함수의 목적
  - `pad_at()`: 클릭 좌표를 버튼 ID로 변환하는 함수
  - `pos`: 마우스 입력 좌표

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `for pad in pads:`: 버튼 목록 전체를 순회해 어떤 버튼 영역인지 검사한다.
  - `pad["rect"].collidepoint(pos)`: 좌표가 버튼 사각형 내부인지 정확히 판정한다.
  - `return pad["id"]`: 클릭 버튼을 ID로 반환해 이후 로직(시퀀스 비교)에서 단순 값으로 처리한다.
  - `return None`: 어느 버튼도 누르지 않은 경우를 명시해 오입력을 안전하게 무시한다.

#### 단계 3) 클릭 하이라이트 상태

##### 세부목표
  - 클릭한 버튼을 짧게 점등해 입력 피드백을 준다.
  - 시간 기반으로 자동 소등되게 만든다.

```python
active_pad = None
active_until = 0

if event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
    pid = pad_at(event.pos)
    if pid is not None:
        active_pad = pid
        active_until = pygame.time.get_ticks() + 220

if active_pad is not None and pygame.time.get_ticks() >= active_until:
    active_pad = None
```

##### 선언한 변수/함수의 목적
  - `active_pad`: 현재 점등된 버튼 ID
  - `active_until`: 점등 종료 시각
  - `pid`: 방금 클릭된 버튼 ID

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `pid = pad_at(event.pos)`: 클릭 좌표를 즉시 버튼 ID로 바꿔 입력 판정 로직을 단순화한다.
  - `active_until = pygame.time.get_ticks() + 220`: 현재 시각 기준 종료 시점을 저장해 점등 시간을 일정하게 유지한다.
  - `if active_pad is not None and ... >= active_until:`: 점등 시간이 끝났을 때만 소등해 깜빡임 과다를 막는다.
  - `active_pad = None`: 소등 상태를 명시해 다음 렌더링 프레임에서 기본색으로 복귀시킨다.

#### 단계 4) 버튼 렌더링 함수

##### 세부목표
  - 버튼 색상을 상태에 따라 렌더링한다.
  - 점등 여부가 화면에서 직관적으로 보이게 만든다.

```python
def draw_pads(surface):
    for pad in pads:
        color = pad["light"] if pad["id"] == active_pad else pad["base"]
        pygame.draw.rect(surface, color, pad["rect"], border_radius=18)
```

##### 선언한 변수/함수의 목적
  - `draw_pads()`: 버튼 렌더링 함수
  - `color`: 현재 버튼 출력 색상

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `color = ... if ... else ...`: 점등 상태일 때만 밝은 색으로 바꿔 입력 피드백을 즉시 전달한다.
  - `pygame.draw.rect(...)`: 사각 버튼을 실제 화면에 그려 클릭 가능한 영역을 명확히 보여준다.
  - `for pad in pads:`: 버튼 개수 변화가 있어도 같은 렌더링 규칙을 재사용할 수 있게 한다.
  - `border_radius=18`: 둥근 모서리를 적용해 버튼별 경계를 시각적으로 부드럽게 구분한다.

#### class 1 최종 코드

```python
import pygame
import sys

pygame.init()
WIDTH, HEIGHT = 640, 640
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Simon - Class 1")
clock = pygame.time.Clock()

pads = [
    {"id": 0, "rect": pygame.Rect(120, 120, 180, 180), "base": (185, 28, 28), "light": (248, 113, 113)},
    {"id": 1, "rect": pygame.Rect(340, 120, 180, 180), "base": (22, 163, 74), "light": (74, 222, 128)},
    {"id": 2, "rect": pygame.Rect(120, 340, 180, 180), "base": (37, 99, 235), "light": (96, 165, 250)},
    {"id": 3, "rect": pygame.Rect(340, 340, 180, 180), "base": (202, 138, 4), "light": (250, 204, 21)},
]

active_pad = None
active_until = 0


def pad_at(pos):
    for pad in pads:
        if pad["rect"].collidepoint(pos):
            return pad["id"]
    return None


def draw_pads(surface):
    for pad in pads:
        color = pad["light"] if pad["id"] == active_pad else pad["base"]
        pygame.draw.rect(surface, color, pad["rect"], border_radius=18)


while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        if event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
            pid = pad_at(event.pos)
            if pid is not None:
                active_pad = pid
                active_until = pygame.time.get_ticks() + 220

    if active_pad is not None and pygame.time.get_ticks() >= active_until:
        active_pad = None

    screen.fill((15, 23, 42))
    draw_pads(screen)
    pygame.display.flip()
    clock.tick(60)
```

---

## class 2. 시퀀스 재생 + 사용자 입력 판정

### 목표

- 게임이 시퀀스를 자동 재생한다.
- 사용자가 같은 순서로 버튼을 누르는지 판정한다.
- 실패/성공 상태를 구분하는 기본 구조를 만든다.

### 핵심 변수/함수

- `sequence`: 정답 시퀀스 목록
- `input_index`: 현재 맞춰야 할 위치
- `mode`: `show`/`input` 상태
- `play_step()`: 시퀀스 재생 한 단계 함수

### 단계별 구현

#### 단계 1) 시퀀스 상태값 추가

##### 세부목표
  - 재생 모드와 입력 모드를 나눠 관리한다.
  - 첫 라운드 시퀀스를 초기화한다.

```python
import random

sequence = [random.randint(0, 3)]
input_index = 0
mode = "show"
show_idx = 0
next_tick = pygame.time.get_ticks() + 400
```

##### 선언한 변수/함수의 목적
  - `sequence`: 정답 버튼 순서
  - `input_index`: 사용자 입력 위치
  - `mode`: 현재 게임 상태

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `sequence = [random.randint(0, 3)]`: 첫 라운드 정답 버튼 1개를 랜덤으로 생성한다.
  - `mode = "show"`: 시작 상태를 시퀀스 재생으로 두어 사용자가 먼저 정답을 듣고 보게 만든다.
  - `show_idx = 0`: 재생 중 현재 출력할 시퀀스 인덱스를 0부터 추적한다.
  - `next_tick = ... + 400`: 재생 타이밍을 절대 시각으로 관리해 프레임 속도 차이를 줄인다.

#### 단계 2) 시퀀스 자동 재생

##### 세부목표
  - 재생 모드에서 버튼을 순서대로 점등한다.
  - 재생이 끝나면 입력 모드로 전환한다.

```python
def play_step(now):
    global show_idx, mode, input_index, active_pad, active_until, next_tick
    if now < next_tick:
        return

    if show_idx < len(sequence):
        active_pad = sequence[show_idx]
        active_until = now + 220
        show_idx += 1
        next_tick = now + 420
    else:
        mode = "input"
        input_index = 0
```

##### 선언한 변수/함수의 목적
  - `play_step()`: 시퀀스 재생 제어 함수
  - `show_idx`: 현재 재생 위치
  - `next_tick`: 다음 재생 시각

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if now < next_tick: return`: 재생 타이밍 전에는 대기해 시퀀스 속도를 일정하게 유지한다.
  - `active_pad = sequence[show_idx]`: 현재 단계 정답 버튼을 점등해 사용자가 시각 패턴을 기억하게 만든다.
  - `show_idx += 1`: 다음 재생 단계로 인덱스를 이동해 순서가 한 칸씩 진행되게 한다.
  - `mode = "input"`: 모든 재생이 끝나면 입력 모드로 전환해 사용자 조작을 허용한다.

#### 단계 3) 사용자 입력 판정

##### 세부목표
  - 입력 모드에서 클릭 버튼과 정답을 비교한다.
  - 틀리면 실패, 끝까지 맞히면 성공 상태로 표시한다.

```python
status = "watch"

if mode == "input" and event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
    pid = pad_at(event.pos)
    if pid is not None:
        active_pad = pid
        active_until = pygame.time.get_ticks() + 180
        if pid == sequence[input_index]:
            input_index += 1
            if input_index == len(sequence):
                status = "clear"
        else:
            status = "fail"
```

##### 선언한 변수/함수의 목적
  - `status`: 현재 라운드 결과 상태
  - `pid`: 클릭된 버튼 ID

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if mode == "input" ...`: 입력 허용 상태에서만 클릭 판정을 수행해 재생 중 오입력을 막는다.
  - `if pid == sequence[input_index]:`: 현재 순서 정답과 비교해 맞춘 경우에만 인덱스를 진행시킨다.
  - `input_index += 1`: 정답 입력 성공 시 다음 비교 위치로 이동한다.
  - `status = "fail"`: 하나라도 틀리면 즉시 실패 상태로 전환해 라운드 종료 조건을 명확히 만든다.

#### 단계 4) 상태 텍스트 표시

##### 세부목표
  - 재생/입력/성공/실패 상태를 화면에 표시한다.
  - 사용자에게 다음 행동을 직관적으로 안내한다.

```python
label = {
    "watch": "Watch the pattern",
    "clear": "Great!",
    "fail": "Wrong!",
}.get(status, "Your turn")
text = font.render(label, True, (226, 232, 240))
screen.blit(text, (180, 34))
```

##### 선언한 변수/함수의 목적
  - `label`: 상태 안내 문구
  - `text`: 렌더링된 텍스트 Surface

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `label = {...}.get(...)`: 상태 코드를 사용자 문구로 변환해 UI 가독성을 높인다.
  - `font.render(label, ...)`: 현재 상태를 텍스트 Surface로 만들어 화면 출력 가능 형태로 바꾼다.
  - `screen.blit(text, ...)`: 상단 고정 위치에 안내 문구를 그려 플레이 시선을 한 곳에 모은다.
  - `"Your turn"` 기본값을 두어 재생 종료 후 입력 단계 안내가 자연스럽게 이어지게 한다.

#### class 2 최종 코드

```python
# class 1 코드에 sequence/mode/play_step()/입력 판정 로직을 통합해 실행
# 핵심: show 모드 재생 -> input 모드 비교 구조 완성
```

---

## class 3. 라운드 확장 + 실패 처리 + 재시작

### 목표

- 라운드 성공 시 시퀀스 길이를 1 늘린다.
- 실패 시 게임오버 상태로 전환한다.
- `R` 키로 새 게임을 시작한다.

### 핵심 변수/함수

- `round_no`: 현재 라운드 번호
- `game_over`: 종료 상태
- `start_next_round()`: 다음 라운드 진입 함수

### 단계별 구현

#### 단계 1) 다음 라운드 함수

##### 세부목표
  - 시퀀스에 새 버튼을 추가해 난이도를 올린다.
  - 재생 모드 초기값을 한 번에 설정한다.

```python
def start_next_round():
    global round_no, mode, show_idx, next_tick, status
    sequence.append(random.randint(0, 3))
    round_no += 1
    mode = "show"
    show_idx = 0
    next_tick = pygame.time.get_ticks() + 500
    status = "watch"
```

##### 선언한 변수/함수의 목적
  - `start_next_round()`: 라운드 확장 함수
  - `round_no`: 현재 라운드 수

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `sequence.append(random.randint(0, 3))`: 정답 시퀀스를 한 칸 늘려 기억 부담을 단계적으로 높인다.
  - `round_no += 1`: 라운드 번호를 갱신해 HUD에서 난이도 진행 상황을 표시할 수 있게 한다.
  - `mode = "show"`: 새 라운드는 항상 재생 모드부터 시작해 학습 규칙을 일관되게 유지한다.
  - `next_tick = ... + 500`: 라운드 전환 직후 짧은 대기 시간을 둬 사용자가 상태 변화를 인지하게 한다.

#### 단계 2) 성공/실패 처리 분기

##### 세부목표
  - 입력 완료 성공 시 다음 라운드로 이동한다.
  - 실패 시 전체 입력을 중단한다.

```python
game_over = False

if status == "clear":
    start_next_round()
elif status == "fail":
    game_over = True
    mode = "idle"
```

##### 선언한 변수/함수의 목적
  - `game_over`: 종료 상태 플래그
  - `mode = "idle"`: 입력/재생 중단 상태

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if status == "clear":`: 라운드 정답 입력이 끝난 경우에만 시퀀스를 확장한다.
  - `start_next_round()`: 성공 즉시 다음 난이도로 넘어가 게임 템포를 끊지 않는다.
  - `elif status == "fail":`: 실패 상태를 별도 분기로 처리해 성공 로직과 충돌하지 않게 한다.
  - `mode = "idle"`: 종료 시 입력/재생을 멈춰 추가 클릭으로 상태가 오염되는 것을 막는다.

#### 단계 3) 재시작 함수

##### 세부목표
  - 전체 상태를 초기 라운드로 되돌린다.
  - `R` 입력으로 언제든 새 판 시작이 가능하게 만든다.

```python
def reset_game():
    global sequence, input_index, mode, show_idx, next_tick, status, round_no, game_over
    sequence = [random.randint(0, 3)]
    input_index = 0
    mode = "show"
    show_idx = 0
    next_tick = pygame.time.get_ticks() + 500
    status = "watch"
    round_no = 1
    game_over = False
```

##### 선언한 변수/함수의 목적
  - `reset_game()`: 전체 상태 초기화 함수
  - `sequence`: 정답 시퀀스 목록

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `sequence = [random.randint(0, 3)]`: 새 게임의 첫 정답을 다시 생성해 이전 판 패턴 잔존을 제거한다.
  - `mode = "show"`: 재시작 후 즉시 재생 단계로 진입해 게임 규칙을 처음부터 다시 안내한다.
  - `round_no = 1`: 난이도 진행값을 초기화해 점수/표시 상태와 동기화한다.
  - `game_over = False`: 종료 플래그를 해제해 입력 루프를 다시 활성화한다.

#### 단계 4) 라운드/게임오버 HUD

##### 세부목표
  - 라운드 번호를 표시해 진행도를 명확히 한다.
  - 실패 시 재시작 안내를 보여준다.

```python
round_text = small.render(f"Round {round_no}", True, (191, 219, 254))
screen.blit(round_text, (20, 20))

if game_over:
    over = font.render("Game Over", True, (254, 202, 202))
    hint = small.render("Press R to Restart", True, (254, 226, 226))
```

##### 선언한 변수/함수의 목적
  - `round_text`: 라운드 표시 텍스트
  - `over`, `hint`: 종료 안내 텍스트

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `f"Round {round_no}"`: 현재 난이도 단계를 숫자로 표시해 목표 인식을 돕는다.
  - `screen.blit(round_text, ...)`: 좌상단 HUD로 고정해 매 프레임 동일 위치에서 읽히게 한다.
  - `if game_over:`: 종료 상태일 때만 오버레이 문구를 표시해 상태 전환을 명확히 전달한다.
  - `"Press R to Restart"`: 재시작 입력을 바로 안내해 사용자 이탈 없이 다음 시도를 유도한다.

#### class 3 최종 코드

```python
# class 2 코드에 round_no/game_over/start_next_round()/reset_game()를 통합해 실행
# 핵심: 성공 시 시퀀스 확장, 실패 시 종료, R 재시작
```

---

## class 4. 버튼 사운드 + UI 폴리싱

### 목표

- 버튼별 고유 톤 사운드를 재생한다.
- 재생 모드와 입력 모드 모두에서 소리 피드백을 준다.
- 최종 실행 가능한 Simon 사운드 코드를 완성한다.

### 핵심 변수/함수

- `tone_by_pad`: 버튼 ID별 사운드 객체
- `make_tone()`: 주파수 기반 톤 생성 함수
- `play_pad()`: 점등 + 사운드 동시 재생 함수

### 단계별 구현

#### 단계 1) 톤 생성 함수

##### 세부목표
  - 외부 파일 없이 코드로 버튼음을 생성한다.
  - 버튼마다 다른 음높이를 부여해 구분 가능한 청각 패턴을 만든다.

```python
import math
from array import array

pygame.mixer.pre_init(44100, -16, 1, 256)
pygame.init()


def make_tone(freq, ms=190, volume=0.35):
    sample_rate = 44100
    count = int(sample_rate * (ms / 1000.0))
    data = array("h")
    amp = int(32767 * volume)
    for i in range(count):
        t = i / sample_rate
        data.append(int(math.sin(2 * math.pi * freq * t) * amp))
    return pygame.mixer.Sound(buffer=data)
```

##### 선언한 변수/함수의 목적
  - `make_tone()`: 톤 생성 함수
  - `sample_rate`: 샘플링 주파수
  - `data`: PCM 샘플 배열

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `pygame.mixer.pre_init(...)`: 믹서 포맷을 먼저 고정해 버퍼 기반 사운드 재생 호환성을 높인다.
  - `count = int(sample_rate * (ms / 1000.0))`: 재생 길이를 샘플 수로 변환해 정확한 음 길이를 만든다.
  - `math.sin(2 * math.pi * freq * t)`: 사인파를 생성해 깨끗한 단일 톤을 만든다.
  - `pygame.mixer.Sound(buffer=data)`: 생성한 PCM 데이터를 사운드 객체로 변환해 즉시 재생 가능하게 한다.

#### 단계 2) 버튼별 사운드 매핑

##### 세부목표
  - 버튼 ID와 음높이를 매핑한다.
  - 클릭/재생 로직에서 공통 재생 함수를 사용한다.

```python
tone_by_pad = {
    0: make_tone(261.6),
    1: make_tone(329.6),
    2: make_tone(392.0),
    3: make_tone(523.3),
}


def play_pad(pid, now):
    global active_pad, active_until
    active_pad = pid
    active_until = now + 220
    tone_by_pad[pid].play()
```

##### 선언한 변수/함수의 목적
  - `tone_by_pad`: 버튼별 사운드 테이블
  - `play_pad()`: 점등+사운드 실행 함수

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `tone_by_pad = {...}`: 버튼마다 다른 주파수를 지정해 시퀀스를 소리만으로도 구분 가능하게 만든다.
  - `play_pad(pid, now)`: 점등과 사운드 재생을 하나의 함수로 묶어 재생/입력 모드에서 같은 피드백을 보장한다.
  - `active_until = now + 220`: 소리 재생 길이와 점등 길이를 맞춰 시각/청각 피드백 타이밍을 동기화한다.
  - `tone_by_pad[pid].play()`: 선택된 버튼 음을 즉시 재생해 입력 반응성을 높인다.

#### 단계 3) 시퀀스 재생/입력에 사운드 통합

##### 세부목표
  - 자동 재생 단계에서도 소리를 함께 재생한다.
  - 사용자 클릭 때도 같은 음이 나게 한다.

```python
# 재생 모드
pid = sequence[show_idx]
play_pad(pid, now)

# 입력 모드
if pid is not None:
    play_pad(pid, pygame.time.get_ticks())
```

##### 선언한 변수/함수의 목적
  - `pid`: 현재 재생/입력된 버튼 ID

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `pid = sequence[show_idx]`: 재생할 정답 버튼을 인덱스로 가져와 시각/청각 출력을 같은 기준으로 맞춘다.
  - `play_pad(pid, now)`: 자동 재생에서도 점등과 소리를 동시에 출력해 기억 단서를 강화한다.
  - `if pid is not None:`: 유효한 클릭 입력일 때만 사운드를 재생해 공백 클릭 노이즈를 막는다.
  - `play_pad(pid, pygame.time.get_ticks())`: 입력 피드백을 즉시 재생해 사용자가 자신의 입력을 확인하게 한다.

#### 단계 4) 최종 HUD/상태 안내

##### 세부목표
  - 현재 모드/라운드/안내 문구를 정리해 표시한다.
  - 사운드 포함 최종 버전을 안정적으로 실행한다.

```python
mode_text = {"show": "Listen", "input": "Repeat", "idle": "Stopped"}[mode]
hud = small.render(f"Round {round_no}  Mode {mode_text}", True, (226, 232, 240))
screen.blit(hud, (18, 16))
```

##### 선언한 변수/함수의 목적
  - `mode_text`: 모드 표시 문자열
  - `hud`: 상단 상태 HUD

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `mode_text = {...}[mode]`: 내부 상태 코드를 사용자 친화 문구로 변환해 현재 단계 이해를 돕는다.
  - `f"Round {round_no}  Mode {mode_text}"`: 난이도와 모드를 한 줄에 보여줘 즉시 상황 판단이 가능하게 한다.
  - `screen.blit(hud, (18, 16))`: HUD를 상단에 고정 배치해 버튼 영역을 가리지 않게 한다.
  - 모드 정보를 프레임마다 갱신해 재생-입력-종료 전환이 화면에서 즉시 확인되게 만든다.

#### class 4 최종 코드

```python
import pygame
import random
import sys
import math
from array import array

pygame.mixer.pre_init(44100, -16, 1, 256)
pygame.init()

WIDTH, HEIGHT = 640, 640
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Simon Sound - Final")
clock = pygame.time.Clock()
font = pygame.font.SysFont("arial", 42, bold=True)
small = pygame.font.SysFont("arial", 26)

pads = [
    {"id": 0, "rect": pygame.Rect(120, 120, 180, 180), "base": (185, 28, 28), "light": (248, 113, 113)},
    {"id": 1, "rect": pygame.Rect(340, 120, 180, 180), "base": (22, 163, 74), "light": (74, 222, 128)},
    {"id": 2, "rect": pygame.Rect(120, 340, 180, 180), "base": (37, 99, 235), "light": (96, 165, 250)},
    {"id": 3, "rect": pygame.Rect(340, 340, 180, 180), "base": (202, 138, 4), "light": (250, 204, 21)},
]



def make_tone(freq, ms=190, volume=0.35):
    sample_rate = 44100
    count = int(sample_rate * (ms / 1000.0))
    data = array("h")
    amp = int(32767 * volume)
    for i in range(count):
        t = i / sample_rate
        data.append(int(math.sin(2 * math.pi * freq * t) * amp))
    return pygame.mixer.Sound(buffer=data)



tone_by_pad = {
    0: make_tone(261.6),
    1: make_tone(329.6),
    2: make_tone(392.0),
    3: make_tone(523.3),
}

active_pad = None
active_until = 0

sequence = [random.randint(0, 3)]
input_index = 0
mode = "show"
show_idx = 0
next_tick = pygame.time.get_ticks() + 500
status = "watch"
round_no = 1
game_over = False



def pad_at(pos):
    for pad in pads:
        if pad["rect"].collidepoint(pos):
            return pad["id"]
    return None



def draw_pads(surface):
    for pad in pads:
        color = pad["light"] if pad["id"] == active_pad else pad["base"]
        pygame.draw.rect(surface, color, pad["rect"], border_radius=18)



def play_pad(pid, now):
    global active_pad, active_until
    active_pad = pid
    active_until = now + 220
    tone_by_pad[pid].play()



def play_step(now):
    global show_idx, mode, input_index, next_tick
    if now < next_tick:
        return

    if show_idx < len(sequence):
        pid = sequence[show_idx]
        play_pad(pid, now)
        show_idx += 1
        next_tick = now + 430
    else:
        mode = "input"
        input_index = 0



def start_next_round():
    global round_no, mode, show_idx, next_tick, status
    sequence.append(random.randint(0, 3))
    round_no += 1
    mode = "show"
    show_idx = 0
    next_tick = pygame.time.get_ticks() + 550
    status = "watch"



def reset_game():
    global sequence, input_index, mode, show_idx, next_tick, status, round_no, game_over
    sequence = [random.randint(0, 3)]
    input_index = 0
    mode = "show"
    show_idx = 0
    next_tick = pygame.time.get_ticks() + 500
    status = "watch"
    round_no = 1
    game_over = False


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

        if mode == "input" and not game_over and event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
            pid = pad_at(event.pos)
            if pid is not None:
                play_pad(pid, now)
                if pid == sequence[input_index]:
                    input_index += 1
                    if input_index == len(sequence):
                        status = "clear"
                else:
                    status = "fail"

    if active_pad is not None and now >= active_until:
        active_pad = None

    if not game_over and mode == "show":
        play_step(now)

    if status == "clear" and not game_over:
        start_next_round()
    elif status == "fail" and not game_over:
        game_over = True
        mode = "idle"

    label = {
        "watch": "Watch the pattern",
        "clear": "Great!",
        "fail": "Wrong!",
    }.get(status, "Your turn")

    screen.fill((15, 23, 42))
    draw_pads(screen)

    mode_text = {"show": "Listen", "input": "Repeat", "idle": "Stopped"}[mode]
    hud = small.render(f"Round {round_no}  Mode {mode_text}", True, (226, 232, 240))
    screen.blit(hud, (18, 16))

    text = small.render(label, True, (191, 219, 254))
    screen.blit(text, (190, 62))

    if game_over:
        over = font.render("Game Over", True, (254, 202, 202))
        hint = small.render("Press R to Restart", True, (254, 226, 226))
        screen.blit(over, (WIDTH // 2 - over.get_width() // 2, 12))
        screen.blit(hint, (WIDTH // 2 - hint.get_width() // 2, 54))

    pygame.display.flip()
    clock.tick(60)
```
