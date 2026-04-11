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

# GRID AND PHEROMONE 

EVAPORATION_RATE = 0.997                # between 0 and 1, the higher the slower the evaporation
DIFFUSION_SIGMA = 0.3                   # between 0 and inf, the higher the more the pheromone spreads, but also the more it evaporates
GRID_WIDTH = 360                        # width of the grid in cells, also in pixels if cell size is 1, max 2880 for 8k screen
GRID_HEIGHT = 240                       # height of the grid in cells, also in pixels if cell size is 1, max 1920 for 8k screen
CELL_SIZE = 3                           # size of each cell in pixels
FPS = 24                                # frames per second
WINDOW_WIDTH = GRID_WIDTH*CELL_SIZE     # width of the window = width*cellsize (2880 pixels max )
WINDOW_HEIGHT = GRID_HEIGHT*CELL_SIZE   # height of the window = height*cellsize (1920 pixels max )
PHEROMONE_DEPOSIT = 0.7                 # amount of pheromone deposited by an ant at each step, between 0 and 1

# ENVIRONMENT
COLOR_HOME = (139, 90, 43)              # brown color for the pheromone leading to the nest
COLOR_FOOD = (50, 205, 50)              # green color for the pheromone leading to the food
COLOR_BACKGROUND = (0, 0, 0)            # black color for the background
COLOR_NEST = (255, 255, 255)            # white color for the nest
NEST_RADIUS = 2                         # radius of the nest in cells
NEST_X = 100                            # coordinates of the nest
NEST_Y = 100                            # coordinates of the nest

# ANT
N_ANTS = 20                             # number of ants in the simulation, must be an integer greater than 0
LENGTH_ANTENNA = 0.5                    # length from the head to the tip of the antenna in cells, must be greater than 0
ANGLE_ANTENNA = np.pi/4                 # angle between the direction of the ant and the direction of the antenna in radians, between 0 and pi/2, if 0 then antennas are in the same direction as the ant, if pi/2 then antennas are perpendicular to the direction of the ant
COLOR_ANT = (255, 165, 0)               # orange color for the ants
MAX_FOOD_CARRIED = 0.5                  # maximum amount of food an ant can carry, between 0 and 1, if 0.5 then an ant can carry half of a food source
FOOD_COLLECT_AMOUNT = 0.5               # amount of food an ant can collect at one time, between 0 and MAX_FOOD_CARRIED
EAT_DURATION = 8                        # number of steps an ant needs to eat a food source, during this time the ant cannot move or interact with other food sources, must be an integer
ANTENNA_WEIGHT = np.pi/3                # weight of the pheromone bias on the ant's direction, between 0 and pi/2, if 0 then the ant ignores the pheromones, if pi/2 then the ant turns directly towards the strongest pheromone
TRESHOLD_FOOD = 0.45                    # threshold of food carried for an ant to switch from following home pheromone to following food pheromone, between 0 and MAX_FOOD_CARRIED
RANDOM_DIR = np.pi/8                    # maximum random change in direction for an ant at each step, between 0 and pi, if 0 then the ant never changes direction randomly, if pi then the ant can turn in any direction at each step
HALF_LENGTH_BODY = 0.5                  # half of the length of the ant's body in cells, used for drawing the ant as a line before adding the antenna 

# FOOD SOURCES 
N_FOOD_TYPES = 2                        # number of different types of food sources, must be an integer greater than 0
COLOR_APHID = (255, 220, 0)             # color yellow for aphids, which are a type of food source that can recharge 
COLOR_SUGAR = (100, 200, 255)           # color blue for sugar, which is another type of food source that does not recharge
RECHARGE_RATE_APHID = 0.01              # recharge rate for aphids, between 0 and 1
RECHARGE_RATE_SUGAR = 0                 # recharge rate for sugar, must be 0 since sugar does not recharge

