+++
date = '2026-03-24T15:51:14+01:00'
draft = true
title = 'First of the Project'
+++

brouillon en fr : 

pygame — la fenêtre de simulation. C'est lui qui affiche la grille, les fourmis qui bougent, les trainées de phéromones en temps réel. Sans lui, la simulation tourne mais tu ne vois rien. je rajoute que c'est mon environnment et que je vais devoir le modif 

numpy — le moteur de calcul de la grille. La carte des phéromones est un tableau 2D numpy. Toutes les opérations dessus (évaporation, dépôt, lecture) sont vectorisées, donc rapides sans boucles Python. pareil je vais utiliser des noyaux de conv ou autre pour pheromones 

torch — le cerveau des fourmis. C'est lui qui fait tourner le sous-réseau de neurones. Il permet de traiter toutes les fourmis en un seul calcul batch au lieu de les traiter une par une. je vais devoir amenager mon reseau de neurone 

scipy — la physique des phéromones. Les phéromones se diffusent dans l'air comme un gaz. scipy simule ça avec un filtre gaussien sur la grille numpy, en une seule ligne de code. la scipy intervient sur les pheromones 

matplotlib — le journal de l'évolution. Il tracera les courbes de performance de MAP-Elites : est-ce que la colonie devient meilleure au fil des générations ? 

J'ai mis en place l'environnement de developpement pour mon projet de simulation de fourmis. Le projet tourne sur Python 3.10 avec un environnement conda isole. Les outils choisis sont : pygame pour l'affichage, numpy et scipy pour les calculs de pheromones, pytorch pour le reseau de neurones des fourmis, et matplotlib pour visualiser l'evolution genetique. La contrainte principale est la puissance limitee de la machine, ce qui oriente toutes les decisions techniques vers l'efficacite CPU.
