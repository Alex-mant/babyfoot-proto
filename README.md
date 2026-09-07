# Babyfoot — prototype de validation

Prototype jouable d'un babyfoot mobile **au tour par possession**. Il n'existe pas
pour être un jeu : il existe pour répondre à **une seule question** en quatorze jours.

> Est-ce que deux personnes qui viennent de finir un match en redemandent un
> deuxième, puis un troisième — sans qu'on le leur propose ?

Tout le reste (comptes, en ligne, IA, progression, monétisation, clubs
d'entreprise, direction artistique) est délibérément hors périmètre.

## Tester

- **Web, sans rien installer** — ouvre la page publiée sur ton téléphone, en paysage.
- **APK Android** — onglet *Releases* → `babyfoot-proto.apk`. Reconstruit
  automatiquement à chaque push sur `main` par GitHub Actions.

Deux joueurs, un seul téléphone. Match en 5 buts.

## Les contrôles

| Geste | Effet |
|---|---|
| Boutons de barre (bas gauche) | sélectionner la barre — rangés dans leur ordre spatial sur le terrain |
| **Glisser ↕** | faire coulisser la barre sélectionnée (indirect : le doigt ne masque pas la table) |
| **Claquer ↔** | frapper — vers la droite ou vers la gauche, en repère écran |
| Vitesse du claquement | puissance du tir |

**On ne vise pas, on se place.** Il n'y a aucune ligne de visée et aucune
prévisualisation de rebond : l'angle du tir sort du décalage entre la figurine et
la balle. Frapper décentré envoie en biais. Une prévisualisation offrirait
gratuitement au débutant ce que l'expert met vingt matchs à intérioriser — ce qui
est disqualifiant pour un jeu dont la prémisse est la compétition.

L'apprentissage se fait donc **après** le tir, pas avant : la balle laisse une
traînée qui rend les bandes lisibles.

## Le tour

Un tour est **une possession, pas un tir**. On garde la main tant qu'on garde la
balle ; on la perd sur interception, au bout de 4 actions, ou dès qu'on tire
depuis la barre d'attaque.

À chaque changement de possession, celui qui vient de perdre la balle **place sa
défense** avant de passer le téléphone.

> Limite connue du hotseat : sur un seul écran, le placement défensif ne peut pas
> être simultané et caché comme le prévoit la spec. Il est ici posé à l'aveugle
> avant que l'attaquant ne joue. La version cachée n'a de sens qu'en jeu
> asynchrone en ligne.

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
