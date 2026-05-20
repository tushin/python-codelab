## 개요

- 대상: `pygame`으로 처음 퍼즐 게임을 만드는 초급 학습자
- 방식: class별로 "실행되는 코드 + 기능 1개 확장" 방식
- 최종 산출물: 신경쇠약(메모리 카드 짝 맞추기) 게임 1개
- 실행 환경: Python 3.10+ / `pip install pygame`

## 목차

1. class 1. 카드 보드 만들기 + 카드 뒤집기
2. class 2. 두 장 비교 + 짝 판정
3. class 3. 승리 조건 + 재시작 + 시도 횟수
4. class 4. 제한 시간 + 난이도 확장

---

## class 1. 카드 보드 만들기 + 카드 뒤집기

### 목표

- 카드 짝 데이터(2개씩)를 만들고 섞는다.
- 화면에 카드 뒷면/앞면을 그린다.
- 마우스 클릭으로 카드를 한 장 뒤집는다.

### 핵심 변수/함수

- `ROWS`, `COLS`: 보드 행/열 크기
- `cards`: 카드 상태 리스트 (`symbol`, `revealed`, `matched`, `rect`)
- `build_deck()`: 카드 심볼을 2장씩 만들어 섞음
- `draw_cards()`: 카드 상태에 따라 앞면/뒷면 렌더링

### 단계별 구현

#### 단계 1) 기본 창과 보드 크기 상수

##### 세부목표
  - 초기화와 화면 생성 코드를 연결해 실행 가능한 기본 게임 화면을 만든다.
  - 현재 단계의 상태값이 다음 단계 동작으로 어떻게 이어지는지 이해한다.

```python
import pygame
import random
import sys

pygame.init()

ROWS, COLS = 4, 4
CARD_W, CARD_H = 120, 120
GAP = 14
MARGIN = 24

WIDTH = MARGIN * 2 + COLS * CARD_W + (COLS - 1) * GAP
HEIGHT = 160 + ROWS * CARD_H + (ROWS - 1) * GAP

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Memory Game - Class 1")
clock = pygame.time.Clock()
```

##### 선언한 변수/함수의 목적
  - `ROWS`: 보드 행 개수
  - `COLS`: 보드 열 개수
  - `CARD_W`: 카드 가로 크기
  - `CARD_H`: 카드 세로 크기
  - `GAP`: 요소 사이 간격
  - `MARGIN`: 바깥 여백
  - `WIDTH`: 게임 화면 가로 크기
  - `HEIGHT`: 게임 화면 세로 크기
  - `screen`: 화면 출력 대상 객체
  - `clock`: 프레임 속도 제어 객체

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `screen = pygame.display.set_mode((WIDTH, HEIGHT))`: 게임 창 크기를 확정해 좌표 기준을 만든다.
  - `pygame.init()`: 키보드 입력, 화면 출력, 사운드 같은 pygame 하위 모듈을 한 번에 초기화해 하드웨어와 통신할 준비를 마친다.
  - `ROWS, COLS = 4, 4`: 행/열 개수를 먼저 고정해 카드 수, 덱 크기, 배치 좌표 계산의 기준을 통일한다.
  - `CARD_W, CARD_H = 120, 120`: 카드 크기를 고정해 클릭 판정용 `Rect`와 실제 렌더링 크기가 어긋나지 않게 맞춘다.

#### 단계 2) 짝 카드 덱 만들기

##### 세부목표
  - 반복되는 규칙을 함수로 분리해 가독성과 유지보수성을 높인다.
  - 학습 목표는 `def`/매개변수/반환값 기준으로 코드를 읽고 기능 단위로 나누는 것이다.

```python
SYMBOLS = ["A", "B", "C", "D", "E", "F", "G", "H"]

def build_deck(rows, cols):
    pair_count = (rows * cols) // 2
    selected = SYMBOLS[:pair_count]
    deck = selected * 2
    random.shuffle(deck)
    return deck
```

##### 선언한 변수/함수의 목적
  - `build_deck()`: 짝 카드 덱을 만들고 섞는 함수
  - `rows`: 보드 행/열 입력값이다. 카드 개수와 좌표 배치 계산의 기준이 된다.
  - `cols`: 보드 행/열 입력값이다. 카드 개수와 좌표 배치 계산의 기준이 된다.
  - `SYMBOLS`: 덱 생성에 사용할 심볼 집합을 담는 기준 목록이다.
  - `pair_count`: 카드 쌍 개수
  - `selected`: 선택된 심볼 목록
  - `deck`: 게임 객체 목록을 저장한다. 루프에서 순회하며 이동·충돌 판정·렌더링에 함께 사용한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `def build_deck(rows, cols):`: `build_deck()` 함수 정의를 시작해 관련 로직을 기능 단위로 분리한다.
  - `random.shuffle(deck)`: 카드 순서를 섞어 매 판 배치를 랜덤화한다.
  - `SYMBOLS = ["A", "B", "C", "D", "E", "F", "G", "H"]`: 사용할 심볼 풀을 미리 정해 덱 생성 시 난이도와 시각 구성을 일관되게 유지한다.
  - `pair_count = (rows * cols) // 2`: 보드 칸 수를 2로 나눠 필요한 짝 개수를 정확히 계산해 덱 길이 불일치를 방지한다.

#### 단계 3) 카드 상태 리스트 생성

##### 세부목표
  - 반복되는 규칙을 함수로 분리해 가독성과 유지보수성을 높인다.
  - 학습 목표는 `def`/매개변수/반환값 기준으로 코드를 읽고 기능 단위로 나누는 것이다.

```python
def make_cards(rows, cols):
    deck = build_deck(rows, cols)
    cards = []
    idx = 0

    for r in range(rows):
        for c in range(cols):
            x = MARGIN + c * (CARD_W + GAP)
            y = 120 + r * (CARD_H + GAP)
            rect = pygame.Rect(x, y, CARD_W, CARD_H)
            cards.append(
                {
                    "symbol": deck[idx],
                    "revealed": False,
                    "matched": False,
                    "rect": rect,
                }
            )
            idx += 1

    return cards

cards = make_cards(ROWS, COLS)
```

