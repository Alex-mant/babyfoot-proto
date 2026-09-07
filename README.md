# Babyfoot — prototype de validation

Prototype jouable d'un babyfoot mobile **au tour par possession**. Il n'existe pas
pour être un jeu : il existe pour répondre à **une seule question** en quatorze jours.

> Est-ce que deux personnes qui viennent de finir un match en redemandent un
> deuxième, puis un troisième — sans qu'on le leur propose ?

Tout le reste (comptes, en ligne, IA, progression, monétisation, clubs
d'entreprise, direction artistique) est délibérément hors périmètre.

## Tester

**En 1v1, chacun son téléphone** — les deux ouvrent la page GitHub Pages, en
paysage. L'un fait *Créer une partie* et lit son code à quatre lettres, l'autre
fait *Rejoindre* et le saisit. Le même Wi-Fi marche toujours ; sur des réseaux
différents la traversée de NAT échoue parfois (pas de serveur TURN).

    https://alex-mant.github.io/babyfoot-proto/

**APK Android** — onglet *Releases* → `babyfoot-proto.apk`. Reconstruit à chaque
push sur `main`. Le mode en ligne y fonctionne aussi.

**Solo contre l'IA** — disponible partout, y compris dans l'aperçu web publié
par Claude. Attention : cet aperçu bloque toute connexion réseau sortante, donc
**le 1v1 en ligne n'y fonctionne pas**. Utilise GitHub Pages ou l'APK.

Match en 5 buts.

## Les contrôles

| Geste | Effet |
|---|---|
| Boutons de barre (colonne de gauche) | sélectionner la barre — rangés dans leur ordre spatial sur le terrain |
| **Glisser ↕** n'importe où | faire coulisser la barre sélectionnée (indirect : le doigt ne masque pas la table) |
| **Claquer ↔** | frapper — vers la droite ou vers la gauche, en repère écran |
| Vitesse du claquement | puissance du tir |

Le jeu est en **temps réel**. Chaque joueur voit la table depuis son propre côté :
la vue de l'invité est pivotée de 180°, donc les gestes restent absolus pour
chacun.

## Les quatre mécaniques

**La portée.** Le pied ne touche la balle que si elle est réellement à sa portée.
Claquer plus loin tape dans le vide. C'est ce qui empêche un gardien de frapper
une balle à l'autre bout de la table.

**Le contrôle de balle.** Réglable en trois régimes, parce qu'un seuil de vitesse
rend le collage imprévisible — on ne sait pas pourquoi ça a collé cette fois et
pas la précédente, donc on n'apprend rien.

| Régime | Comportement |
|---|---|
| **toujours** (défaut) | tout contact bloque la balle. Toucher un homme, c'est lui donner la balle ; viser les intervalles entre figurines devient tout le jeu. |
| jamais | aucun collage, physique pure |
| selon la vitesse | sous le seuil ça bloque, au-dessus ça rebondit |

Les intervalles laissent passer la balle partout : 76 mm aux demis, 141 mm à
l'attaque, 186 mm en défense, pour une balle de 35 mm.

**Le report de la barre sur la balle.** La figurine est un corps cinématique :
une barre qui coulisse pousse la balle, et au moment du tir la balle **hérite de
la vitesse latérale de la barre**. Coulisser puis claquer pendant le mouvement,
c'est le tir tiré — le timing du claquement dans le glissement est une compétence
à part entière.

**Le pied plat à angles arrondis.** Un disque dévie toujours le long du rayon,
donc rien n'est jamais franchement droit. Le rectangle à angles arrondis donne
une face plate qui renvoie droit et des angles qui ouvrent les diagonales. Le
rayon d'arrondi est réglable : petit = très binaire, grand = retour vers le disque.

## Prévisualisation

Un trait pointillé montre la **direction** du tir, et rien d'autre. Aucun rebond
n'est calculé ni affiché : une prévisualisation de trajectoire offrirait
gratuitement au débutant ce que l'expert met vingt matchs à intérioriser, ce qui
est disqualifiant pour un jeu dont la prémisse est la compétition. La direction,
elle, est de la géométrie au présent, pas une prédiction — et elle enseigne la
forme du pied.

L'apprentissage des bandes se fait **après** le tir : la balle laisse une traînée.

## Mesurer

Le bouton **RÉG** ouvre les réglages (restitution des bandes, friction, rayon de
capture, puissance, seuils de départage du geste) et le **Journal**, qui donne :

- le nombre de tirs et le **taux de tirs accidentels** — cible **< 5 %**
- le nombre de **revanches** consécutives — cible **médiane ≥ 3**
- la longueur moyenne des chaînes de passes — cible **2 à 3**
- le **temps jusqu'au premier but** — cible **< 3 min**, arrêt au-delà de 5 min

La restitution des bandes est le réglage dont tout dépend : sans prévisualisation,
un rebond qui semble arbitraire est un rebond dont on n'apprend rien. Règle-le
avant tout le reste, et ne change **qu'un seul paramètre entre deux binômes**.

## Critères d'arrêt

À assumer avant de commencer, pas à assouplir après les tests.

1. Chaîne médiane de revanches **inférieure à 2**.
2. Aucune progression de précision mesurable entre la session 1 et la session 2 —
   pas de courbe de compétence, donc pas de jeu compétitif possible.
3. Temps jusqu'au premier but **supérieur à 5 minutes** pour la majorité des
   testeurs — le contrôle est insoluble sous cette forme.

## Structure

    www/index.html   le prototype complet, sans aucune dépendance
    game.html        même source, sans l'enveloppe <html> (version web publiée)
    capacitor.config.json
    .github/workflows/android.yml   construit l'APK et publie la release
