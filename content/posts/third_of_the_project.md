+++
date = '2026-04-06T00:00:00+01:00'
draft = true
title = 'Step 3 of Ant Simulation Project : Colony and Emergence'
math = true
summary = 'Premier essai de suivi de traces par gradient d antennes. Observations, bugs, et prochaines etapes.'
+++

## Introduction

Dans le step précédent on a construit la classe `Ant` : mouvement par angle, rebond sur les murs, dépôt de phéromones, et les positions des antennes. Les antennes étaient calculées mais complètement inutilisées - les fourmis faisaient une marche aléatoire pure en ignorant toutes les traces qu'elles déposaient.

Le step 3 branche les entrées sensorielles sur le mouvement. La règle est encore codée à la main : une simple différence pondérée entre antenne gauche et droite biaise `delta_theta`. L'objectif est d'observer les **premières traces émergentes** — une structure collective qui naît de décisions purement locales, sans aucune coordination centrale.

---

## La règle de direction

$$\delta\theta = W \cdot (C_g - C_d) + U(-\delta_r,\, \delta_r)$$

$W$ est `ANTENNA_WEIGHT`, $C_{\text{left}}$ et $C_{\text{right}}$ sont les concentrations aux deux antennes, et $\delta_{\text{rand}}$ est `RANDOM_DIR`. Si la gauche est plus forte, la fourmi tourne à gauche, et vice versa. La magnitude scale continûment avec le gradient.

**Quelle phéromone suivre ?** Une fourmi avec de la nourriture veut rentrer au nid, une sans nourriture cherche une source :

- `food_carried > THRESHOLD` → a de la nourriture → suit les phéromones `HOME`
- `food_carried ≤ THRESHOLD` → cherche → suit les phéromones `FOOD`

---

## Le code

### `config.py`

```python
import numpy as np 

EVAPORATION_RATE = 0.997
GRID_WIDTH = 360
GRID_HEIGHT = 240
DIFFUSION_SIGMA = 0.3
CELL_SIZE = 3
FPS = 24
WINDOW_WIDTH = GRID_WIDTH * CELL_SIZE
WINDOW_HEIGHT = GRID_HEIGHT * CELL_SIZE
PHEROMONE_DEPOSIT = 0.7

COLOR_HOME = (139, 90, 43)
COLOR_FOOD = (50, 205, 50)
COLOR_BACKGROUND = (0, 0, 0)
COLOR_NEST = (255, 255, 255)
NEST_RADIUS = 2
NEST_X = 100
NEST_Y = 100

LENGTH_ANTENNA = 1
ANGLE_ANTENNA = np.pi / 4
N_ANTS = 20
COLOR_ANT = (255, 165, 0)
MAX_FOOD_CARRIED = 0.5
FOOD_COLLECT_AMOUNT = 0.5
EAT_DURATION = 8
ANTENNA_WEIGHT = np.pi / 3
TRESHOLD_FOOD = 0.45
RANDOM_DIR = np.pi / 8

N_FOOD_TYPES = 2
COLOR_APHID = (255, 220, 0)
COLOR_SUGAR = (100, 200, 255)
RECHARGE_RATE_APHID = 0.01
RECHARGE_RATE_SUGAR = 0
```

### `core/ant.py`

```python
from config import *
import numpy as np

class Ant:

    def __init__(self, x, y, angle_antenna, length_antenna):
        self.x = x
        self.y = y
        self.food_carried = 0
        self.eating_timer = 0
        self.direction = np.random.uniform(0, 2 * np.pi)
        self.angle_antenna = angle_antenna
        self.length_antenna = length_antenna

    def move(self, delta_theta):
        if self.eating_timer > 0:
            self.eating_timer -= 1
            return int(self.x), int(self.y)
        self.direction += delta_theta
        new_x = self.x + np.cos(self.direction)
        new_y = self.y + np.sin(self.direction)

        x_clipped = np.clip(new_x, 0, GRID_WIDTH - 1)
        y_clipped = np.clip(new_y, 0, GRID_HEIGHT - 1)

        if x_clipped != new_x:
            self.direction = np.pi - self.direction
        if y_clipped != new_y:
            self.direction = -self.direction

        old_x, old_y = int(self.x), int(self.y)
        self.x = x_clipped
        self.y = y_clipped
        return old_x, old_y

    def get_antenna_pos(self):
        return [
            (self.x + self.length_antenna * np.cos(self.direction + self.angle_antenna),
             self.y + self.length_antenna * np.sin(self.direction + self.angle_antenna)),
            (self.x + self.length_antenna * np.cos(self.direction - self.angle_antenna),
             self.y + self.length_antenna * np.sin(self.direction - self.angle_antenna))
        ]

    def is_at_nest(self, nest):
        x_nest, y_nest = nest.get_x_y()
        return (self.x - x_nest) ** 2 + (self.y - y_nest) ** 2 <= NEST_RADIUS ** 2

    def interact(self, source, at_nest):
        if source is not None and self.food_carried < MAX_FOOD_CARRIED:
            want_to_collect = min(FOOD_COLLECT_AMOUNT, MAX_FOOD_CARRIED - self.food_carried)
            taken = source.consume(want_to_collect)
            self.food_carried += taken
            if taken > 0:
                self.eating_timer = EAT_DURATION
        elif self.food_carried > 0 and at_nest:
            deposited = self.food_carried
            self.food_carried = 0
            return deposited
        return 0.0
```