##### 선언한 변수/함수의 목적
  - `make_cards()`: 카드 상태 리스트를 초기화하는 함수
  - `rows`: 보드 행/열 입력값이다. 카드 개수와 좌표 배치 계산의 기준이 된다.
  - `cols`: 보드 행/열 입력값이다. 카드 개수와 좌표 배치 계산의 기준이 된다.
  - `deck`: 게임 객체 목록을 저장한다. 루프에서 순회하며 이동·충돌 판정·렌더링에 함께 사용한다.
  - `cards`: 게임 객체 목록을 저장한다. 루프에서 순회하며 이동·충돌 판정·렌더링에 함께 사용한다.
  - `idx`: 덱에서 현재 카드에 할당할 심볼 위치를 가리키는 인덱스다.
  - `rect`: 충돌/렌더링용 사각형 객체

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `def make_cards(rows, cols):`: `make_cards()` 함수 정의를 시작해 관련 로직을 기능 단위로 분리한다.
  - `for r in range(rows):`: 행 단위로 반복해 카드 배치의 세로 위치를 규칙적으로 계산한다.
  - `deck = build_deck(rows, cols)`: 현재 보드 크기에 맞는 카드 덱을 생성해 이후 셔플/배치 로직이 정확한 데이터로 시작되게 한다.
  - `cards = []`: 카드 상태를 누적할 리스트를 분리해 생성/뒤집기/매칭 갱신을 한 구조에서 관리하게 한다.

#### 단계 4) 카드 그리기 함수

##### 세부목표
  - 그리기 코드를 함수로 분리해 같은 렌더링 규칙을 재사용 가능하게 만든다.
  - 학습 목표는 `def`/매개변수/반환값 기준으로 코드를 읽고 기능 단위로 나누는 것이다.

```python
BG = (245, 250, 244)
BACK = (73, 115, 84)
FRONT = (255, 255, 255)
TEXT = (20, 28, 25)
LINE = (180, 196, 183)

font_title = pygame.font.SysFont("arial", 34, bold=True)
font_card = pygame.font.SysFont("arial", 52, bold=True)

def draw_cards(surface, cards):
    for card in cards:
        if card["revealed"] or card["matched"]:
            pygame.draw.rect(surface, FRONT, card["rect"], border_radius=14)
            pygame.draw.rect(surface, LINE, card["rect"], 2, border_radius=14)
            text = font_card.render(card["symbol"], True, TEXT)
            text_rect = text.get_rect(center=card["rect"].center)
            surface.blit(text, text_rect)
        else:
            pygame.draw.rect(surface, BACK, card["rect"], border_radius=14)
```

##### 선언한 변수/함수의 목적
  - `draw_cards()`: 카드 상태에 맞춰 화면에 그리는 함수
  - `surface`: 그리기 대상 Surface
  - `cards`: 게임 객체 목록을 저장한다. 루프에서 순회하며 이동·충돌 판정·렌더링에 함께 사용한다.
  - `BG`: 배경 색상
  - `BACK`: 카드 뒷면 색상
  - `FRONT`: 카드 앞면 색상
  - `TEXT`: 텍스트 색상
  - `LINE`: 선/테두리 색상
  - `font_title`: 텍스트 렌더링용 폰트 객체
  - `font_card`: 텍스트 렌더링용 폰트 객체
  - `text`: 카드 앞면 심볼을 렌더링한 텍스트 Surface다.
  - `text_rect`: 심볼 텍스트를 카드 중앙에 배치하기 위한 위치 박스다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `def draw_cards(surface, cards):`: `draw_cards()` 함수 정의를 시작해 관련 로직을 기능 단위로 분리한다.
  - `if card["revealed"] or card["matched"]:`: 이미 열린 카드나 매칭 완료 카드는 다시 처리하지 않아 상태 불일치를 막는다.
  - `for card in cards:`: 카드 목록 전체를 순회해 각 카드의 앞면/뒷면 상태를 현재 값대로 렌더링한다.
  - `BG = (245, 250, 244)`: 배경색을 별도 상수로 고정해 카드 강조 색과 대비를 안정적으로 유지한다.

#### 단계 5) 클릭한 카드만 뒤집기

##### 세부목표
  - 입력 이벤트를 상태값 변화와 연결해 사용자의 조작이 게임 동작으로 이어지게 만든다.
  - 학습 목표는 `if/elif` 분기로 입력 조건을 나누고 상태 업데이트 순서를 이해하는 것이다.

```python
while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
            pos = event.pos
            for card in cards:
                if card["rect"].collidepoint(pos) and not card["revealed"] and not card["matched"]:
                    card["revealed"] = True
                    break

    screen.fill(BG)
    title = font_title.render("Memory Game", True, TEXT)
    screen.blit(title, (MARGIN, 28))
    draw_cards(screen, cards)

    pygame.display.flip()
    clock.tick(60)
```

##### 선언한 변수/함수의 목적
  - `pos`: 입력 좌표를 잠깐 저장해 클릭/겹침 판정에 사용한다.
  - `title`: 상단 제목 문구를 렌더링한 텍스트 Surface다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if event.type == pygame.QUIT:`: 이벤트 타입을 비교해 입력/업데이트 처리를 분기한다.
  - `screen.blit(title, (MARGIN, 28))`: Surface를 지정 위치에 배치해 UI/텍스트를 출력한다.
  - `pygame.display.flip()`: 현재 프레임 렌더링 결과를 화면에 반영한다.
  - `clock.tick(60)`: 프레임 속도를 제한해 실행 속도를 안정화한다.

#### class 1 최종 코드

```python
import pygame
import random
import sys

pygame.init()

ROWS, COLS = 4, 4
CARD_W, CARD_H = 120, 120
GAP = 14
MARGIN = 24

WIDTH = MARGIN * 2 + COLS * CARD_W + (COLS - 1) * GAP
HEIGHT = 160 + ROWS * CARD_H + (ROWS - 1) * GAP

BG = (245, 250, 244)
BACK = (73, 115, 84)
FRONT = (255, 255, 255)
TEXT = (20, 28, 25)
LINE = (180, 196, 183)

