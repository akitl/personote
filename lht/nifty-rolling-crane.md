# Plan — vague 7 : retours de jeu du 2026-10-18 (progression, tours, titan mondial, quête puzzle, Architecte, rendu)

## Contexte
Retours de game design après 40 min de jeu sur la V1 déployée (`docs/retours_gd_2026-10-18.md`).

**Le constat central : la progression est beaucoup trop rapide.** En 40 min, le joueur a fini la quête, monté tout le village et tué des titans mondiaux en un passage. La cause est confirmée dans le code : le joueur simulé de `serveur/internal/balance` est volontairement lent.
- Il pose ses tours en murs naïfs.
- Il ignore le voisinage quand il choisit ses tours (statistiques calculées sans ancre, `sim/builder.go:24`).
- Il ne cumule jamais dégâts fixes et cadence. Or un archer descend à 5 tirs par seconde avec +25 par flèche (`tower_stats.cpp:143-150`).

**Autres constats confirmés dans le code :**
- **Feu :** la brûlure est neutre (`archetypes.json:25`) et ignore les dégâts fixes. La « Force de la brûlure » est mangée par l'arrondi : 3 dégâts × 1,33 donne encore 3.
- **Bonus des tours :** toute tour tirée a exactement 1 à 4 propriétés selon sa rareté, plus toujours un bonus de voisinage (`generate.go:64-103`). Une tour est donc toujours pleine, et un transfert à l'Atelier ne peut que remplacer.
- **Titan mondial :**
  - 18 000 PV par joueur ;
  - pas de hasard sur les visites : 5 min d'annonce, le combat, puis 10 min de repos (`cycle.go`) ;
  - le client ne sait rien de la visite suivante ;
  - le lot du dernier coup est une tour épique et 150 or.
- **Phases du titan :** elles ne reviennent jamais en arrière et aucun soin n'existe dans le moteur (`combat.cpp:170-182`).
- **Chemin du titan :** c'est un ruban avec des chevrons sur les lignes droites (`art_grid_meshes.gd:89`).
- **Décor :** il est fixe autour de la grille, et le plateau est plat seulement jusqu'à 32 × 22.
- **Tirets cadratins :** 779 dans environ 70 fichiers suivis.

