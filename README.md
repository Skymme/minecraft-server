# Serveur Minecraft — ATM10 To the Sky + MineColonies

Serveur privé pour 4 joueurs, Minecraft Java **1.21.1 / NeoForge**, basé sur le modpack
[All the Mods 10: To the Sky](https://www.curseforge.com/minecraft/modpacks/all-the-mods-10-sky)
avec [MineColonies](https://www.curseforge.com/minecraft/mc-mods/minecolonies) en plus.

Le serveur tourne dans Docker via l'image [`itzg/minecraft-server`](https://docker-minecraft-server.readthedocs.io/),
qui télécharge le modpack depuis CurseForge et ajoute les mods supplémentaires.
Hébergement actuel : PC perso + tunnel playit.gg. Option future : une VM cloud
(voir [docs/oracle-cloud.md](docs/oracle-cloud.md)).

## Contenu du repo

| Fichier | Rôle |
|---|---|
| `docker-compose.yml` | Serveur Minecraft + conteneur de sauvegardes automatiques |
| `.env.example` | Modèle des variables (whitelist, RAM, mot de passe RCON…) |
| `docs/oracle-cloud.md` | Création et configuration de la VM Oracle |

Non versionnés : `.env` (secrets), `data/` (monde, mods, logs), `data-*/` (anciens mondes archivés), `backups/`.

## Lancer le serveur

```bash
cp .env.example .env      # puis éditer .env
docker compose up -d
docker compose logs -f mc # le 1er démarrage télécharge ~320 mods, compter 5-15 min
```

Le serveur est prêt quand les logs affichent `Done (...)! For help, type "help"`.

Commandes utiles :

```bash
docker compose exec mc rcon-cli            # console serveur (op, whitelist add, stop…)
docker compose restart mc
docker compose down                        # arrêt propre
```

## Héberger depuis un PC perso (tunnel playit.gg)

Pour que des joueurs extérieurs rejoignent un serveur qui tourne sur un PC perso, sans
ouvrir de port sur la box :

1. Créer un compte sur [playit.gg](https://playit.gg), puis *Agents → Add Agent → Docker*.
2. Copier la clé secrète dans `.env` : `PLAYIT_SECRET_KEY=...` (jamais dans `.env.example`).
3. Lancer avec le profil playit : `docker compose --profile playit up -d`.
4. Sur playit.gg, *Add Tunnel* : type `Minecraft Java`, local address `172.30.0.10`,
   local port `25565`, proxy protocol `None`.
5. L'adresse publique (`xxx.tun.ply.gg:PORT`) est celle à donner aux joueurs. Le port
   n'est pas toujours affiché sur le site : il faut bien l'ajouter après le domaine.

Le serveur n'est joignable que quand le PC est allumé et que Docker tourne.

## Côté joueurs (client)

Chaque joueur doit avoir **exactement** les mêmes versions que le serveur :

1. Installer le launcher CurseForge (ou Prism Launcher).
2. Installer **All the Mods 10: To the Sky**, version indiquée dans `MODPACK_VERSION`.
3. Ajouter au profil : **MineColonies**, **Structurize**, **BlockUI**, **Domum Ornamentum**,
   **Multi-Piston**, **TownTalk** (versions 1.21.1 NeoForge — les mêmes que dans `data/mods/`
   sur le serveur).
4. Allouer 8 à 10 Go de RAM au jeu.
5. Se connecter à l'adresse du serveur (tunnel playit `xxx.tun.ply.gg:PORT`, ou `localhost` depuis le PC hôte).

## Sauvegardes

Le conteneur `backups` archive le monde toutes les 6 h dans `backups/` et supprime les
archives de plus de 7 jours. Penser à copier régulièrement une archive hors de la VM.

## Dépannage

- **NeoForge ne s'installe pas** (serveur : `Unable to resolve NeoForge metadata` / 404 ;
  CurseForge : « Forge Modloader installation failed ») : souvent une panne temporaire de
  maven.neoforged.net. Réessayer plus tard. Côté client, on peut aussi lancer l'installeur
  officiel : `java -jar neoforge-<version>-installer.jar --install-client <curseforge>\minecraft\Install`.
- Les versions exactes de MineColonies & dépendances téléchargées (`data/mods/`) : les
  épingler ensuite dans `CURSEFORGE_FILES` (`minecolonies:<fileId>`) pour éviter les
  mises à jour surprises.
