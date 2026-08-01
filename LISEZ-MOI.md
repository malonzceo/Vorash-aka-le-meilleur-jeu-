# Vorash : L’Éveil des Ombres — version 3D

ARPG dark fantasy solo, caméra troisième personne, six paysages 3D, tes modèles de personnages importés. Tourne hors ligne dans le navigateur.

## Lancer le jeu

Double-clique sur `index.html`.

**Garde le dossier entier.** `index.html` et le sous-dossier `lib` doivent rester côte à côte : `lib/three.min.js` est le moteur 3D (Three.js r148, licence MIT, incluse dans `lib/LICENSE-three.txt`). Si tu déplaces `index.html` tout seul, il essaiera de télécharger le moteur en ligne — ça marche, mais tu perds le hors ligne.

Les modèles de personnages, eux, sont **embarqués dans `index.html`** : aucun fichier `.glb` à côté, rien à charger.

Chrome ou Edge à jour recommandés. Firefox marche aussi.

Au premier clic dans la fenêtre, la souris est capturée pour piloter la caméra. `Échap` la relâche et ouvre le menu.

## Tes modèles

Les quatre `.glb` riggés sont en jeu, un par classe :

| Fichier | Classe |
|---|---|
| `knight.glb` | Chevalier de Cendre |
| `mage.glb` | Mage des Éclipses |
| `archer.glb` | Rôdeur de Veillenuit |
| `necro.glb` | Oblat du Suaire |

Le Rôdeur a enfin son modèle : plus aucune classe ne tourne avec le personnage
géométrique d'origine.

### Ce qu'il a fallu leur faire

Squelette **UE5 Mannequin**, 41 os, skinning propre (poids normalisés,
2,3 à 3,0 influences par sommet). Vérifié : le skinning au repos reproduit
exactement le maillage d'origine, écart os/chair de 2 à 15 cm.

1. **Décimation** par simplification quadrique, `JOINTS_0` et `WEIGHTS_0`
   préservés : 1 438 686 → 24 000 triangles pour le chevalier, autant pour
   les trois autres.
2. **Textures** ramenées en 1024² JPEG.
3. **Quantification** `KHR_mesh_quantization` : positions 14 bits, normales 8,
   UV 12, poids 8.
4. Le necro n'était pas centré sur son root (décalé de 1,10 m en X) — recentré
   sur le bassin au chargement. Les hauteurs sont normalisées par classe.

Bilan : **146 Mo → 2,8 Mo** pour les quatre.

### Les animations

**Tes fichiers ne contiennent aucun clip d'animation** — `animations: 0` sur
les quatre. Tu as le squelette, pas le mouvement. Tout ce qui bouge est donc
calculé image par image dans la section `SQUELETTE` du fichier.

Deux choses rendaient ça délicat, et le code les contourne :

