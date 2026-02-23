# EightPuzzle.py paaiškinimas

## Kas yra Aštuonių dėlionių uždavinys?

Aštuonių dėlionių uždavinys — tai klasikinis paieškos uždavinys: **3×3 lentoje** yra 8 numeruotos plytelės (1–8) ir viena tuščia vieta (0). Tikslas — **perstumdinėjant plytelę po plytelę** pasiekti tikslinę tvarką.

```
Pradinė būsena:        Tikslinė būsena:
┌───┬───┬───┐          ┌───┬───┬───┐
│ 2 │ 4 │ 3 │          │ 1 │ 2 │ 3 │
├───┼───┼───┤    →     ├───┼───┼───┤
│ 1 │ 5 │ 6 │          │ 4 │ 5 │ 6 │
├───┼───┼───┤          ├───┼───┼───┤
│ 7 │ 8 │ _ │          │ 7 │ 8 │ _ │
└───┴───┴───┘          └───┴───┴───┘
```

---

## Failo struktūra

```python
from search import *

puzzle = EightPuzzle((2, 4, 3, 1, 5, 6, 7, 8, 0))
solution = breadth_first_graph_search(puzzle).solution()
print(f'{solution}')
```

- **`from search import *`** — importuojama `EightPuzzle` klasė ir paieškos algoritmai iš `search.py`
- **`EightPuzzle((2, 4, 3, 1, 5, 6, 7, 8, 0))`** — sukuriamas uždavinio objektas su nurodyta pradine būsena
- **`breadth_first_graph_search(puzzle)`** — sprendimas ieškomas **paieška platyn (BFS)**
- Alternatyva (užkomentuota): `best_first_graph_search` su heuristika `h(n)`

---

## Būsenos reprezentacija

Būsena yra **9 elementų kortežas** (tuple), kur:
- **Indeksas** = pozicija lentoje (nuo 0 iki 8, kairėje viršuje pradedant)
- **Reikšmė** = plytelės numeris (0 = tuščia vieta)

```
Lentoje:               Korteže:
┌───┬───┬───┐
│ 2 │ 4 │ 3 │   →   (2, 4, 3, 1, 5, 6, 7, 8, 0)
│ 1 │ 5 │ 6 │       indeksai: 0,1,2,3,4,5,6,7,8
│ 7 │ 8 │ _ │
└───┴───┴───┘

Tikslinė:
(1, 2, 3, 4, 5, 6, 7, 8, 0)
```

### 5 būsenų pavyzdys (veiksmai su tuščia vieta _):

| Žingsnis | Veiksmas       | Būsena (kortežas)              | Lenta                    |
|----------|----------------|-------------------------------|--------------------------|
| 0 (pradinė) | —           | `(2, 4, 3, 1, 5, 6, 7, 8, 0)` | `2 4 3 / 1 5 6 / 7 8 _` |
| 1        | LEFT (_ ←)    | `(2, 4, 3, 1, 5, 6, 7, 0, 8)` | `2 4 3 / 1 5 6 / 7 _ 8` |
| 2        | UP (_ ↑)      | `(2, 4, 3, 1, 0, 6, 7, 5, 8)` | `2 4 3 / 1 _ 6 / 7 5 8` |
| 3        | LEFT (_ ←)    | `(2, 4, 3, 0, 1, 6, 7, 5, 8)` | `2 4 3 / _ 1 6 / 7 5 8` |
| 4        | UP (_ ↑)      | `(0, 4, 3, 2, 1, 6, 7, 5, 8)` | `_ 4 3 / 2 1 6 / 7 5 8` |

> **Pastaba:** veiksmai (UP/DOWN/LEFT/RIGHT) aprašo kuria kryptimi juda **tuščia vieta**.

---

## EightPuzzle klasė (iš search.py)

| Metodas | Paaiškinimas |
|---|---|
| `__init__(initial, goal)` | Pradinė būsena — korteže nurodytas išdėstymas; tikslinė — `(1,2,3,4,5,6,7,8,0)` |
| `find_blank_square(state)` | Randa tuščios vietos indeksą (kur `0`) |
| `actions(state)` | Grąžina galimus veiksmus: UP/DOWN/LEFT/RIGHT (pašalina negalimus kraštuose) |
| `result(state, action)` | Apkeičia tuščią vietą su kaimynine plytelė, grąžina naują būseną |
| `goal_test(state)` | Tikslas pasiektas, kai būsena lygi `(1,2,3,4,5,6,7,8,0)` |
| `h(node)` | **Heuristika** — kiek plytelių yra ne savo vietoje (misplaced tiles) |

---

## Kaip veikia galimų veiksmų nustatymas

