# Babyfoot — prototype de validation

Prototype jouable d'un babyfoot mobile **en temps réel**, jusqu'à deux joueurs
chacun sur son téléphone. Il n'existe pas pour être un jeu : il existe pour
répondre à **une seule question**.

> Est-ce que deux personnes qui viennent de finir un match en redemandent un
> deuxième, puis un troisième — sans qu'on le leur propose ?

Tout le reste (comptes, progression, monétisation, clubs d'entreprise, direction
artistique) est délibérément hors périmètre.

## Tester

**1v1, chacun son téléphone** — les deux ouvrent la page, en paysage. L'un fait
*Créer une partie* et lit son code à quatre lettres, l'autre fait *Rejoindre* et
le saisit.

    https://alex-mant.github.io/babyfoot-proto/

Le même Wi-Fi marche toujours ; sur des réseaux différents la traversée de NAT
échoue parfois, faute de serveur TURN. Les deux messages d'erreur disent des
choses différentes : *« Aucune partie avec ce code »* = le courtier répond mais
l'hôte est introuvable ; *« Réseau indisponible »* = le courtier lui-même est
injoignable.

**APK Android** — onglet *Releases* → `babyfoot-proto.apk`, reconstruit à chaque
push sur `main`. Le mode en ligne y fonctionne.

**Solo contre l'IA** — partout, y compris dans l'aperçu web publié par Claude.
Attention : cet aperçu bloque toute connexion réseau sortante, donc **le 1v1 en
ligne n'y fonctionne pas**.

Match en 5 buts. Le score est incrusté en boules de compteur sur la traverse.

## Les contrôles

| Geste | Effet |
|---|---|
| Colonne de gauche | sélectionner la barre — rangées dans leur ordre spatial sur le terrain |
| **AUTO** | la barre se choisit seule. Elle ne se **déplace** jamais seule : toucher une barre à la main coupe le mode |
| **Glisser ↕** n'importe où | faire coulisser la barre sélectionnée (indirect : le doigt ne masque pas la table) |
| **Claquer ↔** | frapper, vers la droite ou vers la gauche |
| Inclinaison du claquement | choisit la zone du pied : à plat, angle haut, angle bas |
| Vitesse du claquement | puissance du tir |

Chaque joueur voit la table depuis son propre côté : la vue de l'invité est
pivotée de 180°, donc les gestes restent absolus pour chacun et les deux
attaquent vers la droite de leur écran.

## Les mécaniques

**La portée.** Le pied ne touche la balle que si elle est réellement à sa portée.
Claquer plus loin tape dans le vide — c'est ce qui empêche un gardien de frapper
une balle à l'autre bout de la table.

**Le contrôle de balle**, réglable en trois régimes. Un simple seuil de vitesse
rendait le collage imprévisible : on ne savait pas pourquoi ça avait collé cette
fois et pas la précédente, donc on n'apprenait rien.

| Régime | Comportement |
|---|---|
| **toujours** (défaut) | tout contact bloque la balle. Toucher un homme, c'est lui donner la balle ; viser les intervalles devient tout le jeu |
| jamais | aucun collage, physique pure |
| selon la vitesse | sous le seuil ça bloque, au-dessus ça rebondit |

Les intervalles laissent passer la balle partout : 76 mm aux demis, 141 mm à
l'attaque, 186 mm en défense, pour une balle de 35 mm.

**L'angle se choisit, il ne se subit pas.** Balle tenue, la balle est collée : le
glissement ne sert qu'à se placer en travers du terrain. L'angle vient de
l'inclinaison de la chiquenaude — trois zones franches, un seul geste. Tant que
la balle est tenue, les trois options sont affichées en éventail.

Auparavant l'angle dérivait comme effet de bord du déplacement de la barre :
impossible à maîtriser, puisqu'on ne le choisissait pas.

**Le pied plat à angles arrondis.** Un disque dévie toujours le long du rayon,
donc rien n'est jamais franchement droit. Le rectangle à angles arrondis donne
une face plate qui renvoie droit et des angles qui ouvrent les diagonales. Le
rayon d'arrondi est réglable : petit = très binaire, grand = retour vers le disque.

