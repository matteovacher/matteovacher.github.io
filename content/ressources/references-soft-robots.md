+++
date = '2026-04-16T16:32:51+02:00'
draft = true
title = 'References Soft Robots'
+++

## Co-evolving morphology and control of soft robots using a single genome

Fabio Tanaka

Ce papier est le papier sur lequel je vais me baser pour mes recherches sur les soft robots et sur co evoluer le corps et le controller, je vasi utiliser la meme methode. 

c'est un cppn evoler par neat qui produit deux ann, l'un pour le corps et l'autre pour le cerveau. c'est ca hyper neat. 

l'ann du corps est simple, il prend la position et dit quoi mettre, l'ann du cerveau est different, des valeurs sont enregistré (celle produites par les capteurs) et il y a une sortie par voxel. 

Neat utilise une fonction qui mesure la distance genotypique or ici cppn c'est notre genome donc celui ci ne determine pas le corps entierement ie deux genome oproche peuvent donner un corps tres differents, donc dans le but de mieux classer en espece les individus, on utilise ici la somme de cette distance et celle du phenotype, voir papier. 

j'ai des idées de comment changer ce hyper neat avec la lecture du papier et aussi avec moi ce que j'aime dans la vie, je n'ai pas encore lu totalement les autres, mais deja : 

- on peut utiliser hyperneat pour generer des ann mais beaucoup plus, ie le premier s'aoccuperait d'une partie du design du corps, disons la souche exterieur et le deuxieme, la couche interieur, le cerveau quand a lui pourrait voir plusieurs de ses voisins, on pourrait imaginer egalement une division des centres de controles, le but etant d'avoir un controler pour toute une region et de meme pour les autres. ensuite on remonte a un plus gros controller qui voit en quelques sorte toute les infos comme le cerveau. 
- Une piste aussi interessante mais difficile a monter serait de changer la structure des reseaux apres le cppn, deja, on pourrait mettre un snn apres pour le controller, pour reproduire plus la nature. 

une autre chose a faire, serait de partir de cette base pour se rapprocher de ce que font les cellules du corps humain ie activer certains genes et en silencier d'autres mais du peu que je viens de rechercher je comprends rien. truc onteressant cepandant 1. active les genes, 2. eteint les autres, 3 maintiens l'etat de maniere hereditaire. 

design un troisieme ann qui chosit les frontieres, un point va dans telle ou telle zone mais avec comme regles de ils doivent etre collés. ca complexifie mais ca peut etre une idée, ensuite par chacune des zones fait la fitness et bonus si les zones correspondes ou autre. 

aussi utiliser une fonction cout exemple produire des puscles demande plus d'energie 