- Chaque modèle a **sa propre pose de repos** (bras le long du corps pour le
  chevalier, bras écartés pour le rôdeur, faux en main pour l'oblat). Le code
  ne remplace jamais la rotation d'un os : il part de sa rotation de repos et
  y ajoute un delta.
- Les os UE pointent le long de leur **axe X local**, différemment orienté d'un
  os à l'autre — plier un genou n'est pas la même rotation locale que plier un
  coude. Le code précalcule pour chaque os les trois axes anatomiques
  (fléchir / tourner / écarter) exprimés dans le repère de son parent, puis
  applique `R_local = (R_parent⁻¹ · W · R_parent) · R_repos`.

Ce qui est animé : repos avec respiration, marche, course (foulée allongée,
cadence ×1,5, buste penché), saut (ciseau des jambes, corps groupé, buste qui
se redresse pendant la chute), réception (flexion des genoux, hanches qui
descendent de 16 cm), roulade (culbute complète autour des hanches), et une
attaque par classe.

### Une attaque par classe

| Classe | Geste | Effet |
|---|---|---|
| Chevalier | épée armée derrière l'épaule, fend en diagonale, buste vissé | croissant de vent blanc-bleu le long de la lame |
| Mage | main gauche qui cale le bâton, droite qui pousse | boule dorée + traînée dorée |
| Rôdeur | bras gauche tendu sur l'arc, droite qui tire à la joue puis lâche | flèche + traînée pâle |
| Oblat | faux qui balaie à l'horizontale, corps qui suit | faucheée violette large et basse |

Le mage et le rôdeur attaquent donc à distance en coup de base, le chevalier
et l'oblat au corps à corps.

### Pour aller plus loin

Une animation écrite à la main reste une animation écrite à la main. Si tu
veux du mouvement capturé, passe un des quatre `.glb` dans Mixamo, récupère
`idle` / `walk` / `run` / `attack`, et je branche un `AnimationMixer` par-dessus :
le squelette est déjà là, il n'y a que le retargeting des noms d'os à faire.

## Commandes

**Clavier & souris**

| Action | Touche |
|---|---|
| Se déplacer (relatif à la caméra) | ZQSD / WASD / flèches |
| Orienter la caméra | souris |
| Zoom caméra | molette |
| Attaquer | clic gauche |
| Sauter | Espace |
| Courir | Maj **maintenu** |
| S'accroupir | Ctrl ou C (bascule) |
| Parler / récolter | E |
| Manger ou boire | F |
| Sorts 1 à 4 | 1 2 3 4 |
| Sac & équipement | I |
| Journal des quêtes | J |
| Talents & sorts | C |
| Ombres on/off | O |
| Menu / statistiques | Échap |

**Manette PS5 / Xbox** — branche-la, appuie sur un bouton, c'est reconnu.

| Action | Bouton |
|---|---|
| Se déplacer | stick gauche |
| Caméra | stick droit |
| Sauter | ✕ / A |
| Attaquer | R2 / RT |
| Courir | L3 **maintenu** (clic stick gauche) |
| S'accroupir | ◯ / B (bascule) |
| Parler / récolter | ▢ / X |
| Manger ou boire | △ / Y |
| Sorts 1 · 2 · 3 · 4 | R1 · L1 · L2 · R3 |
| Sac | Create / Share |
| Menu | Options / Start |
| Dialogues | croix directionnelle, ✕ valide |

## Décor et ambiance

**Ciel de nuit.** Construit depuis ton panorama : silhouettes d'arbres coupées
(le jeu a les siennes), couture entre les bords recousue par fondu croisé
(22/255 → 1,7/255, il boucle maintenant sans raccord), projeté en
équirectangulaire 2048×1024, dégradé prolongé jusqu'au zénith et 2 600 étoiles
ajoutées avec une densité qui décroît vers l'horizon. 167 Ko. La teinte de
chaque zone vient le multiplier, donc les six zones restent distinctes avec une
seule image.

**Lumière.** Bleu nuit venant du ciel, rebond vert venant du sol — c'est ce
contraste qui fait toute l'ambiance de tes références, bien plus que la finesse
des modèles. Brume assombrie de 66 % et densifiée de 35 % pour la profondeur.
Chaque maison porte maintenant une lumière chaude de fenêtre.

**Textures.** Gazon et roche recousus pour boucler (couture ramenée de 16/255 à
9,7 et 4,0). Le gazon a en plus été aplati en luminosité, sinon ses variations à
grande échelle font un effet damier une fois répété. Le terrain les répète tous
les 6 m, teintés par les couleurs d'altitude et de pente déjà en place.

**Sapin et maison.** 2,0 M → 8 990 triangles, 1,5 M → 35 722 triangles.
Le sapin remplace le sapin procédural dans toutes les instances, la maison
remplace la maison procédurale partout où elle était posée — les douze du bourg,
la taverne et la forge.

## Correctifs du 31/07

**Armes attachées aux pieds.** L'auto-skinning avait donné une partie des armes
aux os des jambes, parce qu'au repos la lame pend le long du mollet. L'épée du
chevalier était pilotée à 38 % par `hand_r` mais à 30 % par `ball_r` et 13 % par
`calf_r` — d'où la lame étirée vers l'orteil à chaque foulée. Deux règles
appliquées à la source : aucun sommet ne peut appartenir à la fois à une main et
à un pied, et un sommet piloté par une jambe à plus de 45 cm de l'axe du corps
est une arme, pas de la chair. Résultat : chevalier 94,7 % `hand_r`, oblat 100 %,
plus aucun os de jambe sur aucune arme.

**T-pose du mage et du rôdeur.** Leurs bras sont à 89,5° et 91,5° de la verticale
dans le fichier — une T-pose pure — contre 15° pour le chevalier et 4° pour
l'oblat. Comme le code anime *relativement à la pose de repos*, ils restaient
en croix. Correction : une pose de base permanente par classe (`POSE_BASE`),
qui les ramène à ~11°.

**Accroupissement à la place du dash.** Ctrl ou C bascule, ◯ à la manette.
Hanches 42 cm plus bas, genoux pliés, buste penché, vitesse à 42 %, sprint
impossible. Sauter relève automatiquement.

## Saut & course

**Saut** — un seul, pas de double saut. La vitesse horizontale au moment du décollage
est conservée pendant tout le vol : un saut sur place ne va nulle part, un saut pris
en pleine course porte loin. On ne récupère que 30 % de direction en l'air. Apex :
1,00 m sur place, 1,25 m lancé. Gravité 20,5 m/s². À la réception les jambes amortissent
(bref ralentissement, plus long si la chute était haute).

**Course** — jauge de souffle verte sous le mana. Environ 6,5 s à fond, puis
récupération complète en ~6,7 s (avec 0,85 s de temps mort avant que ça remonte).
Barre vidée = **3 secondes de jambes coupées** : vitesse à 55 % de la marche normale,
sprint interdit, récupération deux fois plus lente. On ne court pas dans l'eau,
ni le ventre vide. Chaque saut coûte aussi 13 % de souffle.

Tous les réglages sont regroupés en haut de la section joueur du fichier
(`GRAVITE`, `SAUT_V0`, `SPRINT_MULT`, `END_DRAIN`, `EPUISE_DUREE`…).

## Si ça rame

1. `O` coupe les ombres portées — c'est le plus gros gain.
2. Réduis la fenêtre du navigateur (le rendu suit la taille).
3. Ferme les autres onglets : le jeu tourne en WebGL, il partage le GPU.

Sur un GPU de milieu de gamme récent, les Bois de Veillenuit (la zone la plus chargée, ~97 000 triangles) tournent sans effort.

## Les six zones

Chaque zone est un terrain généré au bruit fractal : relief, palette par altitude et par pente, ciel dégradé, lune, étoiles, brouillard coloré, montagnes à l'horizon, eau animée. Les portails brillants bloquent le passage si tu es trop faible, ou s'il te manque une clef.

1. **Bourg-de-Cendre** (niv. 1) — vallée aplanie, dix maisons à colombages, chapelle, taverne, forge, palissade, feu de place. Sept PNJ. Zone sûre : la vie et le mana remontent vite.
2. **Bois de Veillenuit** (niv. 2) — 400 arbres, rivière encaissée, champignons luminescents, cabane de l'ermite. Loups, gobelins, sangliers.
3. **Clairière des Ronces** (niv. 5) — anneau d'arbres géants, camp de tentes, totems. Boss : **Groll, Grand Gobelin**, 2 phases.
4. **Tourbières de Val-Morne** (niv. 7) — marécage, ruines de garnison, arbres morts, tombes, brasiers turquoise. Squelettes, spectres, orcs.
5. **Crypte des Rois Muets** (niv. 11) — intérieur : huit salles reliées par des couloirs, piliers, sarcophages, torches. Boss : **le Roi Muet**, 3 phases.
6. **Château de Noirlune** (niv. 15) — enceinte, six tours, salle du trône. Porte fermée à clef. Boss final : **le Chevalier d'Éclipse**, 3 phases.

## Systèmes

- **4 classes** (Chevalier de Cendre, Mage des Éclipses, Rôdeur de Veillenuit, Oblat du Suaire), 16 sorts, 5 rangs par sort.
- **3 difficultés** : Errant, Proscrit, Damné (dégâts majorés, faim vorace, or divisé, 20 % de la bourse perdue à la mort).
- **Vie / Mana / Faim.** La faim descend seule ; sous 35 % la vie ne régénère plus ; à zéro tu perds de la vie et tu ralentis.
- **Cuisine** : 10 recettes chez Bram. Les fioles agissent tout de suite, les plats donnent des bonus de plusieurs minutes. Les récoltes repoussent partiellement à chaque retour dans une zone.
- **Économie** en écus d'or, 3 marchands, achat et revente.
- **20 quêtes** en 4 paliers, plusieurs chaînes qui mènent à l'épilogue.
- **60 objets**, 4 raretés, 3 emplacements. L'équipement change ton apparence : plaque, robe ou cuir, et l'arme que tu tiens vraiment en main.
- **2 points de talent par niveau** : +1 attribut, ou +1 rang de sort pour 2 points.

Note : sur les trois classes à modèle importé, l'apparence ne change plus avec l'armure équipée — c'est ton modèle qui s'affiche. Les statistiques de l'équipement comptent normalement. Le Rôdeur, lui, change encore de silhouette selon son armure.

## Sauvegarde

Automatique toutes les 25 secondes, à chaque changement de zone et à chaque quête rendue. Le bouton **Reprendre** du titre recharge la partie.

Elle vit dans le stockage local du navigateur : liée au navigateur, pas au fichier. Autre navigateur = nouvelle partie. Vider les données du site efface la sauvegarde.

## Notes techniques

- Aucun asset externe hors tes trois modèles : terrains, PNJ, ennemis, armes, bâtiments, ciel, lune et texture lunaire sont générés en code.
- PNJ et ennemis articulés en primitives, animés par rotation des membres : cycle de marche, balancement des bras, coup d'épée, cape qui suit.
- Les aperçus de l'écran de création passent par un second contexte WebGL miniature recopié dans le canvas de chaque carte.
- Les décors répétés (arbres, murs, tombes, piliers) passent par des `InstancedMesh` : une passe de rendu pour des centaines d'objets.
- Éclairage : hémisphérique + directionnelle lunaire avec ombres portées, plus les 7 sources ponctuelles les plus proches du joueur, réaffectées à chaque frame parmi les torches, feux, brasiers et portails de la zone.
- Collisions accélérées par grille spatiale ; les flèches traversent le feuillage mais s'écrasent sur la pierre.
