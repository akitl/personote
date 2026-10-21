# Plan : retours du test du 18/10 (terrains, dés, bonus, mini-golf, options) et vidéos Steam

## Contexte

Après le dernier test, l'utilisateur juge le jeu « très réussi » et demande des retouches :

1. **Terrains débloqués :** un nouveau type de terrain atterrit dans la réserve, ce qui n'est pas clair. Il doit apparaître directement dans le menu de création de terrain (la boutique), le premier exemplaire à 0 or.
2. **Sélecteur de dés :** la molette sur le sélecteur doit changer de dé sans zoomer ni dézoomer. Il faut aussi un raccourci clavier pour changer de dé.
3. **Tuiles bonus « +1 coup » :**
   - elles ne sont presque jamais sur un chemin utile vers le trou ;
   - aller les chercher coûte un coup, donc on n'y gagne rien ;
   - il faut les ramasser quand la balle s'arrête à 1 case, pas seulement pile dessus.
4. **Bâtiment mini-golf :** un clic dessus doit ouvrir son propre petit menu avec « Jouer ». Passer par le club-house n'est pas clair.
5. **Mini-golf :** les 10 niveaux sont « tous bien ». Il en veut d'autres.
6. **Menu d'options :** ajouter un bouton « Quitter le jeu ».
7. **Vidéos :** produire les deux vidéos de la page Steam avec ffmpeg (`C:\travaille\tools\ffmpeg-9.0-full_build\bin`), à partir de situations de jeu créées et capturées par nos outils.

**Décisions de l'utilisateur (18/10)**
- **Godot 4.7.2 :** tests, simulations et captures passent sur 4.7.2, la version de son éditeur et de l'export.
- **Vidéos :** un trailer de 60 à 90 s et une vidéo de gameplay de 2 à 3 min, en 1920×1080, 60 i/s, MP4 H.264.
- **Musique :** celle du jeu. L'utilisateur confirme que la licence permet l'usage promotionnel.
- **Mini-golf :** 10 nouveaux niveaux (20 au total), avec les mêmes récompenses : 200 or à la première réussite et 0,25 % des revenus des visiteurs par étoile, soit jusqu'à 15 % pour 60 étoiles.

**Règles inchangées**
- `doc/claude.md` ; arbitrages au GDD §20 et règles numérotées dans RULES_SPEC.
- Clés i18n via `tools/i18n/ui_keys_*.csv` puis `build_trad.ps1`.
- Un commit Conventional en anglais par lot, sans push.
- Ne jamais commiter `project.godot`, `README.md`, `.research-trail/` ni les `.uid` créés par son éditeur.
- N'arrêter que mes propres processus (Godot lancé par moi), jamais son éditeur 4.7.2 ni ses tests GUT.
- Agents : Read, Edit et Write, pas sed ni cat.

**Exécution :** une suite de workflows avec implémentation, revue adversariale, correctifs, puis vérification par la suite complète et des captures en rendu réel que je regarde. Ensuite un commit par lot. Le lot B et le lot C touchent des fichiers disjoints et peuvent tourner en parallèle.

---

## Lot 0 : bascule sur Godot 4.7.2 (préalable)

- Exécutable : `C:\travaille\godot\Godot_v4.7.2-stable_win64_console.exe`. Mettre à jour `doc/claude.md` (chemin, commandes de test), `ARCHITECTURE.md` et les commentaires d'usage des outils.
- Lancer la suite complète en 4.7.2, puis corriger ce qui casse : API, avertissements devenus erreurs, `.import`.
- Lancer une visite de captures pour repérer d'éventuelles différences de rendu.
- `project.godot` reste un fichier de l'utilisateur : ne pas le commiter.

## Lot A : terrains, dés, options, menu des bâtiments (commit 1)

### A1. Terrain débloqué : directement dans la boutique, premier exemplaire à 0 or

**Cœur (`script/core/progression.gd`)**
- `course_offers()` (l.353-386) propose aussi le contenu `acquisition == "license"` dès que sa zone est débloquée, au prix de 0, déjà dans `courses.csv`. Aujourd'hui, le `break` de la l.375 l'écarte.
- `buy_skill` / `_license_plan` (l.268-323) n'appellent plus `_grant_content` pour la licence. On garde le refus `no_license_course` seulement si le contenu manque au catalogue. `granted_instance_id` disparaît, et le texte flottant « Nouveau terrain ! » de l'arbre (`skill_tree.gd:1092`) devient « Nouveau terrain dans la boutique ! ».
- `new_game` (l.97-107) ne crée plus green_01 dans la réserve : il devient une offre gratuite.
- `granted_content_ids` reste le registre des terrains « déjà fournis ».

