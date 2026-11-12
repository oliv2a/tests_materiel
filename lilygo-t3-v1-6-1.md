# LILYGO T3 V1.6.1 (LoRa32, ESP32 + SX1276)

| Champ | Valeur |
|---|---|
| Référence | LILYGO T3 V1.6.1, également désignée TTGO LoRa32 V2.1 (1.6.1) ou LoRa32-OLED |
| Variante | 868 / 915 MHz |
| Quantité | 1 |
| État | Testé (mise en service, afficheur, identifiant radio) |
| Mise à jour | 12/11/2026 |

## 1. Caractéristiques générales

| Caractéristique | Valeur |
|---|---|
| Processeur | ESP32 deux cœurs, 240 MHz, 4 Mo de flash |
| Radio | Semtech SX1276, bande 868 / 915 MHz |
| Affichage | OLED SSD1306 0,96 pouce, 128 × 64, bus I²C |
| Connectique | Micro-USB (convertisseur CH9102), lecteur microSD, connecteur de batterie, connecteurs d'extension au pas de 2,54 mm |
| Sans fil complémentaire | Wi-Fi 2,4 GHz, Bluetooth et Bluetooth à basse consommation |

## 2. Fabricant

- **2.1 Nom** : Shenzhen Xin Yuan Electronic Technology (LILYGO)
- **2.2 Site web** : https://www.lilygo.cc

## 3. Applications

- Nœud LoRaWAN autonome avec affichage local
- Liaison LoRa point à point entre deux cartes
- Mesure de couverture radio en déplacement
- Comptage d'appareils Wi-Fi et Bluetooth à proximité
- Support de développement pour capteurs connectés

## 4. Procédures de test unitaire

### 4.1 Préparation du poste de développement

1. Installer le pilote du convertisseur **CH9102** (série CH343 du fournisseur WCH).
2. Installer **Arduino IDE 2**.
3. **Fichier → Préférences** → ajouter l'URL de gestionnaire de cartes :
   `https://espressif.github.io/arduino-esp32/package_esp32_index.json`
4. **Outils → Carte → Gestionnaire de cartes** → installer le paquet **esp32** (Espressif).
5. **Croquis → Inclure une bibliothèque → Gérer les bibliothèques** → installer **RadioLib**.
6. **Outils → Carte** : sélectionner **TTGO LoRa32-OLED**, puis **Board Revision : TTGO LoRa32 V2.1 (1.6.1)**.

### 4.2 Mise sous tension et lecture de l'identifiant

1. Visser l'antenne correspondant à la bande de la carte.
2. Brancher la carte en micro-USB.
3. Ouvrir le moniteur série à **115200** bauds.
4. Appuyer brièvement sur le bouton **RST**.
5. Lire les informations affichées sur l'écran au démarrage : nom et version du programme, caractéristiques du processeur, puis identifiant radio de l'équipement.
   - Résultat obtenu : programme Paxcounter v3.2.1, ESP32 deux cœurs, 4 Mo de flash, identifiant radio affiché sous forme de code à barres bidimensionnel et de quatre groupes hexadécimaux.
6. Appuyer brièvement sur le bouton **BOOT** pour faire défiler les pages de l'afficheur.

### 4.3 Test d'émission avec un programme de contrôle

1. Ouvrir le fichier `t3_test1.ino` dans Arduino IDE.
2. Vérifier la carte et le port sélectionnés.
3. Cliquer **Téléverser**.
   - Échec de connexion : maintenir **BOOT**, appuyer brièvement sur **RST**, relâcher **BOOT**, puis relancer.
4. Ouvrir le moniteur série à **115200** bauds et appuyer sur **RST**.
5. Résultat attendu : message d'initialisation de la radio, puis une ligne d'émission toutes les 30 s, accompagnée d'un éclat de la LED.
   - Clignotement rapide de la LED : échec d'initialisation ; relever le code d'erreur affiché.

Paramètres du programme de contrôle : 868,1 MHz, largeur de bande 125 kHz, facteur d'étalement 9, puissance 10 dBm, une émission toutes les 30 s.

### 4.4 Mise en service LoRaWAN

1. Enregistrer la carte sur le serveur réseau en utilisant l'identifiant relevé au § 4.2 et faire générer la clé d'application.
2. Installer **Git**, **VS Code** et l'extension **PlatformIO IDE**.
3. Récupérer le projet correspondant au programme embarqué.
4. Copier le fichier de configuration d'exemple sous le nom attendu par le projet, et y reporter les identifiants et la clé au format **MSB**.
5. Sélectionner la cible matérielle correspondant à la carte dans le fichier de configuration du projet.
6. Compiler et téléverser par le port USB.
7. Contrôler dans la console du serveur l'acceptation de la demande de connexion puis l'arrivée des messages applicatifs.

## 5. Limites et précautions

- Ne jamais mettre la carte sous tension sans antenne vissée.
- La variante de la carte, celle de l'antenne et celle du réseau visé doivent correspondre : vérifier la bande imprimée sur le module et sur l'antenne.
- Les broches du processeur numérotées à partir de 33 sont en entrée seule : en tenir compte lors du choix des extensions.
- Un grand nombre de broches est déjà affecté à la radio, à l'afficheur et au lecteur de carte : vérifier la table de brochage de la révision exacte avant tout raccordement, les affectations différant d'une révision à l'autre.
- Le téléversement d'un nouveau programme efface celui déjà présent : relever au préalable les informations utiles affichées au démarrage.
- La réglementation européenne limite la puissance et le taux d'occupation du canal : régler la puissance d'émission et l'intervalle entre messages en conséquence.
- Un programme de comptage d'appareils à proximité traite des données issues d'équipements personnels : en usage réel, vérifier le cadre réglementaire applicable à la protection des données.

## 6. Interfaces et raccordement

**Brochage de la révision V1.6.1**

| Fonction | Broche |
|---|---|
| Radio – horloge série | 5 |
| Radio – entrée de données | 19 |
| Radio – sortie de données | 27 |
| Radio – sélection | 18 |
| Radio – réinitialisation | 23 |
| Radio – interruption principale | 26 |
| Radio – interruptions secondaires | 33 et 32 |
| Afficheur – données I²C | 21 |
| Afficheur – horloge I²C | 22 |
| Carte microSD – sélection | 13 |
| Carte microSD – entrée / sortie / horloge | 15 / 2 / 14 |
| Mesure de tension de batterie | 35 |
| LED embarquée | 25 |
| Bouton utilisateur | 0 |

**Éléments de commande**

| Élément | Rôle |
|---|---|
| Bouton RST | Redémarrage |
| Bouton BOOT | Mode programmation, et navigation dans les pages de l'afficheur |
| Micro-USB | Alimentation, programmation, liaison série |
| Connecteur SMA / IPEX | Antenne radio |

## 7. Ressources

- Documentation constructeur : https://wiki.lilygo.cc/products/t3-series/t3-lora32/
- Guide de démarrage : https://wiki.lilygo.cc/products/t3-series/t3-lora32/quick-start.html
- Dépôt des exemples : https://github.com/Xinyuan-LilyGO/LilyGo-LoRa-Series
- Bibliothèque radio RadioLib : https://github.com/jgromes/RadioLib
- Configuration ESPHome de référence : https://devices.esphome.io/devices/lilygo-t3-lora32-v1.6.1
