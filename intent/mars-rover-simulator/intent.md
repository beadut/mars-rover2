# Intent : Simulateur de commandes Mars Rover

Auteur : Non renseigné (Platform Engineer Rover).

## Problème

Avant d'envoyer des commandes au rover réel, l'équipe a besoin de pouvoir les tester sur un simulateur afin de valider leur comportement sans risque.

## Résultat proposé

Un simulateur qui :
- reçoit un point de départ (x, y), une orientation (N, S, E ou W), une carte plaçant les obstacles et une liste de commandes exprimées en mots ("Avance", "Recule", "Tourne à droite", "Tourne à gauche") ;
- fait avancer, reculer ou tourner le rover de 90° à droite ou à gauche selon les commandes ;
- laisse le rover immobile lorsqu'un obstacle bloque son avancée ;
- prend en charge des cartes utilisant les symboles 🟩 (zone forestière) / 🌳 (arbre) ou 🟫 (zone rocailleuse) / 🪨 (roche) ;
- arrête le rover et signale qu'il est au bord de la carte lorsqu'il l'atteint ;
- affiche en sortie le trajet parcouru sur la carte, au format ASCII art.

## Utilisateurs et systèmes concernés

Les ingénieurs de la tour de contrôle, qui utilisent le simulateur pour préparer les commandes envoyées à distance au rover.

## Contraintes

- Langage : Rust.
- Carte de taille limitée à 1 km² pour le MVP.
- Un message d'alerte est affiché et le rover s'arrête lorsqu'il atteint un bord de la carte.
- Format d'entrée : point de coordonnées (x, y), orientation (N, S, E ou W), carte plaçant les obstacles, liste de commandes en mots.
- Format de sortie : trajet affiché sur la carte en ASCII art.

## Questions ouvertes

_Aucune à ce stade._