SYMBOLS = ["A", "B", "C", "D", "E", "F", "G", "H"]

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Memory Game - Class 1")
clock = pygame.time.Clock()
font_title = pygame.font.SysFont("arial", 34, bold=True)
font_card = pygame.font.SysFont("arial", 52, bold=True)

def build_deck(rows, cols):
    pair_count = (rows * cols) // 2
    selected = SYMBOLS[:pair_count]
    deck = selected * 2
    random.shuffle(deck)
    return deck

def make_cards(rows, cols):
    deck = build_deck(rows, cols)
    cards = []
    idx = 0

    for r in range(rows):
        for c in range(cols):
            x = MARGIN + c * (CARD_W + GAP)
            y = 120 + r * (CARD_H + GAP)
            rect = pygame.Rect(x, y, CARD_W, CARD_H)
            cards.append(
                {
                    "symbol": deck[idx],
                    "revealed": False,
                    "matched": False,
                    "rect": rect,
                }
            )
            idx += 1
    return cards

def draw_cards(surface, cards):
    for card in cards:
        if card["revealed"] or card["matched"]:
            pygame.draw.rect(surface, FRONT, card["rect"], border_radius=14)
            pygame.draw.rect(surface, LINE, card["rect"], 2, border_radius=14)
            text = font_card.render(card["symbol"], True, TEXT)
            text_rect = text.get_rect(center=card["rect"].center)
            surface.blit(text, text_rect)
        else:
            pygame.draw.rect(surface, BACK, card["rect"], border_radius=14)

cards = make_cards(ROWS, COLS)

while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
            pos = event.pos
            for card in cards:
                if card["rect"].collidepoint(pos) and not card["revealed"] and not card["matched"]:
                    card["revealed"] = True
                    break

    screen.fill(BG)
    title = font_title.render("Memory Game", True, TEXT)
    screen.blit(title, (MARGIN, 28))
    draw_cards(screen, cards)
    pygame.display.flip()
    clock.tick(60)
```

---

## class 2. 두 장 비교 + 짝 판정

### 목표

- 한 번에 2장까지만 뒤집히도록 제한한다.
- 두 카드 심볼이 같으면 `matched=True`로 고정한다.
- 다르면 0.8초 뒤 다시 뒷면으로 되돌린다.

### 핵심 변수/함수

- `first_pick`, `second_pick`: 현재 뒤집은 카드 인덱스
- `evaluating`: 비교 대기 중인지 여부
- `evaluate_started`: 뒤집은 두 카드를 보여주기 시작한 시점(ms)

### 단계별 구현

#### 단계 1) 선택 상태 변수 추가

##### 세부목표
  - 선택 상태 변수 추가 단계의 코드를 게임 루프에 연결해 실제 동작으로 확인한다.
  - 현재 단계의 상태값이 다음 단계 동작으로 어떻게 이어지는지 이해한다.

```python
first_pick = None
second_pick = None
evaluating = False
evaluate_started = 0
```

##### 선언한 변수/함수의 목적
  - `first_pick`: 첫 번째로 선택한 카드 인덱스를 저장해 다음 클릭 분기 기준으로 사용한다.
  - `second_pick`: 두 번째로 선택한 카드 인덱스를 저장해 짝 비교 시점을 제어한다.
  - `evaluating`: 두 카드 비교 대기 상태를 나타내며 입력 차단/비교 실행 분기를 제어한다.
  - `evaluate_started`: 시간 기준값을 저장한다. 현재 시각과 차이를 계산해 제한시간/지연 판정에 사용한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `first_pick = None`: 첫 선택 카드가 아직 없음을 `None`으로 명시해 클릭 처리 분기를 명확히 시작한다.
  - `second_pick = None`: 두 번째 선택 전 상태를 `None`으로 두어 한 장만 뒤집힌 상태와 구분한다.
  - `evaluating = False`: 비교 대기 상태를 `False`로 시작해 초기 클릭이 차단되지 않도록 한다.
  - `evaluate_started = 0`: 비교 지연 타이머 시작값을 0으로 두어 첫 비교 전 시간 계산 오차를 막는다.

#### 단계 2) 클릭 처리에서 "2장까지만" 허용

##### 세부목표
  - 입력 이벤트를 상태값 변화와 연결해 사용자의 조작이 게임 동작으로 이어지게 만든다.
  - 학습 목표는 `if/elif` 분기로 입력 조건을 나누고 상태 업데이트 순서를 이해하는 것이다.

```python
elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
    if evaluating:
        continue

    pos = event.pos
    for i, card in enumerate(cards):
        if card["rect"].collidepoint(pos) and not card["revealed"] and not card["matched"]:
            card["revealed"] = True
            if first_pick is None:
                first_pick = i
            elif second_pick is None:
                second_pick = i
                evaluating = True
                evaluate_started = pygame.time.get_ticks()
            break
```

##### 선언한 변수/함수의 목적
  - `pos`: 입력 좌표를 잠깐 저장해 클릭/겹침 판정에 사용한다.
  - `first_pick`: 첫 번째로 선택한 카드 인덱스를 저장해 다음 클릭 분기 기준으로 사용한다.
  - `second_pick`: 두 번째로 선택한 카드 인덱스를 저장해 짝 비교 시점을 제어한다.
  - `evaluating`: 두 카드 비교 대기 상태를 나타내며 입력 차단/비교 실행 분기를 제어한다.
  - `evaluate_started`: 시간 기준값을 저장한다. 현재 시각과 차이를 계산해 제한시간/지연 판정에 사용한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:`: 이벤트 타입을 비교해 입력/업데이트 처리를 분기한다.
  - `if evaluating:`: 두 장 비교 대기 중일 때만 지연/입력 차단 로직을 실행해 상태 갱신 순서를 지킨다.
  - `for i, card in enumerate(cards):`: 모든 카드를 검사해 실제로 클릭된 카드와 인덱스를 찾아낸다.
  - `continue`: 현재 반복의 나머지 처리를 건너뛰고 다음 입력/객체 검사로 넘어가도록 제어 흐름을 전환한다.

#### 단계 3) 두 카드 비교 함수

##### 세부목표
  - 반복되는 규칙을 함수로 분리해 가독성과 유지보수성을 높인다.
  - 학습 목표는 `def`/매개변수/반환값 기준으로 코드를 읽고 기능 단위로 나누는 것이다.

