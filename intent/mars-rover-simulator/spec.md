# Spec : Simulateur de commandes Mars Rover

Intention de référence : [intent/mars-rover-simulator/intent.md](./intent.md)

## Périmètre

Le simulateur permet aux ingénieurs de la tour de contrôle de tester des commandes avant de les envoyer au rover réel. Il reçoit en entrée un point de départ (x, y), une orientation (N, S, E ou W), une carte plaçant des obstacles et une liste de commandes en mots ("Avance", "Recule", "Tourne à droite", "Tourne à gauche"), puis produit en sortie le trajet parcouru sur la carte au format ASCII art.

Le MVP se limite à une carte de taille maximale 1 km². Aucune exclusion de périmètre n'est mentionnée dans l'intention.

## Exigences

### EX-01 — Initialisation de la position et de l'orientation du rover

Origine dans l'intention : « reçoit un point de départ (x, y), une orientation (N, S, E ou W) »
Comportement attendu : le simulateur initialise le rover à la position (x, y) et à l'orientation fournies en entrée, avant l'exécution de toute commande.

Scénario
- Situation de départ : une position de départ (x, y) valide sur la carte et une orientation parmi N, S, E, W sont fournies avec la carte.
- Action : le simulateur charge ces paramètres.
- Résultat attendu : le rover est positionné en (x, y) avec l'orientation donnée, prêt à recevoir la liste de commandes.

### EX-02 — Avance et recul du rover

Origine dans l'intention : « fait avancer, reculer [...] le rover [...] selon les commandes » ; commandes « Avance », « Recule ».
Comportement attendu : la commande « Avance » déplace le rover d'une case dans le sens de son orientation courante ; la commande « Recule » le déplace d'une case dans le sens opposé. L'orientation du rover reste inchangée dans les deux cas.

Scénario
- Situation de départ : le rover est en (x, y), orienté N, et la case suivante dans la direction de déplacement est libre et à l'intérieur de la carte.
- Action : la commande « Avance » est exécutée.
- Résultat attendu : le rover se déplace d'une case dans la direction N ; son orientation reste N.

### EX-03 — Rotation du rover

Origine dans l'intention : « tourne [...] de 90° à droite ou à gauche » ; commandes « Tourne à droite », « Tourne à gauche ».
Comportement attendu : la commande « Tourne à droite » fait pivoter l'orientation du rover de 90° dans le sens horaire ; « Tourne à gauche » la fait pivoter de 90° dans le sens antihoraire. La position du rover reste inchangée.

Scénario
- Situation de départ : le rover est en (x, y), orienté N.
- Action : la commande « Tourne à droite » est exécutée.
- Résultat attendu : le rover reste en (x, y) et son orientation devient E.

### EX-04 — Blocage par un obstacle

Origine dans l'intention : « laisse le rover immobile lorsqu'un obstacle bloque son avancée » ; symboles 🌳 (arbre) et 🪨 (roche).
Comportement attendu : lorsqu'une commande « Avance » ou « Recule » désignerait une case contenant un obstacle (🌳 ou 🪨), le rover reste immobile à sa position actuelle.

Scénario
- Situation de départ : le rover est en (x, y), orienté N, et la case en (x, y+1) contient un obstacle (🌳 ou 🪨).
- Action : la commande « Avance » est exécutée.
- Résultat attendu : le rover reste en (x, y) ; seule cette commande de déplacement est annulée, la commande suivante de la liste s'exécute normalement (décision R-01).

### EX-05 — Arrêt au bord de la carte

Origine dans l'intention : « arrête le rover et signale qu'il est au bord de la carte lorsqu'il l'atteint » ; contrainte « Un message d'alerte est affiché et le rover s'arrête lorsqu'il atteint un bord de la carte. »
Comportement attendu : lorsque le rover atteint un bord de la carte, le simulateur affiche un message d'alerte signalant cette situation et arrête le rover.

Scénario
- Situation de départ : le rover est en (x, y), orienté N, et (x, y) est la dernière case valide avant le bord de la carte dans cette direction (ou une commande « Avance » ferait sortir le rover de la carte).
- Action : la commande « Avance » est exécutée.
- Résultat attendu : un message d'alerte signalant le bord de carte est affiché ; le rover reste en (x, y) pour cette seule commande, et la commande suivante de la liste s'exécute normalement (décision R-01).

