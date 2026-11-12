# Raspberry Pi 4 Model B

| Champ | Valeur |
|---|---|
| Référence | Raspberry Pi 4 Model B – 8 Go |
| Quantité | 2 (nommés `lora1` et `lora2`) |
| État | Testé |
| Mise à jour | 12/11/2026 |

## 1. Caractéristiques générales

| Caractéristique | Valeur |
|---|---|
| Processeur | Broadcom BCM2711, quatre cœurs Cortex-A72, 1,5 GHz |
| Mémoire vive | 8 Go LPDDR4 |
| Connectique | 2 × USB 3.0, 2 × USB 2.0, Ethernet gigabit, 2 × micro-HDMI, DSI, CSI |
| Entrées/sorties | Connecteur 40 broches (UART, SPI, I²C, GPIO) |
| Alimentation | 5 V / 3 A par USB-C |

## 2. Fabricant

- **2.1 Nom** : Raspberry Pi Ltd
- **2.2 Site web** : https://www.raspberrypi.com

## 3. Applications

- Hôte de cartes d'extension radio (HAT LoRa, concentrateur LoRaWAN)
- Serveur réseau LoRaWAN autohébergé (ChirpStack)
- Passerelle de protocoles (série, MQTT, HTTP)
- Enregistrement et traitement de données de capteurs
- Poste de supervision avec écran local

## 4. Procédures de test unitaire

### 4.1 Identifier la quantité de mémoire

1. Ouvrir une session SSH.
2. Saisir : `free -h`
3. Lire la colonne **total** de la ligne **Mem**.
   - Résultat obtenu : `7.6Gi` → modèle 8 Go.

### 4.2 Vérifier la version du système

1. Saisir : `cat /etc/os-release`
2. Relever la ligne **VERSION**.
   - Résultat obtenu : Raspberry Pi OS Lite 64 bits, Trixie.

### 4.3 Activer et contrôler le port série matériel

1. Saisir : `sudo raspi-config`
2. **3 Interface Options** → **Serial Port**.
3. « Login shell over serial ? » → **No**.
4. « Serial port hardware enabled ? » → **Yes**.
5. **Finish**.
6. Éditer la configuration : `sudo nano /boot/firmware/config.txt`
7. Ajouter en fin de fichier, sous `[all]` : `dtoverlay=disable-bt`
8. Enregistrer (`Ctrl+O`, `Entrée`, `Ctrl+X`).
9. Désactiver le service associé : `sudo systemctl disable hciuart`
10. Redémarrer : `sudo reboot`
11. Contrôler le lien symbolique : `ls -l /dev/serial0`
    - Résultat attendu : `/dev/serial0 -> ttyAMA0`

### 4.4 Vérifier les droits d'accès au port série

1. Saisir : `groups`
2. Vérifier la présence de **dialout** dans la liste.

### 4.5 Installer les outils de test série

1. Saisir : `sudo apt update && sudo apt full-upgrade -y`
2. Saisir : `sudo apt install -y python3-serial minicom`

## 5. Limites et précautions

- Le connecteur 40 broches n'accepte qu'une seule carte d'extension à la fois : prévoir un Pi par fonction radio, ou choisir une extension à interface USB pour libérer le connecteur.
- Le port série matériel est partagé avec le Bluetooth : sa désactivation est nécessaire pour obtenir une liaison série fiable.
- L'alimentation doit être dimensionnée pour le Pi **et** ses extensions : viser 5 V / 3 A minimum, davantage si un écran et une carte radio sont ajoutés.
- Une installation Lite ne comporte pas d'environnement graphique : prévoir une image avec bureau si l'affichage d'une interface est requis.
- Prévoir une dissipation thermique (boîtier ventilé ou radiateurs) en fonctionnement continu.

## 6. Interfaces et raccordement

| Interface | Repère | Usage dans le projet |
|---|---|---|
| Connecteur 40 broches | GPIO | Carte d'extension radio |
| Port série matériel | `/dev/serial0` → `ttyAMA0` | Dialogue avec un module radio |
| DSI | Connecteur nappe | Écran tactile |
| Ethernet / Wi-Fi | RJ45 / intégré | Administration, SSH, réseau |
| USB-C | Alimentation | 5 V / 3 A |

## 7. Ressources

- Documentation officielle : https://www.raspberrypi.com/documentation/
- Téléchargement des images système : https://www.raspberrypi.com/software/
- Brochage du connecteur 40 broches : https://pinout.xyz