```python
def resolve_pair(cards, first_idx, second_idx):
    first = cards[first_idx]
    second = cards[second_idx]

    if first["symbol"] == second["symbol"]:
        first["matched"] = True
        second["matched"] = True
    else:
        first["revealed"] = False
        second["revealed"] = False
```

##### 선언한 변수/함수의 목적
  - `resolve_pair()`: 두 카드의 짝 여부를 판정하는 함수
  - `cards`: 게임 객체 목록을 저장한다. 루프에서 순회하며 이동·충돌 판정·렌더링에 함께 사용한다.
  - `first_idx`: 첫 번째로 선택된 카드의 인덱스다.
  - `second_idx`: 두 번째로 선택된 카드의 인덱스다.
  - `first`: 첫 선택 카드 객체를 임시로 꺼내 심볼 비교를 단순화한다.
  - `second`: 두 번째 선택 카드 객체를 임시로 꺼내 매칭 분기를 단순화한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `def resolve_pair(cards, first_idx, second_idx):`: `resolve_pair()` 함수 정의를 시작해 관련 로직을 기능 단위로 분리한다.
  - `if first["symbol"] == second["symbol"]:`: 두 카드 심볼이 같을 때만 매칭 처리로 넘어가 규칙 기반 판정이 유지된다.
  - `first = cards[first_idx]`: 첫 카드 객체를 별도 변수로 분리해 심볼 비교와 상태 갱신 코드를 읽기 쉽게 만든다.
  - `second = cards[second_idx]`: 두 번째 카드 객체를 분리해 짝 비교 로직에서 인덱스 접근 중복을 줄인다.

#### 단계 4) 0.8초 뒤 판정 실행

##### 세부목표
  - 타이머/시간 기반 로직으로 업데이트 주기를 제어해 프레임과 게임 규칙을 분리한다.
  - 학습 목표는 `set_timer`·`get_ticks`로 시간 기반 동작을 제어하는 방법을 익히는 것이다.

```python
if evaluating:
    now = pygame.time.get_ticks()
    if now - evaluate_started >= 800:
        resolve_pair(cards, first_pick, second_pick)
        first_pick = None
        second_pick = None
        evaluating = False
```

##### 선언한 변수/함수의 목적
  - `now`: 현재 시각(ms) 기준값으로 지연 시간 계산에 사용한다.
  - `first_pick`: 첫 번째로 선택한 카드 인덱스를 저장해 다음 클릭 분기 기준으로 사용한다.
  - `second_pick`: 두 번째로 선택한 카드 인덱스를 저장해 짝 비교 시점을 제어한다.
  - `evaluating`: 두 카드 비교 대기 상태를 나타내며 입력 차단/비교 실행 분기를 제어한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if evaluating:`: 두 장 비교 대기 중일 때만 지연/입력 차단 로직을 실행해 상태 갱신 순서를 지킨다.
  - `now = pygame.time.get_ticks()`: 현재 시각(ms)을 고정해 같은 프레임에서 지연 시간 계산 기준이 흔들리지 않게 한다.
  - `if now - evaluate_started >= 800:`: 카드 공개 후 0.8초가 지난 시점에만 비교를 실행해 확인 시간을 보장한다.
  - `resolve_pair(cards, first_pick, second_pick)`: 두 카드의 짝 여부를 확정해 `matched` 또는 `revealed` 상태를 바꾸고, 다음 입력 가능 상태로 넘어가게 한다.

#### class 2 최종 코드

```python
import pygame
import random
import sys

pygame.init()

ROWS, COLS = 4, 4
CARD_W, CARD_H = 120, 120
GAP = 14
MARGIN = 24

WIDTH = MARGIN * 2 + COLS * CARD_W + (COLS - 1) * GAP
HEIGHT = 160 + ROWS * CARD_H + (ROWS - 1) * GAP

BG = (245, 250, 244)
BACK = (73, 115, 84)
FRONT = (255, 255, 255)
TEXT = (20, 28, 25)
LINE = (180, 196, 183)

SYMBOLS = ["A", "B", "C", "D", "E", "F", "G", "H"]

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Memory Game - Class 2")
clock = pygame.time.Clock()
font_title = pygame.font.SysFont("arial", 34, bold=True)
font_card = pygame.font.SysFont("arial", 52, bold=True)

def build_deck(rows, cols):
    pair_count = (rows * cols) // 2
    selected = SYMBOLS[:pair_count]
    deck = selected * 2
    random.shuffle(deck)
    return deck

def make_cards(rows, cols):
    deck = build_deck(rows, cols)
    cards = []
    idx = 0
    for r in range(rows):
        for c in range(cols):
            x = MARGIN + c * (CARD_W + GAP)
            y = 120 + r * (CARD_H + GAP)
            rect = pygame.Rect(x, y, CARD_W, CARD_H)
            cards.append(
                {
                    "symbol": deck[idx],
                    "revealed": False,
                    "matched": False,
                    "rect": rect,
                }
            )
            idx += 1
    return cards

def draw_cards(surface, cards):
    for card in cards:
        if card["revealed"] or card["matched"]:
            pygame.draw.rect(surface, FRONT, card["rect"], border_radius=14)
            pygame.draw.rect(surface, LINE, card["rect"], 2, border_radius=14)
            text = font_card.render(card["symbol"], True, TEXT)
            text_rect = text.get_rect(center=card["rect"].center)
            surface.blit(text, text_rect)
        else:
            pygame.draw.rect(surface, BACK, card["rect"], border_radius=14)

def resolve_pair(cards, first_idx, second_idx):
    first = cards[first_idx]
    second = cards[second_idx]
    if first["symbol"] == second["symbol"]:
        first["matched"] = True
        second["matched"] = True
    else:
        first["revealed"] = False
        second["revealed"] = False

cards = make_cards(ROWS, COLS)
first_pick = None
second_pick = None
evaluating = False
evaluate_started = 0