**Sauvegarde**
- Aucun nouveau champ.
- Les anciennes sauvegardes gardent leurs terrains déjà accordés, en réserve ou posés.

**Interface (`scene/domain/domain_hud.gd`, `domain_hud_model.gd`)**
- La boutique liste l'offre gratuite avec un libellé « Gratuit », en tête de liste.
- L'onglet « Terrains ● » signale l'offre gratuite.
- Un clic achète l'offre puis enchaîne sur la pose, comme pour les achats existants (`domain.gd:_on_buy_requested`, l.830-842).

**Accueil et indices**
- Le panneau de bienvenue (l.801-826) et l'indice 2 (l.703-709) disent « Placez votre premier terrain » et leur bouton « Placer » achète l'offre gratuite puis lance la pose.
- La réserve ne garde que les terrains archivés et les poses annulées par Échap. Textes mis à jour : `UI_DOMAIN_RESERVE_*`, `UI_DOMAIN_TOAST_BOUGHT`, `UI_TREE_NEW_COURSE`.

**Joueur simulé et outils**
- `simulate_campaign.gd` (`_buy_mandatory`, `_next_mandatory_cents`, `_place_all`) et `simulate_endless.gd` achètent l'offre gratuite.
- `tools/screenshot.gd` (état « first ») et `screenshot_tour.gd` (l.207-226) sont adaptés.

**Tests et documentation**
- Tests à adapter : `test_progression` (l.62, 274, 553, 590, 617), `test_integration` (l.101, 170), `test_domain_shop` (l.41-73, 139-180), `test_buildings:327`, `test_domain_park:81-100`.
- Nouveau test : licence → offre gratuite → achat → pose ; aucun doublon après rechargement.
- Documentation : R-PROG-07, -13 et -14, R-BUY-03, GDD §03 « Acquisition » et §20, et la ligne de `doc/CLAUDE.md` « une licence fournit son premier terrain ».

### A2. Sélecteur de dés : molette sans zoom, raccourcis clavier

**Zoom parasite**
- Cause : `dice_selector.gd:_input` (l.62-71) ne consomme que l'événement *relâché* de la molette. L'événement *pressé* atteint donc `camera_2d.gd:_unhandled_input` (l.74-89).
- Correctif :
  - consommer aussi `is_action_pressed("wheel_*")` pendant le survol, même pendant l'animation de 0,3 s ;
  - brancher `hover_changed` sur `camera.input_enabled` dans `play.gd:_wire_ui` (l.230), car ce signal prévu n'est jamais connecté.

**Raccourcis** (dans le `match` de `play.gd:650`, seulement en visée, sans dialogue ouvert et hors animation)
- Touches **1 à 4**, en `physical_keycode` pour l'AZERTY, et pavé numérique 1 à 4 : dé direct (D6, D10, D20, D30), ignoré si le dé n'est pas débloqué.
- **Tab** / **Maj+Tab** : dé suivant ou précédent parmi les dés débloqués. Vérifier que Tab ne déplace pas le focus (boutons en `FOCUS_NONE`).
- Ligne d'aide du panneau de visée mise à jour.
- Les raccourcis sont provisoires et notés au GDD §20.

**Tests :** scène de partie headless (molette sur le sélecteur : dé changé, zoom inchangé ; touches 1 à 4 et Tab).

### A3. Menu propre aux bâtiments

- Un clic sur un bâtiment posé (`domain.gd:409-439`) ouvre un **petit panneau du bâtiment** au lieu du club-house.
  - On reprend le gabarit de la fiche terrain (`InfoPanel`, `domain_hud.gd:_rebuild_info`, `_add_line`, `_add_button`) et les données de `ClubhouseModel.building_rows` / `minigolf_activity`.
  - Contenu : nom, effet, état, bouton « Déplacer ».
  - Pour le mini-golf, un gros bouton **« Jouer »** (« n / 60 ★ », revenu actuel) : clic → `Game.go_minigolf()`.
- Le clic sur le club-house ouvre toujours le panneau du club-house, qui garde ses lignes de bâtiments.
- La « plomberie » du HUD suit : `close_top` (Échap), `any_panel_open`, fermetures mutuelles, évitement des messages, `free_view_rect`, blocage pendant la pose.
- Nouveau modèle pur, testable, et test de scène : clic sur le mini-golf → panneau → Jouer → scène du mini-golf.
- R-MINI-17 révisée (entrée directe par le bâtiment) et GDD §20.

### A4. Bouton « Quitter le jeu »

- `scene/common/options_menu.tscn/.gd` : bouton `QuitButton`, avec la clé `UI_OPTIONS_QUIT` dans `ui_keys_options.csv`.
- Le gestionnaire appelle `Game.save_now()` via `get_node_or_null("/root/Game")`, puis `get_tree().quit()`. `quit()` n'envoie pas WM_CLOSE_REQUEST, d'où l'enregistrement explicite.
- Bouton caché sur le web (`OS.has_feature("web")`).
- Tests : `test_settings.gd` (l.145-147, 165).