## Arbitrages de l'utilisateur (2026-10-19)
| Sujet | Décision |
|---|---|
| Rythme | Quête finie en **8 à 12 h par un bon joueur**, moins de 20 h pour un joueur lent. Un joueur simulé « bon joueur » est ajouté à l'équilibrage. |
| Titan mondial | **Pas de plafond par passage.** Énormément de PV (200 k, 1 M ou plus). Chacun fait ses dégâts, et le dernier coup est une loterie. |
| PV mondiaux | À son apparition, **chaque joueur actif ajoute des PV selon sa progression** : par exemple de 150 k en début de quête jusqu'à 1 M en fin de quête et au-delà. Le titan reste jusqu'à sa mort, même sur un serveur presque vide. |
| Visites | **Délai aléatoire de 20 à 60 min par joueur.** Risque affiché Faible (plus de 30 min), Moyen (10 à 30 min) ou Élevé (moins de 10 min). L'annonce de 5 min est gardée. |
| Lot mondial | Au dernier coup : **une tour unique du titan mondial, une épique à 4 bonus et beaucoup d'or**. Les autres joueurs gardent l'or au prorata de leurs dégâts. |
| Nombre de bonus | Plafond par rareté : Commune 0–1, Peu commune 0–2, Rare 0–3, Épique 0–4, +1 par rareté future. Le voisinage compte dans le plafond, et une tour a **au plus un voisinage**. Jamais deux bonus du même type. Le nombre est **tiré selon une table par source** (dans les données). Ordre du tirage : rareté, puis nombre de bonus, puis chaque bonus et sa qualité. Le plafond s'applique aussi aux transferts. |
| Marchand | Niveau 1 : communes de base (50 % sans bonus, sinon 1 bonus), moins de 10 % de peu communes à 1–2 bonus. Niveau 2, avec délai de réapprovisionnement : 80 % de peu communes à 1–2 bonus, 20 % de rares à 2–3 bonus. |
| Feu | Brûlure d'**élément feu**. **Dégâts fixes ajoutés à chaque tick** de brûlure. **Valeurs de base revues** et « Force » sans perte d'arrondi. Brûlures **cumulables** jusqu'à un maximum. |
| Tours uniques | Sans famille, avec leur filtre propre. **Bonus propre en plus du plafond**, ni retirable ni transférable. Premier lot : **tour divine** (Rare), **tour d'aura** (n'attaque pas, bonus sur ses 12 cases voisines), **tour du titan mondial** (Épique, lot du dernier coup). |
| Quête puzzle | Ajouts au moteur : **immunité** à un élément, **absorption** (les dégâts de l'élément soignent), **accélération en ligne droite**. Le sprint final passe par les vitesses de phase existantes. **La phase suit les PV**, y compris quand le titan remonte grâce à un soin. |
| Architecte | Le « Parchemin » devient **l'Architecte** : un bâtiment du village qui vend la taille de grille débloquée par la quête. Prix en **or, croissant avec la taille**. |
| Chemin | **Sentier peint sur le sol**, en autotile par case, diagonales comprises. |
| Arène | **Bordure générée** à la taille de la grille (falaises, rochers, palissades, forêt), ouverte seulement à l'entrée et à la sortie. Plateau agrandi au-delà de 32 × 22. |

**Sans question, d'après le retour :**
- **Fiche de tour en trois blocs :**
  - Caractéristiques, avec l'élément à côté des dégâts ;
  - Propriétés, en capsules avec une icône de bonus et sa qualité ;
  - Voisinage, avec la même capsule.
- **Nombres de dégâts :** une seule zone centrée, rotation et direction un peu aléatoires, effet fontaine. Cela remplace les 5 couloirs de D-430.
- **Barre de vie :** icônes par portion de phase (botte pour la vitesse, élément barré, soin, ligne droite).
- **Titan mondial :** rendu absurdement grand.
- **Atelier :** reprend les filtres de l'écran de défense.
- **Annonce de mort du titan :** le garde-fou existe déjà (D-347). Il sera vérifié à plusieurs dans l'E2E.

**Hors vague :** passe d'effets sonores (« plus tard ») et décor du monde (l'utilisateur continue de le peaufiner).

## Approche technique

**Tirets cadratins (en premier, seule, pour éviter les conflits)**
- Remplacement contextuel du « — » dans tous les fichiers suivis : « : », « , », « · », parenthèses, ou « - » dans le code. Cela couvre le code, les tests, les CSV des 9 langues, le serveur, les docs et `CLAUDE.md`.
- Nouvelle règle dans `CLAUDE.md` : jamais de tiret cadratin, y compris dans les messages de commit.
- Deux tests empêchent son retour :
  - un test Go qui parcourt `git ls-files` depuis la racine (`serveur/internal/repocheck`) ;
  - un test Godot pour `godot/`.
- L'historique n'est pas réécrit.

**Moteur, règles v2** (`lhtcpp/`, `tdsim.rules.v2`, contrat dans CONTRACTS §2)
- **Brûlure (DoT) :**
  - son élément vient des données ;
  - `DamageAddFlat` s'ajoute à chaque tick ;
  - la force est calculée en fixe ×100 avec un report du reste par statut (déterministe) ;
  - les cumuls par tour sont bornés par `max_stacks` (dans les données).
