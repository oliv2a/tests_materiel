# KinCony N10 – Compteur d'énergie 10 voies

| Champ | Valeur |
|---|---|
| Référence | KinCony N10, 10 Channel AC Energy Meter |
| Quantité | 1 |
| État | Testé (liaison Ethernet et interface web) |
| Mise à jour | 12/11/2026 |

## 1. Caractéristiques générales

| Caractéristique | Valeur |
|---|---|
| Processeur | ESP32-S3-WROOM-1U (16 Mo de flash, 8 Mo de PSRAM) |
| Mesure | 10 voies de courant par transformateurs de courant (rapport 2000:1), 1 voie de tension alternative 0–380 V, puce de mesure BL0910 |
| Communication | Ethernet 100 Mbit/s, Wi-Fi, Bluetooth, RS485, LoRa, I²C, USB-C |
| Alimentation | 12 à 24 V continu |
| Montage | Rail DIN, 122 × 83 × 60 mm, afficheur OLED, carte SD |

## 2. Fabricant

- **2.1 Nom** : Hangzhou KinCony Electronics Co., Ltd
- **2.2 Site web** : https://www.kincony.com

## 3. Applications

- Mesure de consommation électrique répartie par départ de tableau
- Suivi de production photovoltaïque ou de bornes de recharge
- Remontée de données vers une supervision domotique par MQTT
- Journalisation locale des consommations sur carte SD
- Transmission de mesures par liaison radio vers un site distant

## 4. Procédures de test unitaire

### 4.1 Mise en service et raccordement réseau

1. Visser les antennes sur les connecteurs **ESP** (Wi-Fi) et **LoRa**.
2. Relier le port Ethernet de la carte à la box ou au commutateur du réseau local.
3. Raccorder l'alimentation 12 V continu en respectant la polarité.
4. Mettre sous tension et observer :
   - LED **POWER** allumée ;
   - LED du port Ethernet active ;
   - afficheur OLED allumé.

### 4.2 Diagnostic par la liaison série

1. Relier le port **USB-C** de la carte au PC.
2. Gestionnaire de périphériques → relever le port **COMx** apparu.
3. Ouvrir PuTTY → **Serial**, **COMx**, **115200** bauds.
4. Appuyer brièvement sur le bouton **RESET-ESP**.
5. Lire les messages affichés.
   - Absence de messages réseau (adresse IP, interface) : le programme en place n'assure pas la fonction réseau ; procéder à l'installation du micrologiciel officiel (§ 4.3).

### 4.3 Installation du micrologiciel KCS v3

1. Télécharger l'outil de programmation Espressif :
   `https://www.kincony.com/wp-content/uploads/2022/08/flash_download_tool_3.9.2.zip`
2. Sur le forum du fabricant, rubrique micrologiciel KCS v3, télécharger l'archive correspondant au **modèle N10** dans sa version la plus récente, puis la décompresser pour obtenir le fichier `.bin`.
3. Fermer tout terminal série occupant le port COM.
4. Lancer `flash_download_tool_3.9.2.exe` et choisir :
   - ChipType : **ESP32-S3** ;
   - WorkMode : **Develop** ;
   - LoadMode : **USB**.
5. Première ligne : cocher la case, sélectionner le fichier `.bin`, saisir l'adresse **0x0**.
6. Sélectionner le port **COMx**.
7. Placer la carte en mode téléchargement :
   - couper l'alimentation 12 V et débrancher l'USB-C ;
   - maintenir le bouton **DOWNLOAD** ;
   - rebrancher l'alimentation et l'USB-C ;
   - relâcher **DOWNLOAD**.
8. Cliquer **ERASE**, attendre la fin de l'effacement.
9. Répéter l'étape 7, puis cliquer **START**.
10. Attendre l'indication **FINISH**.
11. Débrancher l'USB-C, couper l'alimentation, attendre 5 s, remettre sous tension.

### 4.4 Accès à l'interface web