## Lot B : tuiles bonus utiles (commit 2)

**Placement (`CourseGenerator.bonus_sites`, `course_generator.gd:274-336`)**
- Les candidats sont les cases **sur ou tout près d'un bon chemin** : `d_départ(c) + d_trou(c) ≤ optimum + 1`.
  - Les distances en coups sont calculées avec le dé de référence du palier et le D6 (`shot_distances` dans les deux sens).
  - Mêmes exclusions qu'aujourd'hui : départ, trou, arbre, eau, VOID, moins de 2 cases du départ ou du trou.
  - Repli sur les candidats actuels si le palier n'en a pas assez.
- Le classement par hachage de case est gardé (R-BONUS-04).
- Les tracés ne changent pas : les tuiles ne sont pas dans les empreintes figées, donc aucun changement de `GENERATOR_VERSION`.

**Ramassage (`attempt.gd:930-939`)**
- Une tuile active est ramassée quand la balle **s'arrête à 1 case ou moins** (Chebyshev ≤ 1), pas en vol.
- Plusieurs tuiles à portée sont toutes ramassées, dans la limite de `max_bonus_tile_shots`.
- L'aperçu de visée met en évidence les tuiles qui seraient ramassées à l'arrivée sur la cible (`play.gd`, `aim_overlay.gd`).

**Terrains existants**
- Les sites sont des données dérivées. Une sauvegarde qui contient d'anciens sites les recalcule au chargement grâce à un champ optionnel `bonus_version` (absent = 1).
- Une tentative en cours garde les siens.

**Équilibrage**
- Les compétences de tuiles bonus figurent dans les équipements de référence. Il faut donc remesurer les taux des 18 terrains (`select_seeds.gd`, graines épinglées) et lancer `simulate_campaign` (human et optimal, 10 graines) ; la durée humaine visée est de 90 à 120 min.
- Si la campagne devient trop facile, documenter avant tout ajustement. Seul ajustement prévu, et seulement si nécessaire : la valeur des compétences de chance de tuile.
- Mettre à jour le critère « détour au jaune » de `select_seeds.gd` (l.65, 362-368), qui utilise le site 0.

**Tests et documentation**
- Tests : `test_attempt` (R-BONUS, ramassage à 1 case), `test_course_generator` (sites sur une route quasi optimale, indépendance du pool), `test_save_system` (recalcul), `test_putt`, `test_dice_zone`.
- Documentation : R-BONUS-01..08 révisées, GDD §20, SIMULATION_CAMPAGNE.

## Lot C : 10 nouveaux niveaux de mini-golf (commit 3, en parallèle du lot B)

**Outil de conception** (`tools/minigolf_design.gd`, dans le dépôt cette fois)
- Pour un niveau, il rapporte :
  - l'optimum et une solution ;
  - le nombre de suites optimales (`count_optimal`) ;
  - la nécessité de chaque mécanisme (avec `avoid`, ou en remplaçant le mécanisme par du sol ou un mur) ;
  - les cases inaccessibles et la carte ASCII.
- Il propose aussi une recherche de variantes aléatoires autour d'une esquisse. L'agent dessine, l'outil vérifie et trie.

**Niveaux 11 à 20** (`res/data/minigolf.json`, `ui_keys_minigolf_more.csv`)
- Dessinés à la main avec l'outil, en progression plus difficile.
- Les 5 mécanismes en combinaisons nouvelles : poussoirs et glace, plusieurs groupes de portes, rebondisseurs en chaîne, interrupteurs à compter.
- Par visés de 5 à 12, une solution optimale unique ou presque, des tailles jusqu'à 16×11.
- Ids `level_11` à `level_20` : les étoiles existantes ne bougent pas.

**Tests et règles**
- `test_minigolf.gd` : `LEVEL_COUNT = 20`.
- Règle pour 11 et plus : `combo` avec au moins 3 mécanismes, chacun nécessaire, aucune case de décor inaccessible, par égal à l'optimum.
- Compteurs « 10 », « 30 » et `level_10` codés en dur dans `test_minigolf_scene.gd`, `minigolf_scene.gd`, `screenshot_tour.gd`, `BUILDING_MINIGOLF_DESC`, RULES_SPEC et PROVENANCE.

**Écran de choix (`scene/minigolf/minigolf.tscn`)**
- Deux pages de 10 cartes (onglets « 1–10 » et « 11–20 », grille 5×2 inchangée).
- La page 2 se débloque après le niveau 10 ; l'ouverture reste séquentielle.

