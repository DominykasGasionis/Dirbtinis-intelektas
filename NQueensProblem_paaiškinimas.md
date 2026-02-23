# NQueensProblem.py paaiškinimas

## Kas yra N karalienių uždavinys?

N karalienių uždavinys — tai klasikinis kombinatorikos uždavinys: reikia **N karalienių išdėstyti ant N×N šachmatų lentos** taip, kad **nė viena karalienė nekirstų kitos** (nei eilutėje, nei stulpelyje, nei įstrižainėje).

---

## Failo struktūra

Failą galima suskirstyti į **3 dalis**:

### 1 dalis — Uždavinio sprendimas (1–11 eilutės)

```python
from search import *

number_of_queens = 4

nq_problem = NQueensProblem(number_of_queens)
solution = best_first_graph_search(nq_problem, lambda n: nq_problem.h(n)).solution()

print(solution)
```

- **Importuojama** viskas iš `search.py` — ten apibrėžta klasė `NQueensProblem` ir paieškos algoritmai.
- **`number_of_queens = 4`** — nustatomas lentos dydis (4×4, t.y. 4 karalienės).
- **`NQueensProblem(4)`** — sukuriamas uždavinio objektas. Pradinė būsena: `(-1, -1, -1, -1)` — visos 4 kolonos tuščios.
- **`best_first_graph_search(..., h(n))`** — sprendimas ieškomas **geriausio-pirmojo grafo paieška**, kur vertinimo funkcija `h(n)` grąžina kertančių karalienių porų skaičių (kuo mažiau — tuo geriau).
- Užkomentuotos alternatyvos:
  - `recursive_best_first_search` — rekursyvi geriausio-pirmojo paieška
  - `breadth_first_graph_search` — paieška platyn

### 2 dalis — Paprastas tekstinis sprendinio piešimas (20–38 eilutės)

```python
def plot_solution(solution):
    n = len(solution)
    fig, ax = plt.subplots()
    # ... tinklelio nustatymas ...
    for row in range(n):
        for col in range(n):
            if solution[row] == col:
                ax.text(col, row, "Q", ha='center', va='center', color='red', fontsize=24)
    ax.invert_yaxis()
    plt.show()
```

- Naudoja **matplotlib** šachmatų lentai nupiešti.
- Kiekvienoje lentos pozicijoje, kur stovi karalienė, rašoma raudona raidė **„Q"**.
- **`solution`** yra sąrašas, kur indeksas = eilutė, reikšmė = stulpelis (pvz., `[1, 3, 0, 2]` reiškia: 0-oje eilutėje karalienė 1-ame stulpelyje, 1-oje eilutėje — 3-iame ir t.t.).

### 3 dalis — Grafinė vizualizacija su karalienės paveikslėliu (45–74 eilutės)

```python
def plot_NQueens(solution):
    n = len(solution)
    board = np.array([2 * int((i + j) % 2) for ...]).reshape((n, n))
    im = Image.open('images/queen_s.png')
    # ... karalienės paveikslėlis dedamas ant lentos ...
    plt.show()
```

- Piešia **šachmatų lentą** su juodais ir baltais langeliais (naudojant `cmap='binary'`).
- Karalienės vaizduojamos **paveikslėliu** (`images/queen_s.png`), o ne tekstu.
- Palaiko du sprendinio formatus:
  - **`dict`** — iš `NQueensCSP` (CSP sprendiklio)
  - **`list`** — iš `NQueensProblem` (paieškos algoritmo)
- **Aktyvuojama tik kai `number_of_queens == 8`** (73 eilutė), nes paveikslėlio pozicijos apskaičiuotos 8×8 lentai.

---

## NQueensProblem klasė (iš search.py)

| Metodas | Paaiškinimas |
|---|---|
| `__init__(N)` | Sukuria pradinę būseną `(-1, -1, ..., -1)` — tuščią lentą su N stulpelių |
| `actions(state)` | Grąžina galimas eilutes kairiausiam tuščiam stulpeliui (tik tas, kurios nesukelia konflikto) |
| `result(state, row)` | Pastato karalienę nurodytoje eilutėje kairiausiam tuščiam stulpelyje |
| `conflicted(state, row, col)` | Tikrina, ar pozicija (row, col) kertasi su jau esančiomis karalienėmis |
| `conflict(r1, c1, r2, c2)` | Tikrina dvi karalienės: ar tos pačios eilutės, stulpelio, arba įstrižainės |
| `goal_test(state)` | Tikslas pasiektas, kai visi stulpeliai užpildyti ir nėra konfliktų |
| `h(node)` | **Heuristika** — skaičiuoja kertančių karalienių porų skaičių (naudojama informuotoje paieškoje) |

---

## Būsenos reprezentacija

Būsena yra **N ilgio kortežas** (tuple), kur:
- **Indeksas** = stulpelio numeris
- **Reikšmė** = eilutės numeris (arba `-1`, jei stulpelis dar tuščias)

Pavyzdys su 4 karalienėmis — sprendimas `(1, 3, 0, 2)`:

```
     0   1   2   3
  ┌───┬───┬───┬───┐
0 │   │   │ Q │   │   ← karalienė stulpelyje 2, eilutėje 0
  ├───┼───┼───┼───┤
1 │ Q │   │   │   │   ← karalienė stulpelyje 0, eilutėje 1
  ├───┼───┼───┼───┤
2 │   │   │   │ Q │   ← karalienė stulpelyje 3, eilutėje 2
  ├───┼───┼───┼───┤
3 │   │ Q │   │   │   ← karalienė stulpelyje 1, eilutėje 3
  └───┴───┴───┴───┘
```

---

## Vykdymo eiga

```
1. Sukuriama pradinė būsena: (-1, -1, -1, -1)
2. Algoritmas bando dėti karalienę į 0-ąjį stulpelį (eilutės 0–3)
3. Kiekvienam pasirinkimui — bando dėti į 1-ąjį stulpelį (tik nekonfliktines eilutes)
4. Kartojama, kol visi stulpeliai užpildyti be konfliktų
5. Grąžinamas veiksmų sąrašas (solution) — eilučių numeriai kiekvienam stulpeliui
6. Sprendinys atvaizduojamas grafiškai
```

---

## Pastabos

- Dideliems N (pvz., 30) sprendimas gali užtrukti **>30 min** (nurodyta komentare).
- Grafinė vizualizacija su karalienės paveikslėliu veikia tik **8×8 lentai** (pozicijos hardcoded).
- Užkomentuotame kode (76–84 eil.) yra interaktyvus vartotojo klausimas, ar rodyti grafinį vaizdą.
