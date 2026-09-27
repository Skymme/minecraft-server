# Serveur Minecraft — ATM9 To the Sky + MineColonies

Serveur privé pour 4 joueurs, Minecraft Java **1.20.1 / Forge**, basé sur le modpack
[All the Mods 9 - To the Sky](https://www.curseforge.com/minecraft/modpacks/all-the-mods-9-to-the-sky)
avec [MineColonies](https://www.curseforge.com/minecraft/mc-mods/minecolonies) en plus.

Le serveur tourne dans Docker via l'image [`itzg/minecraft-server`](https://docker-minecraft-server.readthedocs.io/),
qui télécharge le modpack depuis CurseForge et ajoute les mods supplémentaires.
Hébergement cible : une VM **Oracle Cloud Always Free** (voir [docs/oracle-cloud.md](docs/oracle-cloud.md)).

## Contenu du repo

| Fichier | Rôle |
|---|---|
| `docker-compose.yml` | Serveur Minecraft + conteneur de sauvegardes automatiques |
| `.env.example` | Modèle des variables (whitelist, RAM, mot de passe RCON…) |
| `docs/oracle-cloud.md` | Création et configuration de la VM Oracle |

Non versionnés : `.env` (secrets), `data/` (monde, mods, logs), `backups/`.

## Lancer le serveur

```bash
cp .env.example .env      # puis éditer .env
docker compose up -d
docker compose logs -f mc # le 1er démarrage télécharge ~400 mods, compter 5-15 min
```

Le serveur est prêt quand les logs affichent `Done (...)! For help, type "help"`.

Commandes utiles :

```bash
docker compose exec mc rcon-cli            # console serveur (op, whitelist add, stop…)
docker compose restart mc
docker compose down                        # arrêt propre
```

## Côté joueurs (client)

Chaque joueur doit avoir **exactement** les mêmes versions que le serveur :

1. Installer le launcher CurseForge (ou Prism Launcher).
2. Installer **All the Mods 9 - To the Sky**, version indiquée dans `ATM9_SKY_VERSION`.
3. Ajouter au profil : **MineColonies**, **Structurize**, **BlockUI**, **Domum Ornamentum**
   (versions 1.20.1 Forge — les mêmes que dans `data/mods/` sur le serveur).
4. Allouer au moins 8 Go de RAM au jeu.
5. Se connecter à `IP_DU_SERVEUR:25565`.

## Sauvegardes

Le conteneur `backups` archive le monde toutes les 6 h dans `backups/` et supprime les
archives de plus de 7 jours. Penser à copier régulièrement une archive hors de la VM.

## À vérifier au premier lancement

- Que le monde généré est bien un **skyblock** (le pack fournit son propre type de monde).
- Les versions exactes de MineColonies & dépendances téléchargées (`data/mods/`) : les
  épingler ensuite dans `CURSEFORGE_FILES` (`minecolonies:<fileId>`) pour éviter les
  mises à jour surprises.
