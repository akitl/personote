---
name: project-state-handoff
description: "État du chantier Dices Golf au 2026-10-21 (derniers commits, livrables Steam/vidéos hors dépôt, points ouverts) — à lire en reprenant sur un autre PC"
metadata:
  node_type: memory
  type: project
  originSessionId: 84640d2f-4728-4b16-904d-02b49dbb5ac8
  modified: 2026-10-21T11:03:29.196Z
---

État au 2026-10-21 (branche feat/v02-core, rien poussé par moi) :
- Commits récents : d8e5fb6 (lot A : terrain gratuit en boutique, touches dés 1-4/Tab, panneau bâtiment, quitter), b993f44 (lots B+C : tuiles bonus sur routes utiles ramassées à 1 case, mini-golf 20 niveaux), 2eeb647 (outils vidéos Steam), ce71d5c (Steam : SteamService, 33 succès, outils d'images store).
- Suite complète 4.7.2 : 595 tests, 0 échec au commit ce71d5c.
- Binaires GodotSteam (addons/godotsteam/win64, linux64, osx ; 31 Mo) NON commités : à recopier à la main ou re-télécharger (lien + SHA256 dans addons/godotsteam/PROVENANCE.md). Décision commit/LFS/Windows seul laissée à l'utilisateur.
- Hors dépôt : C:\travaille\godot\dices_golf_steam (visuels, captures, icônes succès, textes EN/FR, STEAM_CHECKLIST.md) et C:\travaille\godot\dices_golf_videos (trailer + gameplay EN/FR ; work\ = rushes 3,6 Go jetables).
- Points ouverts : licence musique/bruitages inconnue ; 5 GIF cités dans about_*.bbcode inexistants ; capture 01_domain coupe la rangée rouge ; langue par défaut anglais si système non FR/EN (provisoire GDD §20) ; liste des 33 succès à valider ; test d'un export Windows lancé depuis Steam ; yellow_02/pink_02 hors bande ; vidéo gameplay finit sur 2 s de silence ; mix audio jamais écouté.
- Attention : si le client Steam tourne, un lancement réel du jeu initialise Steam (app 4165680) et rattrape les succès de la sauvegarde.

**Why:** l'utilisateur change de PC et veut reprendre au même point.
**How to apply:** vérifier `git log` et ces chemins avant de supposer qu'ils existent sur la machine courante. Voir aussi [[dices-golf-scope-decisions]] et [[stop-only-own-godot-processes]].
