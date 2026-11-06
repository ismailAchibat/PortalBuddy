# PortalBuddy

Jeu de puzzle en vue FPS développé sous **Unreal Engine 5.6.1** (Blueprints) dans le cadre du module *Réalité étendue et interactions multimodales* (ENSIM).

## Mécanique innovante

Le joueur incarne un personnage en vue à la première personne, accompagné d'un petit robot, son *buddy*.

- **Le joueur** est lent, armé, mais vulnérable.
- **Le robot** est petit, agile et peut accéder à des endroits que le joueur ne peut pas atteindre.
- En un appui, la vue **bascule sur le robot**. Pendant ce temps, le corps du joueur reste immobile et exposé aux ennemis.
- Le robot tire un projectile qui ouvre un **portail** à l'impact. Le joueur peut ensuite s'y **téléporter**.

L'idée centrale est l'interdépendance entre les deux corps : le robot repère et ouvre la voie, le joueur traverse et affronte les défenseurs.

## But du jeu

Chaque salle est un puzzle : sortir de la salle et atteindre le **point d'arrivée**. Des bots défenseurs gardent certaines salles.

## Contrôles (prototype)

| Action | Touche |
|---|---|
| Déplacement | WASD |
| Saut | Espace |
| Tir | Clic gauche |
| Basculer joueur / robot | À définir |
| Quitter | Échap |

## Avancement

- [x] Projet UE créé (template First Person) et dépôt GitHub
- [ ] Pawn du robot et changement de vue
- [ ] Tir du robot et portail
- [ ] Téléportation du joueur
- [ ] Salles de puzzle et point d'arrivée
- [ ] Bots défenseurs
- [ ] Build Windows

## Récupérer le projet

```bash
git clone https://github.com/ismailAchibat/PortalBuddy.git
```

Ouvrir `PortalBuddy.uproject` avec Unreal Engine 5.6.1. Les dossiers générés (`Intermediate`, `Saved`, `DerivedDataCache`…) sont recréés automatiquement.