**Récompenses :** inchangées par niveau, comme décidé. Ajuster le test de calibrage de la prime (`test_minigolf_scene.gd:44-46`) : 20 × 200 = 4 000 or.

**Captures :** mode `minigolf` étendu (page 2, quelques nouveaux niveaux en visée), regardées.

## Lot D : vidéos Steam (après les lots A à C, pour montrer le jeu final)

**Capture**
- Nouvel outil `tools/trailer.gd`, sur le modèle de `screenshot_tour.gd`, pilote une liste de plans scénarisés. Chaque plan a une durée et des mouvements de caméra lents et lisibles. La partie est pilotée par le solveur, avec jets forcés pour créer les moments voulus (`attempt._debug_force_rolls`).
- Rendu avec le Movie Maker de Godot : `--write-movie <dossier>/plan_XX.png --fixed-fps 60 --resolution 1920x1080` (suite d'images PNG et piste audio WAV), un plan par lancement pour pouvoir le refaire seul. Si le Movie Maker de la 4.7.2 pose problème, repli sur une capture image par image par l'outil.

**Plans prévus**
- titre (logo, nom) ;
- choix du dé et zone d'arrivée (D6 pile contre D30 large) ;
- réussite critique ;
- éclaboussure dans l'eau ;
- putt final et public qui réagit ;
- tracés en fourche, en coude et en L ;
- domaine qui grandit : nouvelle partie, milieu, fin, parc embelli ;
- club-house et pose d'un bâtiment ;
- casse-tête de mini-golf résolu ;
- Enfer infini ;
- arbre de compétences.

**Montage (ffmpeg)**
- Cartons de texte dessinés dans Godot, police Jersey10, pour garder le style pixel. Accroches courtes en **anglais**, la langue par défaut de Steam, et version française des deux vidéos en bonus si c'est simple.
- Coupes et fondus, musique du jeu en fond (par exemple « Title Theme », puis « Victory! » à la fin) avec les bruitages mixés dessous.
- **Trailer, 60 à 90 s :** accroche, boucle de jeu, progression, fin sur le titre.
- **Gameplay, 2 à 3 min :** un vrai trou joué sans coupe rapide, un passage de domaine, du mini-golf.
- Export en MP4 H.264 High, yuv420p, 1920×1080, 60 i/s, environ 12 Mb/s, audio AAC 192 kb/s.
- Sortie dans `C:\travaille\godot\dices_golf_videos\`, hors du dépôt, sans rien versionner.

**Contrôle**
- J'extrais des images clés de chaque vidéo avec ffmpeg et je les regarde.
- Je vérifie durée, résolution, débit et audio avec ffprobe.
- L'utilisateur juge le rendu final.

## Vérification (à chaque lot)

- Suite complète en **Godot 4.7.2** : `"C:/travaille/godot/Godot_v4.7.2-stable_win64_console.exe" --headless --path . -s tests/run_tests.gd`, 0 échec avant chaque commit, lancée par moi.
- Captures en rendu réel des écrans touchés, regardées : boutique avec l'offre gratuite, partie avec la molette et les touches, panneau du mini-golf, menu d'options, tuiles bonus sur la route avec ramassage à 1 case, page 2 du mini-golf.
- Lot B : taux remesurés et simulations de campagne chiffrés dans la documentation.
- Lot D : images clés et sortie de ffprobe des deux MP4.
- Les tests à la souris restent à faire par l'utilisateur, et je le signale.

## Fichiers principaux

- **Cœur :** `script/core/progression.gd`, `attempt.gd`, `course_generator.gd`, `minigolf.gd`, `minigolf_solver.gd`.
- **Domaine :** `scene/domain/domain.gd`, `domain_hud.gd`, `domain_hud_model.gd`, `clubhouse_model.gd`, `domain_club_panel.gd`.
- **Partie :** `scene/play/play.gd`, `scene/common/dice_selector.gd`, `camera_2d.gd`, `options_menu.gd/.tscn`, `scene/tree/skill_tree.gd`.
- **Mini-golf :** `scene/minigolf/minigolf.gd/.tscn`, `minigolf_model.gd`, `res/data/minigolf.json`.
- **Outils :** `tools/simulate_campaign.gd`, `simulate_endless.gd`, `select_seeds.gd`, `screenshot.gd`, `screenshot_tour.gd`, nouveaux `minigolf_design.gd` et `trailer.gd`.
- **Documentation :** `doc/claude.md`, `RULES_SPEC.md` (R-PROG, R-BUY, R-BONUS, R-MINI), GDD §03 et §20, `ARCHITECTURE.md`, `SIMULATION_CAMPAGNE.md`, `res/data/PROVENANCE.md`.
