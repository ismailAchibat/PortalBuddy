# PortalBuddy : contexte du projet (pour Claude)

Ce fichier résume la conversation précédente pour qu'une nouvelle session reprenne sans rien perdre. L'utilisateur parle français. Réponds en français.

## Cadre
- **Cours** : *Réalité étendue et interactions multimodales – Unreal Engine* (ENSIM, Corentin Coupry, édition 2026). Projet solo, environ **6 semaines**.
- **Contrainte unique du projet** : une **mécanique innovante**.
- **PC de l'université** (lecteur réseau H:). Pas de droits d'administrateur supposés, pas de Visual Studio vérifié.
- **UE 5.6.1**, projet **Blueprint uniquement** (pas de code C++), template First Person, UI de l'éditeur **en français**.
- **Dépôt** : https://github.com/ismailAchibat/PortalBuddy (branche `main`, commits : initial + README). Git sur lecteur réseau : utiliser `git -c safe.directory=*` dans les commandes.
- Dossier du projet : `H:\Home\Bureau\UnrealProjects\Projet-unreal-engine\PortalBuddy`.

## Idée du jeu
Puzzle FPS. Le joueur peut basculer la vue sur son petit robot (**BP_Buddy**), plus agile, qui tire un projectile ouvrant un **portail** où le joueur se **téléporte**.
- But : sortir de chaque salle et atteindre le **point d'arrivée**. Quelques **bots défenseurs** simples (stationnaires, type tourelle).
- Règle clé : pendant qu'on pilote le robot, le **corps du joueur reste immobile et vulnérable**. Le robot est ignoré ou moins ciblé par les bots.
- Limites possibles du robot (batterie ou portée, ne tue pas). Le robot peut aussi passer par des conduits et activer des boutons.
- Portail : pas de rendu « de l'autre côté ». Un simple effet visuel (matériau ou Niagara) puis `SetActorLocation` sur le joueur.

## Critères de notation de la séance 1 (tableau fourni)
| Critère | Attendu pour un A |
|---|---|
| Mécanique innovante | Originale, compréhensible, différente d'un gameplay classique |
| Prototype initial | Preuve de concept jouable du principe central |
| Versioning initial | Dépôt créé, projet récupérable, commits clairs, première version identifiable |

Reste à faire pour la séance 1 : prototype jouable minimal (switch joueur/robot, tir, portail, téléportation) et un tag `v0.1-poc`.

## Plan sur ~6 semaines
1. Projet, whitebox d'une salle, personnage et arme du template.
2. Pawn robot, switch de vue (`Possess`), déplacement agile.
3. Tir du robot, portail, téléportation.
4. Salles 2 et 3 avec puzzles, point d'arrivée.
5. Bots défenseurs, vie du joueur, game over.
6. Finitions (sons, matériaux, UI), build Windows (voir TP : Packaging Settings, Development, Full Rebuild).

Prévoir une marge de 30 %. Rester en whitebox jusqu'à la semaine 5. Trois salles bien pensées valent mieux que huit bâclées.

## Étape 1 : Pawn `BP_Buddy` (déjà commencé, dossier `Content/Buddy` existant)
- Blueprint de classe parente **Character**. Capsule : Half Height 40, Radius 25.
- Composants : Static Mesh `BuddyBody` (sphère, échelle 0,5, NoCollision, Owner No See) et Camera `BuddyCamera` (Use Pawn Control Rotation, devant le corps).
- Class Defaults : Max Walk Speed 900, Jump Z Velocity 700 à 900, Air Control 0,6, Max Acceleration 3000, Use Controller Rotation Yaw activé.
- Event Graph (Enhanced Input) : `IA_Move` vers Add Movement Input (forward/right), `IA_Look` vers Add Controller Yaw/Pitch Input, `IA_Jump` vers Jump / Stop Jumping.
- Test : Auto Possess Player = Player 0 sur l'instance placée, puis remettre sur Disabled.
- Étape suivante : **switch de vue** (Possess entre le personnage et le robot, le corps du joueur reste vulnérable).

Traductions d'interface : Content Drawer = Tiroir à contenu, Details = Détails, Outliner = Organiseur, Place Actors = Placer des acteurs, Event Graph = Graphe d'événements, Compile = Compiler, Play = Jouer.

## Pistes pour le switch de vue (à confirmer en pratique)
- Deux Pawns, un `Possess` du PlayerController. Le mapping context Enhanced Input reste actif sur le joueur local.
- Garder une référence au personnage joueur dans le robot (et inversement).
- Le corps du joueur reste dans la scène, immobile, donc ciblable par les bots.

## MCP / EngineWire (pilotage de l'éditeur par Claude)
- Plugin **EngineWire** (Fab, par Chirag Panwar, gratuit, licence MIT, dépôt `ChiR24/Unreal_mcp`) installé dans le moteur. Serveur MCP natif sur `http://127.0.0.1:3000/mcp`, un seul outil `unreal` (opérations : search, describe, execute, configure).
- Le plugin officiel d'Epic (`ModelContextProtocol`, `AllToolsets`) n'existe **pas** en UE 5.6.1. Ne pas l'utiliser.
- Il a fallu activer dans le `.uproject` : `MovieRenderPipeline` et `MoviePipelineMaskRenderPass` (sinon le plugin ne se charge pas, erreur DLL manquantes et l'éditeur se ferme).
- Réglage : Paramètres du projet > Plug-ins > EngineWire > Native MCP > Enable Native MCP Server. La barre d'état affiche `MCP :3000 (0)`, puis `(1)` quand un client est connecté.
- Le jeton est dans `Saved/MCP/capability-token`. Il est lu dans `.mcp.json` (à la racine du projet, **ignoré par git**). **Ne jamais afficher ni committer le jeton.**
- **Proxy de l'université (Squid)** : renvoie 403 sur `127.0.0.1` si on passe par le proxy. Contourner avec `NO_PROXY=127.0.0.1,localhost` (ou `curl --noproxy '*'`). Le test direct avec le jeton répond 200.
- Toujours commencer par des appels en lecture seule, faire un commit avant les grosses modifications, vérifier chaque résultat.

## Règles de travail convenues
- Le TP vise l'apprentissage des Blueprints : laisser l'utilisateur faire (ou comprendre) le switch de vue et le portail. Automatiser les parties répétitives (placement, matériaux, réglages).
- Aucun commit ni push sans demande. Ne pas pousser de secrets.
- Fin de chaque commit : `Co-Authored-By` de la session en cours.

## État du dépôt à la fin de la conversation
Changements locaux non commités : `PortalBuddy.uproject` (plugins), `Config/DefaultGame.ini`, `.gitignore` (+ `.mcp.json`), `Content/Buddy/` (BP_Buddy en cours), ce fichier.