while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
            if evaluating:
                continue
            pos = event.pos
            for i, card in enumerate(cards):
                if card["rect"].collidepoint(pos) and not card["revealed"] and not card["matched"]:
                    card["revealed"] = True
                    if first_pick is None:
                        first_pick = i
                    elif second_pick is None:
                        second_pick = i
                        evaluating = True
                        evaluate_started = pygame.time.get_ticks()
                    break

    if evaluating:
        now = pygame.time.get_ticks()
        if now - evaluate_started >= 800:
            resolve_pair(cards, first_pick, second_pick)
            first_pick = None
            second_pick = None
            evaluating = False

    screen.fill(BG)
    title = font_title.render("Memory Game", True, TEXT)
    screen.blit(title, (MARGIN, 28))
    draw_cards(screen, cards)
    pygame.display.flip()
    clock.tick(60)
```

---

## class 3. 승리 조건 + 재시작 + 시도 횟수

### 목표

- 카드 16장이 전부 `matched=True`가 되면 승리 처리한다.
- 시도 횟수(`tries`)를 화면에 표시한다.
- `R` 키를 누르면 새 게임으로 재시작한다.

### 핵심 변수/함수

- `tries`: 카드 2장을 뒤집은 횟수
- `win`: 게임 클리어 상태
- `is_win(cards)`: 전체 카드 매칭 완료 확인
- `reset_game()`: 카드/상태를 초기값으로 되돌림

### 단계별 구현

#### 단계 1) 시도 횟수 증가

##### 세부목표
  - 타이머/시간 기반 로직으로 업데이트 주기를 제어해 프레임과 게임 규칙을 분리한다.
  - 학습 목표는 `set_timer`·`get_ticks`로 시간 기반 동작을 제어하는 방법을 익히는 것이다.

```python
tries = 0
```

```python
elif second_pick is None:
    second_pick = i
    evaluating = True
    evaluate_started = pygame.time.get_ticks()
    tries += 1
```

##### 선언한 변수/함수의 목적
  - `tries`: 점수/시도 횟수 값
  - `second_pick`: 두 번째로 선택한 카드 인덱스를 저장해 짝 비교 시점을 제어한다.
  - `evaluating`: 두 카드 비교 대기 상태를 나타내며 입력 차단/비교 실행 분기를 제어한다.
  - `evaluate_started`: 시간 기준값을 저장한다. 현재 시각과 차이를 계산해 제한시간/지연 판정에 사용한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `elif second_pick is None:`: 첫 카드가 이미 선택된 상태에서만 두 번째 선택 로직을 실행해 선택 순서를 강제한다.
  - `tries = 0`: 시도 횟수를 0부터 누적해 난이도 피드백과 클리어 효율 표시를 정확히 제공한다.
  - `second_pick = i`: 두 번째 선택 전 상태를 `None`으로 두어 한 장만 뒤집힌 상태와 구분한다.
  - `evaluating = True`: 비교 대기 상태를 `False`로 시작해 초기 클릭이 차단되지 않도록 한다.

#### 단계 2) 승리 함수

##### 세부목표
  - 반복되는 규칙을 함수로 분리해 가독성과 유지보수성을 높인다.
  - 학습 목표는 `def`/매개변수/반환값 기준으로 코드를 읽고 기능 단위로 나누는 것이다.

```python
def is_win(cards):
    return all(card["matched"] for card in cards)
```

##### 선언한 변수/함수의 목적
  - `is_win()`: 클리어 여부를 검사하는 함수
  - `cards`: 게임 객체 목록을 저장한다. 루프에서 순회하며 이동·충돌 판정·렌더링에 함께 사용한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `def is_win(cards):`: `is_win()` 함수 정의를 시작해 관련 로직을 기능 단위로 분리한다.
  - `return all(card["matched"] for card in cards)`: 모든 카드의 매칭 완료 여부를 한 번에 판정해, 클리어 화면 전환 조건으로 바로 사용할 수 있게 한다.
  - `all(...)`을 사용하면 반복문 없이도 "하나라도 False면 실패" 규칙을 간결하게 표현할 수 있다.
  - `is_win(cards)`를 함수로 분리해 매 프레임 UI 갱신과 입력 분기에서 같은 승리 규칙을 재사용할 수 있다.

#### 단계 3) 재시작 함수

##### 세부목표
  - 반복되는 규칙을 함수로 분리해 가독성과 유지보수성을 높인다.
  - 학습 목표는 `def`/매개변수/반환값 기준으로 코드를 읽고 기능 단위로 나누는 것이다.

```python
def reset_game():
    new_cards = make_cards(ROWS, COLS)
    return {
        "cards": new_cards,
        "first_pick": None,
        "second_pick": None,
        "evaluating": False,
        "evaluate_started": 0,
        "tries": 0,
        "win": False,
    }
