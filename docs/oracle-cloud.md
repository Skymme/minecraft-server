# Héberger sur Oracle Cloud (Always Free)

L'offre Always Free donne jusqu'à **4 OCPU ARM (Ampere A1) + 24 Go de RAM**, gratuitement.
C'est largement assez pour ATM9 To the Sky avec 4 joueurs.

## 1. Créer le compte

- S'inscrire sur <https://www.oracle.com/cloud/free/>.
- Une carte bancaire est demandée pour vérification (pas de débit tant qu'on reste en Always Free).
- Choisir une **région d'origine** proche (ex. Paris, Marseille, Francfort) : on ne peut plus la changer.

## 2. Créer la VM

Dans *Compute → Instances → Create instance* :

- **Image** : Ubuntu 22.04 ou 24.04 (aarch64).
- **Shape** : `VM.Standard.A1.Flex` — 4 OCPU, 24 Go RAM.
- **Réseau** : laisser le VCN par défaut, avec une IP publique.
- **Clé SSH** : générer/importer une clé, garder la clé privée précieusement.
- **Disque de boot** : 100 Go (inclus dans le quota gratuit de 200 Go).

> Si Oracle répond « Out of capacity », réessayer plus tard ou dans un autre Availability Domain.

## 3. Ouvrir le port 25565

Deux pare-feux à configurer :

1. **Oracle (Security List)** : *Networking → VCN → Security Lists → Default* →
   *Add Ingress Rule* : source `0.0.0.0/0`, protocole TCP, port de destination `25565`.
2. **Sur la VM** (Ubuntu Oracle a des règles iptables par défaut) :

   ```bash
   sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 25565 -j ACCEPT
   sudo netfilter-persistent save
   ```

## 4. Installer Docker et déployer

```bash
ssh ubuntu@IP_PUBLIQUE
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER && exit   # puis se reconnecter

git clone <URL_DU_REPO> minecraft-server
cd minecraft-server
cp .env.example .env && nano .env        # MEMORY=12G sur la VM A1
docker compose up -d
docker compose logs -f mc
```

## 5. Mettre à jour

```bash
cd minecraft-server
git pull
docker compose up -d
```