**Le report de la barre sur la balle.** La figurine est un corps cinématique :
une barre qui coulisse pousse la balle libre, et au tir la balle hérite de la
vitesse latérale de la barre. Coulisser puis claquer pendant le mouvement, c'est
le tir tiré.

**La balle morte.** Immobile plus de 2,5 s, elle roule doucement vers une
figurine, assez lentement pour être contrôlée à l'arrivée. Derrière une barre de
défense, elle revient **obligatoirement à l'équipe qui défend ce but** — sinon
l'adversaire hérite d'une balle devant un but dégarni alors que le défenseur
aurait dû la récupérer. C'est la règle ITSF de la balle morte entre le but et la
barre de 2.

Elle est nécessaire : la géométrie a de vraies zones mortes. Le gardien couvre
y 230–450 et la défense 110–570, donc une balle sous y = 110 près du fond n'est
atteignable par personne.

## Prévisualisation

Les traits pointillés montrent la **direction** du tir, et rien d'autre. Aucun
rebond n'est jamais calculé ni affiché : une prévisualisation de trajectoire
offrirait gratuitement au débutant ce que l'expert met vingt matchs à
intérioriser, ce qui est disqualifiant pour un jeu dont la prémisse est la
compétition.

La direction, elle, est de la géométrie au présent, pas une prédiction — et elle
enseigne la forme du pied. L'apprentissage des bandes se fait **après** le tir :
la balle laisse une traînée.

## Régler

Bouton **RÉG**. Ne change **qu'un seul paramètre entre deux binômes**, sinon tu
ne sauras pas lequel a fait quoi.

| Ordre | Réglage | Ce qu'il décide |
|---|---|---|
| 1 | Vitesse pour puissance max | si la puissance est un vrai curseur sous le pouce ou un interrupteur tout-ou-rien |
| 2 | Contrôle de balle | le régime de collage, donc tout le rythme des échanges |
| 3 | Puissance max | la vitesse de la balle. À 4600 mm/s elle traversait la table en 260 ms, sous le temps de réaction humain : injouable |
| 4 | Restitution des murs | la profondeur des bandes. Sans prévisualisation, un rebond qui semble arbitraire est un rebond dont on n'apprend rien |
| 5 | Arrondi des angles | la zone plate contre les diagonales |
| 6 | Portée du pied | si on tape trop souvent dans le vide, ou si ça accroche tout |

## Mesurer

Bouton **Journal**, dans le panneau de réglages :

- chiquenaudes, dont **coups dans le vide** et **accidentelles** — cible **< 5 %**
- **contrôles**, dont passes réussies
- **revanches** consécutives — cible **médiane ≥ 3**
- **temps jusqu'au premier but** — cible **< 3 min**

## Critères d'arrêt

À assumer avant de commencer, pas à assouplir après les tests.

1. Chaîne médiane de revanches **inférieure à 2**.
2. Aucune progression de précision mesurable entre la session 1 et la session 2 —
   pas de courbe de compétence, donc pas de jeu compétitif possible.
3. Temps jusqu'au premier but **supérieur à 5 minutes** pour la majorité des
   testeurs — le contrôle est insoluble sous cette forme.

## Structure

    index.html       le prototype, servi par GitHub Pages
    www/index.html   même fichier, source de l'APK
    game.html        même source sans l'enveloppe <html> (aperçu web)
    capacitor.config.json
    .github/workflows/android.yml   construit l'APK et publie la release

Le jeu est un fichier unique. Sa seule dépendance externe est PeerJS, chargé
depuis cdnjs pour le WebRTC du mode 1v1 ; le solo fonctionne sans elle.

## Netcode

Hôte autoritaire, synchronisation à 60 Hz. L'invité prédit ses propres barres en
local et reçoit le reste : ballon, barres adverses, score, phase. Pas de
rollback — c'est le modèle simple et correct pour un réseau de bureau, pas pour
du jeu compétitif à distance.