### EX-06 — Cartes avec zones et obstacles symbolisés

Origine dans l'intention : « prend en charge des cartes utilisant les symboles 🟩 (zone forestière) / 🌳 (arbre) ou 🟫 (zone rocailleuse) / 🪨 (roche) ».
Comportement attendu : le simulateur interprète 🟩 et 🟫 comme des cases franchissables, et 🌳 et 🪨 comme des obstacles bloquant le passage.

Scénario
- Situation de départ : une carte contenant des cases 🟩 et des cases 🌳 est fournie.
- Action : le simulateur charge la carte.
- Résultat attendu : le rover peut se déplacer librement sur les cases 🟩 ; toute case 🌳 est traitée comme un obstacle selon EX-04. Une même carte peut mêler des zones 🟩/🌳 et 🟫/🪨 : le simulateur reconnaît les deux jeux de symboles simultanément (décision R-02).

### EX-07 — Exécution séquentielle d'une liste de commandes en mots

Origine dans l'intention : « une liste de commandes exprimées en mots ("Avance", "Recule", "Tourne à droite", "Tourne à gauche") ».
Comportement attendu : le simulateur reçoit une liste ordonnée de commandes exprimées avec ces quatre libellés et les exécute dans l'ordre.

Scénario
- Situation de départ : la liste de commandes ["Avance", "Avance", "Tourne à droite"] est fournie, sans obstacle ni bord de carte rencontré.
- Action : le simulateur exécute la liste.
- Résultat attendu : les trois commandes sont appliquées dans l'ordre au rover, chacune produisant l'effet décrit par EX-02 ou EX-03.

### EX-08 — Affichage du trajet en ASCII art

Origine dans l'intention : « affiche en sortie le trajet parcouru sur la carte, au format ASCII art » ; contrainte « Format de sortie : trajet affiché sur la carte en ASCII art. »
Comportement attendu : à l'issue de l'exécution de la liste de commandes, le simulateur affiche la carte avec le trajet parcouru par le rover, dans un format ASCII art (caractères ASCII, non les symboles emoji utilisés en entrée).

Scénario
- Situation de départ : l'exécution de la liste de commandes est terminée (normalement ou arrêtée selon EX-05).
- Action : le simulateur produit la sortie.
- Résultat attendu : la carte est affichée en ASCII art, montrant les cases traversées par le rover et sa position/orientation finale. Le jeu de caractères exact utilisé pour représenter terrain libre, obstacles, trajet et rover est une proposition de conception (voir « Conception proposée »).

### EX-09 — Taille de carte limitée pour le MVP

Origine dans l'intention : « Carte de taille limitée à 1 km² pour le MVP. »
Comportement attendu : le simulateur accepte des cartes dont la superficie représentée ne dépasse pas 1 km².

Scénario
- Situation de départ : une carte dont la superficie représentée est inférieure ou égale à 1 km² est fournie.
- Action : le simulateur charge la carte.
- Résultat attendu : la carte est acceptée. Une case de la carte correspond à 1 mètre au sol ; la carte doit donc rester dans une grille d'au plus 1000 cases sur 1000 cases pour respecter la limite de 1 km² (décision R-03).

## Conception proposée