1. Attendre environ une minute après la mise sous tension.
2. Relever l'adresse IP de la carte :
   - sur l'afficheur OLED ;
   - ou dans la liste des équipements du routeur ;
   - ou avec l'outil de recherche du fabricant : `https://www.kincony.com/download/KinCony-SCAN_Device.zip`
     (sélectionner la carte réseau, puis l'adresse IP du PC, cliquer **StartMonitorPort** puis **SCAN**).
3. Ouvrir l'adresse dans un navigateur.
4. S'authentifier avec les identifiants par défaut **admin / admin**.
5. Vérifier la page d'accueil.
   - Résultat obtenu : modèle **N10**, version **v3.27.4**, adresse IP, masque, passerelle et adresses matérielles affichés.

### 4.5 Configuration de base

1. Menu **System** : régler le fuseau horaire et l'heure d'été, puis contrôler l'heure affichée.
2. Menu **System** : remplacer le mot de passe par défaut.
3. Sur le routeur : réserver l'adresse IP de la carte à partir de son adresse matérielle Ethernet.
4. Menu **Protocol** : laisser la fonction radio désactivée tant que le module correspondant n'est pas en service.
5. Menu **System** → **Export Config** : enregistrer une sauvegarde de la configuration.

### 4.6 Connexion par point d'accès Wi-Fi (secours)

1. Antenne vissée sur le connecteur **ESP**.
2. Rechercher le réseau Wi-Fi portant le nom de la carte suivi de son identifiant, sans mot de passe.
3. S'y connecter, puis ouvrir `http://192.168.4.1`.
   - Réseau absent : maintenir le bouton de fonction plus de 10 s pour revenir à la configuration d'usine.

## 5. Limites et précautions

- Le raccordement de la voie de tension se fait sur le réseau alternatif : intervenir hors tension, dans un tableau protégé, et respecter les règles d'installation électrique applicables.
- Les entrées de mesure attendent des transformateurs de courant à sortie en milliampères, de rapport 2000:1. Vérifier le rapport et la nature de la sortie indiqués sur l'étiquette du capteur avant tout raccordement, et retenir un calibre adapté au courant réellement mesuré.
- Un transformateur de courant à secondaire de forte intensité ne doit jamais rester en circuit ouvert autour d'un conducteur en charge : conserver les fils en court-circuit tant qu'ils ne sont pas raccordés.
- Les antennes doivent être en place avant toute activation des fonctions radio.
- La bande du module radio embarqué doit correspondre à celle des équipements avec lesquels la carte doit dialoguer : la vérifier sur le module et sur l'antenne, et la préciser à la commande.
- L'enregistrement local sur carte SD et la fonction radio sont exclusifs l'un de l'autre.
- L'adresse IP obtenue par le réseau peut changer au redémarrage : prévoir une réservation sur le routeur.

## 6. Interfaces et raccordement

| Repère | Fonction |
|---|---|
| L1 / N1 / V | Voie de mesure de tension alternative |
| CT 1 à 10 | Entrées des transformateurs de courant |
| 12-24 V (+ / −) | Alimentation continue |
| RS485 (A / B) | Bus série industriel |
| I²C (SDA / SCL) | Capteurs additionnels |
| GPIO 15 / 16 | Entrées-sorties libres |
| USB-C | Programmation et liaison série de diagnostic |
| DOWNLOAD / RESET-ESP / RESET-ARM | Boutons de programmation et de réinitialisation |
| ANTENNA LoRa / ESP | Connecteurs d'antenne |
| ETHERNET | Réseau local |

**Indicateurs**

| LED | Fonction |
|---|---|
| POWER | Présence de l'alimentation |
| RUN | Exécution du programme |
| ERROR | Défaut |

## 7. Ressources

- Fiche produit : https://www.kincony.com/?p=7632
- Guide d'utilisation du micrologiciel : https://www.kincony.com/how-to-use-kcsv3-firmware-energy-meter.html
- Forum et téléchargement des micrologiciels : https://www.kincony.com/forum/
- Exemple de configuration ESPHome : https://devices.esphome.io/devices/KinCony-N10