```

##### 선언한 변수/함수의 목적
  - `reset_game()`: 게임 상태를 초기값으로 되돌리는 함수
  - `new_cards`: 재시작 시 새로 만든 카드 목록을 담아 상태를 통째로 교체한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `def reset_game():`: `reset_game()` 함수 정의를 시작해 관련 로직을 기능 단위로 분리한다.
  - `new_cards = make_cards(ROWS, COLS)`: 재시작 시 새 카드 목록을 먼저 만들어 기존 상태와 섞이지 않고 판을 통째로 교체하게 한다.
  - `return {`: 게임 재시작에 필요한 상태 묶음을 한 번에 반환해 호출 측이 즉시 상태를 교체하도록 한다.
  - `"cards": new_cards,`: 새로 만든 카드 목록을 반환 상태에 포함해 다음 프레임부터 새 판 구성이 바로 적용되게 한다.

#### 단계 4) 안내 문구와 재시작 키 처리

##### 세부목표
  - 입력 이벤트를 상태값 변화와 연결해 사용자의 조작이 게임 동작으로 이어지게 만든다.
  - 학습 목표는 `if/elif` 분기로 입력 조건을 나누고 상태 업데이트 순서를 이해하는 것이다.

```python
if event.type == pygame.KEYDOWN and event.key == pygame.K_r:
    state = reset_game()
    cards = state["cards"]
    first_pick = state["first_pick"]
    second_pick = state["second_pick"]
    evaluating = state["evaluating"]
    evaluate_started = state["evaluate_started"]
    tries = state["tries"]
    win = state["win"]
```

```python
if is_win(cards):
    win = True

tries_text = font_info.render(f"tries: {tries}", True, TEXT)
screen.blit(tries_text, (MARGIN, 76))

if win:
    msg = font_info.render("clear! press R to restart", True, (21, 107, 42))
else:
    msg = font_info.render("find all pairs", True, TEXT)
screen.blit(msg, (MARGIN + 170, 76))
```

##### 선언한 변수/함수의 목적
  - `state`: 초기화 함수 반환 상태를 잠깐 담아 여러 상태 변수를 한 번에 갱신한다.
  - `cards`: 게임 객체 목록을 저장한다. 루프에서 순회하며 이동·충돌 판정·렌더링에 함께 사용한다.
  - `first_pick`: 첫 번째로 선택한 카드 인덱스를 저장해 다음 클릭 분기 기준으로 사용한다.
  - `second_pick`: 두 번째로 선택한 카드 인덱스를 저장해 짝 비교 시점을 제어한다.
  - `evaluating`: 두 카드 비교 대기 상태를 나타내며 입력 차단/비교 실행 분기를 제어한다.
  - `evaluate_started`: 시간 기준값을 저장한다. 현재 시각과 차이를 계산해 제한시간/지연 판정에 사용한다.
  - `tries`: 점수/시도 횟수 값
  - `win`: 클리어 여부를 저장해 안내 문구와 입력 허용 범위를 제어한다.
  - `tries_text`: 시도 횟수 문구를 렌더링한 HUD 텍스트 Surface다.
  - `msg`: 상황별 안내 문구를 렌더링한 보조 텍스트 Surface다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if event.type == pygame.KEYDOWN and event.key == pygame.K_r:`: 이벤트 타입을 비교해 입력/업데이트 처리를 분기한다.
  - `screen.blit(tries_text, (MARGIN, 76))`: Surface를 지정 위치에 배치해 UI/텍스트를 출력한다.
  - `if is_win(cards):`: 모든 카드 매칭이 끝났을 때만 승리 상태를 켜 클리어 판정을 정확히 맞춘다.
  - `state = reset_game()`: 초기화 함수 반환 상태를 한 번에 받아 각 상태 변수를 동기화된 값으로 갱신한다.

#### class 3 최종 코드

```python
import pygame
import random
import sys

pygame.init()

ROWS, COLS = 4, 4
CARD_W, CARD_H = 120, 120
GAP = 14
MARGIN = 24

WIDTH = MARGIN * 2 + COLS * CARD_W + (COLS - 1) * GAP
HEIGHT = 160 + ROWS * CARD_H + (ROWS - 1) * GAP

BG = (245, 250, 244)
BACK = (73, 115, 84)
FRONT = (255, 255, 255)
TEXT = (20, 28, 25)
LINE = (180, 196, 183)

SYMBOLS = ["A", "B", "C", "D", "E", "F", "G", "H"]

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Memory Game - Class 3")
clock = pygame.time.Clock()
font_title = pygame.font.SysFont("arial", 34, bold=True)
font_card = pygame.font.SysFont("arial", 52, bold=True)
font_info = pygame.font.SysFont("arial", 28, bold=True)

def build_deck(rows, cols):
    pair_count = (rows * cols) // 2
    selected = SYMBOLS[:pair_count]
    deck = selected * 2
    random.shuffle(deck)
    return deck

def make_cards(rows, cols):
    deck = build_deck(rows, cols)
    cards = []
    idx = 0
    for r in range(rows):
        for c in range(cols):
            x = MARGIN + c * (CARD_W + GAP)
            y = 120 + r * (CARD_H + GAP)
            rect = pygame.Rect(x, y, CARD_W, CARD_H)
            cards.append(
                {
                    "symbol": deck[idx],
                    "revealed": False,
                    "matched": False,
                    "rect": rect,
                }
            )
            idx += 1
    return cards

def draw_cards(surface, cards):
    for card in cards:
        if card["revealed"] or card["matched"]:
            pygame.draw.rect(surface, FRONT, card["rect"], border_radius=14)
            pygame.draw.rect(surface, LINE, card["rect"], 2, border_radius=14)
            text = font_card.render(card["symbol"], True, TEXT)
            text_rect = text.get_rect(center=card["rect"].center)
            surface.blit(text, text_rect)
        else:
            pygame.draw.rect(surface, BACK, card["rect"], border_radius=14)

def resolve_pair(cards, first_idx, second_idx):
    first = cards[first_idx]
    second = cards[second_idx]
    if first["symbol"] == second["symbol"]:
        first["matched"] = True
        second["matched"] = True
    else:
        first["revealed"] = False
        second["revealed"] = False

def is_win(cards):
    return all(card["matched"] for card in cards)

def reset_game():
    new_cards = make_cards(ROWS, COLS)
    return {
        "cards": new_cards,
        "first_pick": None,
        "second_pick": None,
        "evaluating": False,
        "evaluate_started": 0,
        "tries": 0,
        "win": False,
    }

state = reset_game()
cards = state["cards"]
first_pick = state["first_pick"]
second_pick = state["second_pick"]
evaluating = state["evaluating"]
evaluate_started = state["evaluate_started"]
tries = state["tries"]
win = state["win"]

while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

        if event.type == pygame.KEYDOWN and event.key == pygame.K_r:
            state = reset_game()
            cards = state["cards"]
            first_pick = state["first_pick"]
            second_pick = state["second_pick"]
            evaluating = state["evaluating"]
            evaluate_started = state["evaluate_started"]
            tries = state["tries"]
            win = state["win"]

        elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
            if evaluating or win:
                continue

            pos = event.pos
            for i, card in enumerate(cards):
                if card["rect"].collidepoint(pos) and not card["revealed"] and not card["matched"]:
                    card["revealed"] = True
                    if first_pick is None:
                        first_pick = i
                    elif second_pick is None:
                        second_pick = i
                        evaluating = True
                        evaluate_started = pygame.time.get_ticks()
                        tries += 1
                    break

    if evaluating:
        now = pygame.time.get_ticks()
        if now - evaluate_started >= 800:
            resolve_pair(cards, first_pick, second_pick)
            first_pick = None
            second_pick = None
            evaluating = False

    if is_win(cards):
        win = True

    screen.fill(BG)
    title = font_title.render("Memory Game", True, TEXT)
    screen.blit(title, (MARGIN, 28))
    tries_text = font_info.render(f"tries: {tries}", True, TEXT)
    screen.blit(tries_text, (MARGIN, 76))

    if win:
        msg = font_info.render("clear! press R to restart", True, (21, 107, 42))
    else:
        msg = font_info.render("find all pairs", True, TEXT)
    screen.blit(msg, (MARGIN + 170, 76))

    draw_cards(screen, cards)
    pygame.display.flip()
    clock.tick(60)
```

---

## class 4. 제한 시간 + 난이도 확장

### 목표

- 제한 시간 안에 짝을 다 맞추는 모드를 만든다.
- 시간 초과 시 실패 문구를 표시한다.
- 4x4(입문)와 4x5(확장) 같은 난이도를 쉽게 바꿀 수 있게 설계한다.

### 핵심 변수/함수

- `TIME_LIMIT`: 제한 시간(초)
- `started_at`: 게임 시작 시각(ms)
- `time_left`: 남은 시간
- `game_over`: 시간 만료 상태

### 단계별 구현

#### 단계 1) 타이머 상태 추가

##### 세부목표
  - 타이머/시간 기반 로직으로 업데이트 주기를 제어해 프레임과 게임 규칙을 분리한다.
  - 학습 목표는 `set_timer`·`get_ticks`로 시간 기반 동작을 제어하는 방법을 익히는 것이다.

```python
TIME_LIMIT = 75
started_at = pygame.time.get_ticks()
game_over = False
```

##### 선언한 변수/함수의 목적
  - `TIME_LIMIT`: 제한 시간(초)
  - `started_at`: 시간 기준값을 저장한다. 현재 시각과 차이를 계산해 제한시간/지연 판정에 사용한다.
  - `game_over`: 게임 종료 상태를 저장해 업데이트 중단과 재시작 입력 허용을 제어한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `TIME_LIMIT = 75`: 제한 시간을 초 단위 상수로 고정해 타이머 계산과 UI 안내 기준을 일치시킨다.
  - `started_at = pygame.time.get_ticks()`: 게임 시작 시각을 저장해 경과 시간과 남은 시간을 매 프레임 일관되게 계산한다.
  - `game_over = False`: 게임 상태를 즉시 종료 상태로 전환해 입력/업데이트 루프가 플레이 모드에서 빠져나오게 한다.
  - `TIME_LIMIT`를 상수로 분리해 두면 난이도 조절 시 타이머 로직 전체를 건드리지 않고 시간값만 바꾸면 된다.

#### 단계 2) 매 프레임 남은 시간 계산

##### 세부목표
  - 타이머/시간 기반 로직으로 업데이트 주기를 제어해 프레임과 게임 규칙을 분리한다.
  - 학습 목표는 `set_timer`·`get_ticks`로 시간 기반 동작을 제어하는 방법을 익히는 것이다.

```python
elapsed_sec = (pygame.time.get_ticks() - started_at) // 1000
time_left = max(0, TIME_LIMIT - elapsed_sec)

if time_left == 0 and not win:
    game_over = True
```

##### 선언한 변수/함수의 목적
  - `elapsed_sec`: 시간 기준값을 저장한다. 현재 시각과 차이를 계산해 제한시간/지연 판정에 사용한다.
  - `time_left`: 시간 기준값을 저장한다. 현재 시각과 차이를 계산해 제한시간/지연 판정에 사용한다.
  - `game_over`: 게임 종료 상태를 저장해 업데이트 중단과 재시작 입력 허용을 제어한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `if time_left == 0 and not win:`: 시간이 0이고 아직 클리어 전일 때만 시간초과 게임오버를 발생시킨다.
  - `elapsed_sec = (pygame.time.get_ticks() - started_at) // 1000`: 시작 시각 대비 경과 초를 계산해 남은 시간 산출과 게임오버 판정에 재사용한다.
  - `time_left = max(0, TIME_LIMIT - elapsed_sec)`: 남은 시간을 매 프레임 갱신해 타이머 UI와 시간초과 분기가 같은 값을 보게 만든다.
  - `game_over = True`: 게임 상태를 즉시 종료 상태로 전환해 입력/업데이트 루프가 플레이 모드에서 빠져나오게 한다.

#### 단계 3) 시간 종료 시 클릭 차단

##### 세부목표
  - 입력 이벤트를 상태값 변화와 연결해 사용자의 조작이 게임 동작으로 이어지게 만든다.
  - 학습 목표는 `if/elif` 분기로 입력 조건을 나누고 상태 업데이트 순서를 이해하는 것이다.

```python
elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
    if evaluating or win or game_over:
        continue
```

##### 선언한 변수/함수의 목적
  - `state`: 초기화 함수 반환 상태를 잠깐 담아 여러 상태 변수를 한 번에 갱신한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:`: 이벤트 타입을 비교해 입력/업데이트 처리를 분기한다.
  - `if evaluating or win or game_over:`: 비교 중이거나 게임이 끝난 상태에서는 클릭을 막아 의도치 않은 상태 변경을 차단한다.
  - `continue`: 현재 반복의 나머지 처리를 건너뛰고 다음 입력/객체 검사로 넘어가도록 제어 흐름을 전환한다.
  - `if evaluating or win or game_over:`를 한 줄로 묶어 비교 중/승리/시간초과 상태에서 동일한 안전 규칙을 유지한다.

#### 단계 4) 난이도 확장 예시

##### 세부목표
  - 난이도 확장 예시 단계의 코드를 게임 루프에 연결해 실제 동작으로 확인한다.
  - 현재 단계의 상태값이 다음 단계 동작으로 어떻게 이어지는지 이해한다.

```python
# 입문: ROWS, COLS = 4, 4
# 확장: ROWS, COLS = 4, 5   # 20장(10쌍)
# 고급: ROWS, COLS = 6, 6   # 36장(18쌍)
```

##### 선언한 변수/함수의 목적
  - `state`: 초기화 함수 반환 상태를 잠깐 담아 여러 상태 변수를 한 번에 갱신한다.

##### 코드 블럭의 핵심코드 및 처음 배우는 표현 설명
  - `# 입문: ROWS, COLS = 4, 4`: 16장(8쌍) 구성으로 초급 학습자가 규칙을 빠르게 익히도록 시작 난이도를 낮춘다.
  - `# 확장: ROWS, COLS = 4, 5`: 카드 수를 20장으로 늘려 기억 부담과 탐색 시간이 함께 증가하는 중급 모드를 만든다.
  - `# 고급: ROWS, COLS = 6, 6`: 36장 구성으로 긴 플레이 타임과 높은 난도를 요구하는 고급 모드로 확장할 수 있다.
  - `ROWS, COLS`만 바꾸는 구조라서 카드 생성, 배치, 판정 로직을 수정하지 않고도 난이도를 단계적으로 조정할 수 있다.

#### class 4 최종 코드

```python
import pygame
import random
import sys

pygame.init()

ROWS, COLS = 4, 4
CARD_W, CARD_H = 120, 120
GAP = 14
MARGIN = 24
TIME_LIMIT = 75

WIDTH = MARGIN * 2 + COLS * CARD_W + (COLS - 1) * GAP
HEIGHT = 170 + ROWS * CARD_H + (ROWS - 1) * GAP

BG = (245, 250, 244)
BACK = (73, 115, 84)
FRONT = (255, 255, 255)
TEXT = (20, 28, 25)
LINE = (180, 196, 183)
OK = (21, 107, 42)
FAIL = (168, 31, 31)

SYMBOLS = ["A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M", "N", "O", "P", "Q", "R"]

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Memory Game")
clock = pygame.time.Clock()

font_title = pygame.font.SysFont("arial", 34, bold=True)
font_card = pygame.font.SysFont("arial", 52, bold=True)
font_info = pygame.font.SysFont("arial", 28, bold=True)

def build_deck(rows, cols):
    pair_count = (rows * cols) // 2
    selected = SYMBOLS[:pair_count]
    deck = selected * 2
    random.shuffle(deck)
    return deck

def make_cards(rows, cols):
    deck = build_deck(rows, cols)
    cards = []
    idx = 0

    for r in range(rows):
        for c in range(cols):
            x = MARGIN + c * (CARD_W + GAP)
            y = 120 + r * (CARD_H + GAP)
            rect = pygame.Rect(x, y, CARD_W, CARD_H)
            cards.append(
                {
                    "symbol": deck[idx],
                    "revealed": False,
                    "matched": False,
                    "rect": rect,
                }
            )
            idx += 1

    return cards

def draw_cards(surface, cards):
    for card in cards:
        if card["revealed"] or card["matched"]:
            pygame.draw.rect(surface, FRONT, card["rect"], border_radius=14)
            pygame.draw.rect(surface, LINE, card["rect"], 2, border_radius=14)
            text = font_card.render(card["symbol"], True, TEXT)
            text_rect = text.get_rect(center=card["rect"].center)
            surface.blit(text, text_rect)
        else:
            pygame.draw.rect(surface, BACK, card["rect"], border_radius=14)

def resolve_pair(cards, first_idx, second_idx):
    first = cards[first_idx]
    second = cards[second_idx]

    if first["symbol"] == second["symbol"]:
        first["matched"] = True
        second["matched"] = True
    else:
        first["revealed"] = False
        second["revealed"] = False

def is_win(cards):
    return all(card["matched"] for card in cards)

def reset_game():
    return {
        "cards": make_cards(ROWS, COLS),
        "first_pick": None,
        "second_pick": None,
        "evaluating": False,
        "evaluate_started": 0,
        "tries": 0,
        "win": False,
        "game_over": False,
        "started_at": pygame.time.get_ticks(),
    }

state = reset_game()
cards = state["cards"]
first_pick = state["first_pick"]
second_pick = state["second_pick"]
evaluating = state["evaluating"]
evaluate_started = state["evaluate_started"]
tries = state["tries"]
win = state["win"]
game_over = state["game_over"]
started_at = state["started_at"]

while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

        if event.type == pygame.KEYDOWN and event.key == pygame.K_r:
            state = reset_game()
            cards = state["cards"]
            first_pick = state["first_pick"]
            second_pick = state["second_pick"]
            evaluating = state["evaluating"]
            evaluate_started = state["evaluate_started"]
            tries = state["tries"]
            win = state["win"]
            game_over = state["game_over"]
            started_at = state["started_at"]

        elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
            if evaluating or win or game_over:
                continue

            pos = event.pos
            for i, card in enumerate(cards):
                if card["rect"].collidepoint(pos) and not card["revealed"] and not card["matched"]:
                    card["revealed"] = True

                    if first_pick is None:
                        first_pick = i
                    elif second_pick is None:
                        second_pick = i
                        evaluating = True
                        evaluate_started = pygame.time.get_ticks()
                        tries += 1
                    break

    if evaluating:
        now = pygame.time.get_ticks()
        if now - evaluate_started >= 800:
            resolve_pair(cards, first_pick, second_pick)
            first_pick = None
            second_pick = None
            evaluating = False

    if is_win(cards):
        win = True

    elapsed_sec = (pygame.time.get_ticks() - started_at) // 1000
    time_left = max(0, TIME_LIMIT - elapsed_sec)
    if time_left == 0 and not win:
        game_over = True

    screen.fill(BG)

    title = font_title.render("Memory Game", True, TEXT)
    screen.blit(title, (MARGIN, 24))

    tries_text = font_info.render(f"tries: {tries}", True, TEXT)
    timer_text = font_info.render(f"time: {time_left}", True, TEXT)
    screen.blit(tries_text, (MARGIN, 74))
    screen.blit(timer_text, (MARGIN + 180, 74))

    if win:
        msg = font_info.render("clear! press R to restart", True, OK)
    elif game_over:
        msg = font_info.render("time over! press R to retry", True, FAIL)
    else:
        msg = font_info.render("find all pairs", True, TEXT)
    screen.blit(msg, (MARGIN + 340, 74))

    draw_cards(screen, cards)

    pygame.display.flip()
    clock.tick(60)
```