```
Lenta (indeksai):     Tuščia 0-oje:      Tuščia 4-oje:
┌───┬───┬───┐         Negalima: LEFT, UP  Galima: visos 4
│ 0 │ 1 │ 2 │
│ 3 │ 4 │ 5 │         Tuščia 8-oje:
│ 6 │ 7 │ 8 │         Negalima: RIGHT, DOWN
└───┴───┴───┘
```

- `index % 3 == 0` → kairysis kraštas → negalima LEFT
- `index < 3` → viršutinė eilutė → negalima UP
- `index % 3 == 2` → dešinysis kraštas → negalima RIGHT
- `index > 5` → apatinė eilutė → negalima DOWN

---

## Paieškos algoritmai ir eilė

### Paieška į plotį (BFS) — `breadth_first_graph_search`

```
Eilė (FIFO – deque):
[pradinė_būsena]
  → išimam, plečiam, dedame vaikus į GALĄ
  → [1_ž_būsena_1, 1_ž_būsena_2, ...]
  → išimam pirmą, plečiam...
```

- Aplankomi **visi** to paties gylio mazgai **prieš** pereinant giliau
- **Garantuoja trumpiausią kelią** (mažiausiai žingsnių)
- Naudoja daug atminties

### Paieška į gylį (DFS) — `depth_first_graph_search`

```
Eilė (LIFO – Stack):
[pradinė_būsena]
  → išimam VIRŠŲ, plečiam, dedame vaikus į VIRŠŲ
  → einame kuo giliau prieš šakojantis
```

- Naudoja mažiau atminties, bet neranda trumpiausio kelio

### Informuota paieška (Best-First) — `best_first_graph_search`

```python
best_first_graph_search(puzzle, lambda n: puzzle.h(n))
```

- Naudoja **heuristiką h(n)** — kiek plytelių ne savo vietoje
- Kiekvieną kartą plečia **perspektyviausią** mazgą (mažiausias h)
- Greitesnė nei BFS didelėms erdvėms

---

## Heuristikos

### 1. Neteisingų plytelių skaičius (Misplaced Tiles)

```python
h(n) = sum(s != g for (s, g) in zip(node.state, self.goal))
```

Pavyzdys: `(2,4,3,1,5,6,7,8,0)` vs tikslas `(1,2,3,4,5,6,7,8,0)`:
- Pozicija 0: `2 ≠ 1` → +1
- Pozicija 1: `4 ≠ 2` → +1
- Pozicija 3: `1 ≠ 4` → +1
- **h = 3**

### 2. Manheteno atstumas (Manhattan Distance) — tikslesnė

```
Kiekvienai plytelei skaičiuojama: |dabartinė_eilutė - tikslinė_eilutė| + |dabartinis_stulpelis - tikslinės_stulpelis|
```

---

## Architektūriniai komponentai

```
┌─────────────────────────────────────────────────────┐
│                   Problem (abstrakti klasė)          │
│  initial, goal, actions(), result(), goal_test(), h()│
└────────────────────┬────────────────────────────────┘
                     │ paveldi
┌────────────────────▼────────────────────────────────┐
│              EightPuzzle                            │
│  Konkretus uždavinys: plytelių dėlionė              │
└────────────────────┬────────────────────────────────┘
                     │ naudoja
┌────────────────────▼────────────────────────────────┐
│                    Node                             │
│  state, parent, action, path_cost, depth            │
│  expand(), solution(), path()                       │
└────────────────────┬────────────────────────────────┘
                     │ valdo
┌────────────────────▼────────────────────────────────┐
│           Paieškos algoritmai                       │
│  breadth_first_graph_search  → FIFO eilė (deque)    │
│  depth_first_graph_search    → Stack (list)          │
│  best_first_graph_search     → Prioritetinė eilė    │
└─────────────────────────────────────────────────────┘
```

### Node klasė

Kiekvienas **mazgas** (node) paieškos medyje saugo:
- `state` — dabartinė būsena (kortežas)
- `parent` — iš kur atėjome (tėvinis mazgas)
- `action` — koks veiksmas buvo atliktas
- `path_cost` — bendros išlaidos nuo pradžios
- `depth` — gylis medyje

---

## Vykdymo eiga

```
1. Sukuriamas pradinės būsenos Node: state=(2,4,3,1,5,6,7,8,0)
2. Dedamas į eilę (frontier)
3. Ciklas:
   a. Išimamas mazgas iš eilės
   b. Tikrinama: ar tai tikslinė būsena? → jei taip, grąžinamas sprendimas
   c. Jei ne — kviečiamas expand(): generuojami vaikai visiems galimams veiksmams
   d. Vaikai (kurie dar neaplankyta) dedami į eilę
4. solution() grąžina veiksmų seką atkūrus kelią per .parent nuorodas
```