### `core/environment_bis.py`

```python
from core.pheromone_grid import PheromoneGrid
from config import *
import numpy as np

class Environment:

    def __init__(self, pheromone_grids, nest, food_grid, ants):
        self.pheromone_grids = pheromone_grids
        self.nest = nest
        self.food_grid = food_grid
        self.ants = ants

    def get_x_y(self):
        return self.nest.get_x_y()

    def update_environment(self):
        self.pheromone_grids.update_pheromone()
        self.food_grid.update()

    def step(self):
        self.update_environment()

        for ant in self.ants:
            delta_theta = np.random.uniform(-RANDOM_DIR, RANDOM_DIR)
            p_type = PheromoneGrid.FOOD if ant.food_carried > TRESHOLD_FOOD else PheromoneGrid.HOME

            (lx, ly), (rx, ry) = ant.get_antenna_pos()
            left_pheromone = self.pheromone_grids.get_pheromone(p_type, lx, ly)
            right_pheromone = self.pheromone_grids.get_pheromone(p_type, rx, ry)

            bias = ANTENNA_WEIGHT * (left_pheromone - right_pheromone)

            old_x, old_y = ant.move(delta_theta + bias)

            self.pheromone_grids.add_pheromones(p_type, old_x, old_y, PHEROMONE_DEPOSIT)

            source = self.food_grid.get_source(int(ant.x), int(ant.y))
            deposited = ant.interact(source, ant.is_at_nest(self.nest))
            if deposited > 0:
                self.nest.food_collected += deposited
```

---

## Observations et problèmes

La simulation est complètement chaotique. Une fourmi peut suivre une trace quelques instants puis l'abandonner. Aucune structure stable n'émerge.

Quatre problèmes principaux identifiés :

**Les fourmis ne savent pas faire demi-tour à la source.** Après `EAT_DURATION = 8` steps gelées, la fourmi repart dans la même direction qu'avant. Elle continue vers l'avant au lieu de rentrer au nid. Une solution possible : ajouter beaucoup de bruit aléatoire juste après la collecte pour augmenter la chance d'un demi-tour, ou inverser directement la direction.

**Les phéromones s'évaporent trop vite.** Avec `EVAPORATION_RATE = 0.997`, la demi-vie d'une trace est d'environ 231 steps, soit ~10 secondes à 24 FPS. Avec seulement 20 fourmis, une trace qui n'est pas renforcée en continu disparaît avant qu'une autre fourmi arrive. Réduire l'évaporation laisserait les traces exister assez longtemps pour être exploitées.

**Pas assez de fourmis.** 20 fourmis sur une grille de 360×240, c'est une densité trop faible. Plus de fourmis = plus de renforcement des traces = meilleure émergence. Pour passer à 100-200 fourmis sans surcharger l'affichage, il faudrait passer `CELL_SIZE` de 3 à 2 pixels, ce qui agrandit la grille utilisable à 540×360 cases.

**Bug critique : inversion du type de phéromone.** Dans `environment_bis.py`, une seule variable `p_type` est utilisée à la fois pour ce que la fourmi **suit** et pour ce qu'elle **dépose**. Or ces deux types doivent être opposés :

| État | Dépose | Doit suivre |
|---|---|---|
| A de la nourriture | `FOOD` | `HOME` |
| Sans nourriture | `HOME` | `FOOD` |

Actuellement, une fourmi qui vient de ramasser de la nourriture suit les phéromones `FOOD` — ce qui la ramène vers la source depuis laquelle elle vient de partir. Elle tourne en rond. C'est probablement la cause principale du chaos. Le fix :

```python
deposit_type = PheromoneGrid.FOOD if ant.food_carried > TRESHOLD_FOOD else PheromoneGrid.HOME
follow_type  = PheromoneGrid.HOME if ant.food_carried > TRESHOLD_FOOD else PheromoneGrid.FOOD
```

Une remarque aussi sur `ANTENNA_WEIGHT = π/3` : le biais maximum est d'environ 60°, auquel s'ajoute le bruit de 22°. Ça fait jusqu'à ±82° de rotation par step, ce qui est très violent et peut provoquer des oscillations plutôt qu'un suivi fluide. Ce paramètre devra être ajusté une fois le bug d'inversion corrigé.

---