```

---

## Observations et problèmes

La simulation est complètement chaotique. Une fourmi peut suivre une trace quelques instants puis l'abandonner. Aucune structure stable n'émerge.

Quatre problèmes principaux identifiés :

**Les fourmis ne savent pas faire demi-tour à la source.** Après `EAT_DURATION = 8` steps gelées, la fourmi repart dans la même direction qu'avant. Elle continue vers l'avant au lieu de rentrer au nid. Une solution possible : ajouter beaucoup de bruit aléatoire juste après la collecte pour augmenter la chance d'un demi-tour, ou inverser directement la direction.

**Les phéromones s'évaporent trop vite.** Avec `EVAPORATION_RATE = 0.997`, la demi-vie d'une trace est d'environ 231 steps, soit ~10 secondes à 24 FPS. Avec seulement 20 fourmis, une trace qui n'est pas renforcée en continu disparaît avant qu'une autre fourmi arrive. Réduire l'évaporation laisserait les traces exister assez longtemps pour être exploitées.

**Pas assez de fourmis.** 20 fourmis sur une grille de 360×240, c'est une densité trop faible. Plus de fourmis = plus de renforcement des traces = meilleure émergence. Pour passer à 100-200 fourmis sans surcharger l'affichage, il faudrait passer `CELL_SIZE` de 3 à 2 pixels, ce qui agrandit la grille utilisable à 540×360 cases.

Une remarque aussi sur `ANTENNA_WEIGHT = π/3` : le biais maximum est d'environ 60°, auquel s'ajoute le bruit de 22°. Ça fait jusqu'à ±82° de rotation par step, ce qui est très violent et peut provoquer des oscillations plutôt qu'un suivi fluide. 


REMARQUE INDEPENDANTE 
J'ai egalement ajouté un nouveau modele pour les antennes, l'ancien avec une longueur d'antenne de 1 qui part du centre de la fourmi ne marchait pas a cause de l'angle des antenne qui se retrouvait alors faussé, j'ai rajouté dans les calculs la demi longueur du corps qui premet aux antennes de bien partir de la tete. 

**Ajustements des paramètres pour favoriser l'émergence.** Pour essayer de voir apparaître les premières structures collectives, plusieurs paramètres ont été modifiés : `EVAPORATION_RATE` passe à `0.999`, `N_ANTS` à 50, `PHEROMONE_DEPOSIT` à `0.8`. Le `NEST_RADIUS` a également été augmenté pour faciliter le retour au nid — une fourmi qui passe près du nid sans le déclencher ne dépose pas sa nourriture, ce qui casse le renforcement des traces HOME. Un rayon plus grand augmente le taux de succès des retours et devrait améliorer la cohérence des pistes émergentes. de plus antenna weight passe a pi/6 et random dir passe a pi/10 pour favoriser le retour au nid. aussi rayon nid = 10
RESULTAT ? la simulation recolte plus, mais toujours pas assez. et aucun chemin n'apparait, comme elles ne fond pas demi tour la map se rempli de verre, j'enregistre la video en fps 60 pour accelerer le rendu 


## Suivant 

j'ai mis un diffusion rate a 0.4 pour que ca difuse plus et fasse plus gradient, vu que les fourmis se base dans notre cas sur le gradient ca devrait les aider. 
en plus je vais leur faire demi tour lorque elle ont assez mangé pour leur donner une chance de suivre leur ancienne piste. c'est ajouté dans la methode move de la classe Ant. diminuer diffusion rate a 0.35 et truncate a 4.0 pour voir si ca va mieux car ca diffusait trop. le score etait neanmoins plus elevé. Elle se debrouille mieux mais de pas grand chose, pour mieux visualiser je vais tenter de separer la fenetre en double fentre pour observer les deux pheromones. 

## Suivant 

on va modifier l'affichage pour que : 
- on puisse switcher entre les differents affichage : 
en 1 : le max est affiché 
en 2 : juste home 
en 3 : juste food 

c'est fait, on verra le rendu dans la prochaine video qui sortira en meme temps que toute les autres updates. 

## Suivant

On bloque alors on repart du modele pour l'amliorer, mon modele fonctionne sur un gradient de pheromone, hors ce gradient n'est pour linstant utilisé seulement pour faire des gauche droite, on va donc essayer de l'utiliser pour plus guider la fourmis, pour cela, on va faire en sorte que plus la fourmis s'eloigne en nombre de pas, moins elle depose. chez les fourmis, on compte ses pas, Collett cite Wittlinger. On va donc partir sur une diminution de pheromone au fur et a mesure. En plus de cela, on va donner un boost de direction au retour au nid, quand la fourmi retourne au nid elle va faire demmi tour et la random dir sera plus grande pour favoriser l'exploration. mais tourjours avoir la possibilité de revenir sur la piste de nourriture, on va egalement augmenter la diffusion pour permettre au fourmis de mieux detecter le gradient. On va egalement auglenter le pheromone deposit pour partir de 1 puisque mtn celui ci diminue avbev le temps. on va egalement augmenter le antenna weight pour renforcer le suivi. On va egalement augmenter le nombre de fourmis pour tenter de mieux voir des phenomenes de groupe. 
on rajoute aussi une difference de evaporation rate entre les deux type de pheromone, car au bout d'un moment toute les fourmis sont vertes (je sais pas si je l'ai dis mais les fourmis qui ont recolté la nourriture sont verte maintenant) donc il faut que la pheromone marron ie home dure plus longtemps 

RESULTATS ? ca ne marche pas pafaitement, ca marche mieux mais pas encore bien, 

## SUIVANT 

pour resoudre le probleme, on va discretiser le systeme de pheromone, on va interpoler les valeurs de toutes les cases autour, en plus de ca on va modifier la grille et son fonctionnement, les fourmis seront plus grosses que les cases pheromones, ainsi 1 case sera 1 pixels et une fourmis pourra enb faire, avec ces deux choses, on espere voir une amelioration 
j'ai egalement augmenté la taille d'un peu tout et desormais si la fourmis, son centre est dans le cerche de la nourriture alors elle est considéré comme dessus et la recolte. en augmentant le antenna weight ca ne marche pas, pas besoin de video, c'est sensiblement la meme qu'avant les fourmis sortent des pistes dans les virages 

## SUIVANT 
On essaye encore, cette fois on se dit que les antennes sont trop loin, je separe donc le rendu visuelle de la simulation physique ie les fourmis apparaissent plus grosses qu'elle ne sont dans la simulation. Je diminue le antenna weight pour eviter de trop tourner en virage et ensuite je clip la valeur de total_theta si elle est trop grande. j'ai l'impression que les gradients sont inexistant et donc aucune direction n'apparait, dommage pour un systeme de detection qui repose dessus 

mon gradient spatial semble trop faible alors je vais diminuer sigma diffusion et egalement le decay factor, j'augmente ensuite le evaporation rate car je veux quand meme que les trace loin reste un peu, si c'est trop je le diminuerai 

On va alors chnager le modele de la fourmi, je rajoute ce que je disais sur le gradient temporel, je vais prendre une 3e valeur un peu comme dans Draft et al. (2018)

Si la valeur devant est plus grande on reste tout droit (a la direction aleatoire généré pres), les fourmis suivent mieux, mais probleme, elles tournent encore plus autour du nid, mais beaucoup plus donc moins de nourriture recolté, pour ca je diminue l'influence de la diffusion car tout est satturé a 1 pret du nid et les fourmis ne se repere pas, et je change le add_pheromone pour pas le faire saturer en permanence au depot, je prends le max de current et value que je veux deposer. en plus de ca je reduis l'ecart de l'angle des antennes pour qu'il detecte mieux, avec ca j'ai egalement changé pas mal les valeurs de diffusion et evaporation, je vais rajouter deux type de diffusion comme cela je pourrait obtenir dans les deux cas un bon sigma pour chaque piste, apres discussion avec guy thorelaz, mon modele semble etre plus ou moins juste, disons que les pheromones deposé sont seulement pour food le reste c'est des hydrocarbure, odeur du corps ? je vais verif ca avec le papier sur les pheidoles


Après plusieurs journées d'itérations, voici où en est le code au moment où j'écris ces lignes.

**Grille et rendu.** La grille est maintenant à 960×640 pixels avec `CELL_SIZE = 1` — chaque case fait un pixel. C'est beaucoup plus grand qu'avant (360×240 / 3 px) mais ça permet d'avoir des traces fines et un rendu plus lisible. FPS = 45.

**200 fourmis.** Le passage à `N_ANTS = 200` donne enfin assez de masse pour renforcer les traces. Avec 20 ou 50 fourmis sur une grande grille, les traces s'évaporaient avant qu'une deuxième fourmi passe dessus.

**Évaporations différenciées.** `EVAPORATION_RATE_HOME = 0.999` et `EVAPORATION_RATE_FOOD = 0.996` — les traces HOME durent plus longtemps que les traces FOOD. C'est cohérent : les fourmis qui rentrent au nid ont besoin d'une piste stable, celles qui cherchent de la nourriture peuvent se permettre une piste plus volatile qui disparaît si personne ne la renforce.

**Diffusion réduite.** `DIFFUSION_SIGMA = 0.25` — on a diminué la diffusion par rapport aux tentatives précédentes. Trop diffuser annule le gradient spatial dont les antennes ont besoin pour détecter une direction.

**Modèle à 3 antennes finalisé.** `HALF_LENGTH_BODY = 1.5`, `LENGTH_ANTENNA = 1.5`. Les trois capteurs (gauche, droite, avant) sont stables — l'antenne avant sert à détecter si la concentration augmente devant la fourmi.

**Outil de calibration.** J'ai créé `evaluation_parameters.py` qui calcule automatiquement la demi-vie des phéromones, le sigma effectif, le SNR et le ratio séparation/largeur des antennes. Ça permet de vérifier en un coup d'œil si les paramètres sont dans une zone cohérente avant de lancer une simulation. ici c'est claude qui a fait ce fichier pour me fair egagner du temps dur la base de mes instructions. 

**Refactoring.** `environment_bis.py` sépare proprement la logique de simulation du rendu — l'orchestrateur ne touche plus à Pygame. L'ancien `environment.py` est conservé mais legacy.

Pour le moment ca marche bien 
il faut aussi rajuter un compteur totale de decay rate pour mesurer la porté max des fourmis pour adapter la taille de la map


---

## References


- Collett T., Graham P., Heinze S. (2025). The neuroethology of ant navigation.
- Wittlinger, Whener, Wolf (2006)