Choix déjà fixés par l'intention (contraintes) :
- Implémentation en Rust.
- Commandes reconnues : exactement les quatre libellés « Avance », « Recule », « Tourne à droite », « Tourne à gauche ».
- Symboles d'entrée de la carte : 🟩/🌳 pour une zone forestière, 🟫/🪨 pour une zone rocailleuse, pouvant coexister sur une même carte (décision R-02).
- Sortie en ASCII art (donc distincte des symboles emoji d'entrée).
- Échelle de la carte : une case représente 1 mètre au sol, pour une grille d'au plus 1000 × 1000 cases (décision R-03).
- Un obstacle ou un bord de carte n'annule que la commande de déplacement en cours ; les commandes suivantes de la liste s'exécutent normalement (décision R-01).

Propositions à valider par le Product Owner :
- Représentation interne de la carte : grille 2D indexée par (x, y), chaque case étant soit franchissable soit un obstacle, dérivée des symboles d'entrée.
- Convention d'orientation : N/S/E/W associés à des déplacements (dx, dy) fixes sur la grille (par exemple N = y+1, E = x+1) ; à confirmer, car l'intention ne précise pas l'orientation des axes.
- Jeu de caractères ASCII pour la sortie : par exemple `.` pour une case franchissable non visitée, `#` pour un obstacle, `*` pour une case visitée, et un caractère directionnel (`^`, `v`, `<`, `>`) pour la position et l'orientation finales du rover.
- Interface d'entrée/sortie du simulateur (ligne de commande, fichier, entrée standard) : non précisée dans l'intention ; à définir en phase Build une fois la structure des données validée ici.

## Réserves

### R-01 — Portée de l'arrêt du rover (obstacle et bord de carte)

Origine : Résultat proposé (« laisse le rover immobile lorsqu'un obstacle bloque son avancée » et « arrête le rover [...] lorsqu'il l'atteint ») et contrainte associée au bord de carte.
Exigences concernées : EX-04, EX-05, EX-07.
Conséquences : l'intention ne précisait pas si, après un blocage par obstacle ou un arrêt au bord de carte, le simulateur continue d'exécuter les commandes suivantes de la liste, ou s'il arrête entièrement l'exécution de la liste restante.
Décision : un obstacle ou un bord de carte n'annule que la commande de déplacement en cours ; le rover reste immobile pour cette commande, mais les commandes suivantes de la liste (déplacements ou rotations) s'exécutent normalement.
Auteur : Product Owner (répondu via la session de rédaction de cette spécification).
Date : 2026-09-22.
Justification : non détaillée par le Product Owner au-delà du choix de l'option correspondante.
Éléments modifiés : EX-04, EX-05, « Conception proposée ».
Statut : décidée.

### R-02 — Mélange des types de zones sur une même carte

Origine : Résultat proposé (« prend en charge des cartes utilisant les symboles 🟩 [...] ou 🟫 [...] »).
Exigences concernées : EX-06.
Conséquences : l'intention n'indiquait pas si une carte peut mélanger des cases 🟩/🌳 et 🟫/🪨, ou si chaque carte n'utilise qu'un seul jeu de symboles.
Décision : une même carte peut mélanger les deux types de zones ; l'analyseur de carte doit reconnaître les deux jeux de symboles simultanément.
Auteur : Product Owner (répondu via la session de rédaction de cette spécification).
Date : 2026-09-22.
Justification : non détaillée par le Product Owner au-delà du choix de l'option correspondante.
Éléments modifiés : EX-06, « Conception proposée ».
Statut : décidée.

### R-03 — Échelle de la carte par rapport à la limite de 1 km²

Origine : contrainte « Carte de taille limitée à 1 km² pour le MVP. »
Exigences concernées : EX-09.
Conséquences : sans échelle définie, il n'était pas possible de vérifier qu'une carte donnée respecte effectivement la limite de 1 km², ni de dimensionner les structures de données en conséquence.
Décision : une case de la carte représente 1 mètre au sol ; la carte doit donc rester dans une grille d'au plus 1000 × 1000 cases pour respecter la limite de 1 km².
Auteur : Product Owner (répondu via la session de rédaction de cette spécification).
Date : 2026-09-22.
Justification : non détaillée par le Product Owner au-delà du choix de l'option correspondante (échelle définie, exemple 1 case = 1 mètre).
Éléments modifiés : EX-09, « Conception proposée ».
Statut : décidée.

## Questions ouvertes

L'intention ne recense aucune question ouverte (« Aucune à ce stade. »). Aucune question de l'intention n'est donc à suivre ici. Les ambiguïtés identifiées lors de la rédaction de cette spécification sont consignées dans la section « Réserves » ci-dessus, et non ici, puisqu'elles ne proviennent pas de l'intention elle-même.

## Contexte de génération

### Demande initiale

Commande `/spec intent/mars-rover/intent.md`. Remarque : ce chemin ne correspond à aucun fichier du dépôt ; l'unique intention disponible se trouve à `intent/mars-rover-simulator/intent.md`, qui a été utilisée pour cette spécification.

### Skills utilisées

| Chemin | Commit Git de la version utilisée |
| --- | --- |
| `.claude/skills/spec/SKILL.md` | `6bd785d62288c878d7e05e4ceac9b30083952d9e` |

### Révisions

_Aucune révision à ce stade._