- **Résistances par phase :** la borne haute passe à 10000 (immunité). Nouveau champ par phase `absorb_bp` par élément : la part absorbée soigne, sans jamais dépasser les PV max. Nouvel événement de trace `Heal`.
- **Phase :** recalculée à chaque changement de PV, dans les deux sens. Un événement `Phase` est émis à chaque changement.
- **Accélération en ligne droite :** champ par phase `straight_accel {after_steps, add_bp_per_step, max_bp}`. Les séries droites sont dérivées des ancres du chemin (même direction dx, dy). La vitesse est remise à zéro au virage. Un `Move` est émis seulement à chaque changement de vitesse (D-013).
- **Tours uniques :**
  - archétype sans famille ;
  - `attacks: false` pour l'aura ;
  - modificateur propre `innate` (de soi ou de voisinage, sur tout l'anneau), non comptés dans `prim_count`.
- **Fixtures :**
  - régénérer les golden et les tests unitaires ;
  - porter `godot/tests/fixtures/combat/make_fixtures.py` vers une commande Go (`serveur/cmd/makefixtures`), car il n'y a pas de Python sur la machine ;
  - les combats stockés en v1 ne sont plus revérifiables : `Verify` les marque comme anciens, ce qui est sans effet sur le jeu (la base de test peut être remise à zéro).

**Serveur, économie et monde** (`serveur/`)
- **Bonus des tours :**
  - `rarities.json` reçoit `bonus_max` (1/2/3/4) ;
  - tables `bonus_count_weights` par source : coffres, quête, chasse, monde, fusion ;
  - le voisinage devient un bonus facultatif du même tirage (`generate.go`, `ValidateLoadout`) ;
  - transfert : ajout possible dans une place libre sous le plafond (`workshop.go:323`).
- **Marchand :** tables du niveau 1 et du niveau 2 comme décidé (`economy.json`, `buildings.json`). Les niveaux suivants seront calés par la simulation.
- **Tours uniques :** données dans `archetypes.json`, plus un champ `unique` et le filtre côté client.
- **Titan mondial :**
  - `hp_by_tier` par joueur actif, sommé à l'apparition (`world/rules.go`, `titans.json`) ;
  - `next_visit_at` tiré entre 20 et 60 min par joueur (`cycle.go`, migration), qui remplace `revisit_seconds` ;
  - `visit_risk` (low / medium / high) dans `GET /v1/world` ;
  - lot du dernier coup : unique mondiale, épique pleine et or (`world.json`).
- **Architecte :**
  - bâtiment `architect` ;
  - `player_state.grid_size_level` (migration) ;
  - route d'achat de la taille suivante, dans la limite du palier de quête (`grid.json` donne le palier qui débloque et le prix) ;
  - une disposition reste valide quand la grille grandit.
- **Titans de quête redessinés en puzzle** (`titans.json`, `msq.json`) : sprint final, immunité ou absorption, ligne droite. Les mécaniques sont introduites une à une au fil des paliers.
- **Équilibrage :**
  - nouveau joueur de référence « bon joueur » : placement qui exploite le voisinage, chemin en chicanes, cumul cadence + dégâts fixes, choix des tours par puissance effective avec ancre ;
  - critères réécrits : C1 donne 8 à 12 h au bon joueur et moins de 20 h au joueur lent ;
  - C4 devient celui d'un titan loterie : durée de vie de plusieurs heures selon la taille du serveur, part du temps par joueur, visites de 20 à 60 min ;
  - réglage des coûts, gains et PV jusqu'à ce que C1 à C5 passent.

**Client** (`godot/`)
- **Fiche de tour :**
  - `TowerSheet` et `TowerCard` en trois blocs ;
  - capsule `bonus_chip` (icône de bonus, couleur de qualité, valeur), partagée entre propriétés et voisinage ;
  - icônes d'élément (`element_*`, déjà là) à côté des dégâts ;
  - nouvelles icônes par catégorie de modificateur.
- **Nombres de dégâts :** zone unique, angle et dérive aléatoires et déterministes (graine = séquence), effet fontaine. Les soins s'affichent en vert.
- **Atelier :** `InventoryFilterBar` dans l'établi de forge. `InventoryFilter.matches` accepte l'ensemble des tours posées. Ajout du filtre « Uniques » partout.
- **Barre de vie :** `HpBarModel` lit `titan_phases` et pose les icônes par portion (vitesse, immunité, absorption, ligne droite). La phase peut revenir en arrière.
- **Architecte :**
  - les textes visibles « Parchemin » deviennent « Architecte » (msgids des 9 langues, tests de texte) ;
  - un bâtiment Architecte au village, avec un panneau d'achat de taille et un accès au plan ;
  - les identifiants internes (`planning`) restent.
- **Monde :**
  - indicateur de risque dans le bandeau, le panneau et l'infobulle du bouton ;
  - titan mondial très grand (exception à `size_for` pour le genre `world`, cadrage caméra) ;
  - lot du dernier coup dans la fenêtre « Vous avez obtenu ».
- **Chemin :** les cases du chemin forment un masque par case (autotile, diagonales), lu par le shader du sol de la grille (sentier de terre). Le ruban et les chevrons sont retirés.
- **Arène :**
  - bordure construite en code à la taille de la grille, avec les pièces du kit en `.glb` générées par `legacy/procedural_art` et placées en MultiMesh, ouverte à l'entrée et à la sortie ;
  - plateau de la vallée agrandi ;
  - `check_valley` mis à jour.

## Exécution (orchestrateur = moi ; agents en worktrees ; Workflow par vague ; revue adversariale avant fusion)
| Vague | Branches (parallèles dans la vague) |
|---|---|
| **7.0** | `v1-w7-no-emdash` : purge et tests. Seule, fusionnée avant tout le reste. |
| **7.1** | `v1-w7-tower-card` (fiche, capsules, icônes) · `v1-w7-damage-fountain` · `v1-w7-workshop-filters` · `v1-w7-engine-v2` (moteur, builder serveur, fixtures, port Go du générateur) |
| **7.2** | `v1-w7-loot` (bonus, Marchand, Atelier, uniques côté serveur) · `v1-w7-world` (PV par progression, visites, risque, lot) · `v1-w7-architect-server` (bâtiment, achat de grille) |
| **7.3** | `v1-w7-balance` (bon joueur, critères, titans de quête puzzle, réglage de toutes les valeurs) · `v1-w7-client-world-architect` (risque, titan géant, Architecte, barre à icônes, soins, uniques) · `v1-w7-path-arena` (sentier, bordure, plateau) |
| **7.4** | `v1-w7-integration` : captures fr et ru/de/ja, bascule de langue, E2E (journey, double-spend, combat mondial à plusieurs, annonce de mort), revue finale |

Après chaque fusion, je mets à jour `docs/v1/{STATUS,DECISIONS,PLAN,CONTRACTS}.md` (D-436 et suivants). Le déploiement sur tipforgood se fait seulement à ta demande ; la base de test peut être remise à zéro avec les règles v2.

## Vérification
```bash
cmake --build lhtcpp/build-release && ctest --test-dir lhtcpp/build-release --output-on-failure   # golden v2
(cd serveur && gofmt -l . && go vet ./... && go test -count=1 ./...)                             # 55432, LHT_REQUIRE_ENGINE=1
(cd serveur && LHT_BALANCE_REPORT=1 go test -count=1 -run 'TestProgressionReport|TestWorldBalanceReport' -v ./internal/balance/)
$G --headless --path godot --import   # deux fois
$G --headless --path godot --script res://tests/run_tests.gd ; check_scripts.gd (-d) ; art_checks.gd
```
- **Critères d'équilibrage :** bon joueur entre 8 et 12 h, joueur lent sous 20 h, C1 à C5 au vert.
- **Titan mondial :** un passage de fin de jeu retire une petite part des PV, et sa vie dure plusieurs heures à 1, 5 et 20 joueurs (rapport).
- **Tests de garde :** aucun tiret cadratin (test Go et test Godot).
- **Captures fenêtrées relues :**
  - fiche de tour ;
  - fontaine de dégâts ;
  - barre à icônes ;
  - sentier dans les virages et les diagonales ;
  - arène à 16 × 12 et à la taille maximale ;
  - titan mondial ;
  - Architecte ;
  - risque.
- **E2E contre le vrai serveur :** journey (achat de grille chez l'Architecte), double-spend, combat mondial à deux joueurs avec annonce de mort retenue.

## Risques
- **Règles v2 :** tous les golden et toutes les fixtures sont à régénérer, et les combats v1 stockés ne sont plus revérifiables (base de test remise à zéro).
- **Réglage multi-paramètres :** l'équilibrage est long à converger. Si C1 n'atteint pas 8 à 12 h sans casser C2 à C4, je reviens vers toi avec les options.
- **Accélération en ligne droite et soins :** davantage d'événements de trace. Leur taille est surveillée par le test de performance des traces.
- **Arène et plateau agrandi :** il faut régénérer `static.glb` et les instances du décor avec `legacy/procedural_art`.
