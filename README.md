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

**Deux mains, deux barres**, comme sur une vraie table. Chaque moitié d'écran est
une main : le pouce tient une poignée et en pilote les deux mouvements. Les deux
mains agissent **en même temps** — on bloque d'une main pendant qu'on arme de
l'autre, ce qui était impossible tant qu'on ne pilotait qu'une barre.

| Zone | Geste | Effet |
|---|---|---|
| Colonne de gauche | toucher | la barre tenue par la **main gauche** |
| Colonne de droite | toucher | la barre tenue par la **main droite** |
| **AUTO** | toucher | place les deux mains sur les barres les plus proches de la balle |
| Moitié d'écran | glisser ↕ | coulisser la barre de cette main |
| Moitié d'écran | glisser ↔ | la tourner. On s'arrête, elle reste ; on lâche, elle retombe en blocage |
| Moitié d'écran | vitesse du glissement | la puissance, qui n'est pas un réglage — elle sort de la vitesse à laquelle on tourne |

Chaque joueur voit la table depuis son propre côté : la vue de l'invité est
pivotée de 180°, donc les gestes restent absolus pour chacun et les deux
attaquent vers la droite de leur écran.

## La physique

L'objectif est de reproduire fidèlement une vraie table, pas d'approcher sa
sensation avec des raccourcis.

**Les courses.** Défense, demis et attaque balaient toute la largeur : la course
de chaque barre est calculée pour que la figurine extrême affleure la bande,
`span = H/2 − (n−1)·espacement/2 − demi-hauteur du pied`. Seul le gardien reste
bridé — il n'a que la largeur du but à couvrir (210–470 pour un but de 240–440).

Sans ça il restait plus de 100 mm morts le long de chaque bande latérale.

**Les proportions.** La figurine mesure 95 mm de l'axe de la barre au pied,
comme un vrai homme de babyfoot (~110 mm de haut, barre au sommet). Ce n'est pas
un détail : avec une figurine deux fois trop courte, les zones d'atteinte des
barres ne se recouvrent plus et le croisement des barres cesse de fonctionner.
Les barres rouges à 225 et 525 laissent un trou de 110 mm entre elles — trou que
la barre bleue à 375 couvre exactement. C'est la géométrie voulue.

**La figurine est un pendule.** La barre a deux degrés de liberté : elle coulisse
le long de son axe et elle **tourne** autour. Au repos la figurine pend sous son
propre poids — c'est la position de blocage. La chiquenaude ne fait qu'une chose :
lui donner de la vitesse angulaire. Elle décrit alors son arc, frappe la balle où
qu'elle la rencontre, et retombe.

Rien n'est appliqué à la balle : elle est frappée quand le pied arrive dessus, à
la vitesse qu'a ce pied à cet instant. Tout le reste en découle sans paramètre
inventé.

| Ce qui était un paramètre | Ce qui le remplace |
|---|---|
| une « portée » en millimètres | la géométrie du pied et la longueur de la figurine |
| une puissance de tir | la vitesse du bout du pied, soit `longueur × vitesse angulaire` |
| un collage binaire de la balle | l'adhérence tangentielle du pied |

**Une figurine trop haute laisse passer la balle dessous.** La hauteur du pied
vaut `L(1 − cos θ)` ; au-delà du diamètre de la balle, il n'y a plus de contact.
C'est ce qui rend le tour complet coûteux — et le tour complet est de toute façon
borné à ±180°, comme la roulette est interdite en compétition.

**La main tient la poignée.** La rotation n'est pas une impulsion : le pouce
donne l'angle en continu. La barre suit à la vitesse du poignet, s'arrête où l'on
s'arrête — on peut donc **lever un homme et l'y garder**, ce qui était impossible
tant que chaque rotation revenait toute seule. Main lâchée, le rappel gravitaire
amorti la ramène en position de blocage.

L'amortissement est exprimé en fraction du **critique** : 1 = retour franc sans
rebond, quelle que soit la longueur de figurine ou la pesanteur choisie.

Le bout du pied atteint 2090 mm/s à pleine vitesse de poignet.

**L'adhérence.** Au contact, la vitesse relative balle/pied est décomposée : le
choc normal fait rebondir, le frottement tangentiel entraîne. C'est ce qui permet
de balancer la barre pour aller chercher une balle sur le côté : elle suit le
pied, et quand la barre s'arrête, la balle s'arrête avec elle. Sans jamais être
collée.

**Le pied plat à angles arrondis.** Un disque dévie toujours le long du rayon,
donc rien n'est jamais franchement droit. Le rectangle à angles arrondis donne
une face plate qui renvoie droit et des angles qui ouvrent les diagonales.

**La balle morte.** Une balle lente n'est pas une balle morte : le critère est
qu'**aucune figurine ne puisse l'atteindre**, barre poussée à fond. Dans ce cas
elle roule doucement vers une figurine au bout de 3 s. Si elle est atteignable
mais que personne ne la joue, la relance n'intervient qu'au bout de 9 s, en
dernier recours. Derrière une barre de défense, elle revient **obligatoirement à l'équipe
qui défend ce but** — sinon l'adversaire hérite d'une balle devant un but dégarni
alors que le défenseur aurait dû la récupérer. C'est la règle ITSF de la balle
morte entre le but et la barre de 2.

Elle est nécessaire : la géométrie a de vraies zones mortes. Le gardien couvre
y 230–450 et la défense 110–570, donc une balle sous y = 110 près du fond n'est
atteignable par personne.

## L'IA (solo)

Elle pilote **deux barres**, les plus proches de la balle — les autres reviennent
au repos. Exactement ce que peut faire un joueur avec ses deux mains. Elle en
pilotait quatre avec un suivi parfait, ce qui la rendait imbattable.

Son niveau est un **temps de réaction**, pas une précision : elle vise la balle
telle qu'elle l'a perçue il y a un instant. Elle ne frappe pas non plus une balle
qui s'éloigne d'elle.

| Niveau | Réaction | Vitesse de barre (humain : 1500) |
|---|---|---|
| 0,05 | 440 ms | 75 mm/s |
| 0,45 (défaut) | 280 ms | 675 mm/s |
| 1,00 | 60 ms | 1500 mm/s |

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

Bouton **RÉG**, visible au menu et sur l'écran de fin de match — donc au moment
où l'on règle entre deux binômes. Ne change **qu'un seul paramètre à la fois**,
sinon tu ne sauras pas lequel a fait quoi.

| Ordre | Réglage | Ce qu'il décide |
|---|---|---|
| 1 | Vitesse angulaire max | la puissance des tirs, puisque la balle part à la vitesse du pied |
| 2 | Longueur de la figurine | la portée, la hauteur à laquelle elle se lève, et la vitesse du pied à angle égal |
| 3 | Adhérence du pied | si l'on peut mener la balle ou si elle fuit |
| 4 | Amortissement | 1 = critique, retour franc sans rebond. En dessous la figurine ballotte, au-dessus elle est molle |
| 5 | Pesanteur | la vivacité du rappel. À 1, c'est la pesanteur réelle |
| 6 | Restitution des bandes | la profondeur des bandes. Sans prévisualisation, un rebond qui semble arbitraire est un rebond dont on n'apprend rien |
| 7 | Vitesse pour puissance max | si l'élan est un vrai curseur sous le pouce ou un interrupteur tout-ou-rien |

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
