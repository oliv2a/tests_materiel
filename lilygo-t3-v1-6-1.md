# LILYGO T3 V1.6.1 (LoRa32, ESP32 + SX1276)

| Champ | Valeur |
|---|---|
| Référence | LILYGO T3 V1.6.1, également désignée TTGO LoRa32 V2.1 (1.6.1) ou LoRa32-OLED |
| Variante | 868 / 915 MHz |
| Quantité | 1 |
| État | Testé (mise en service, afficheur, liaison LoRaWAN opérationnelle) |
| Mise à jour | 17/11/2026 |

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

Procédure établie avec le programme embarqué d'origine (comptage d'appareils à proximité), recompilé avec les paramètres du réseau visé.

#### 4.4.1 Enregistrement sur le serveur réseau

1. Ouvrir la console du serveur réseau et créer une application.
2. Choisir l'ajout manuel d'un équipement.
3. Renseigner :
   - plan de fréquences : Europe 863-870 MHz, SF9 en fenêtre RX2 ;
   - version LoRaWAN : 1.0.3 ; paramètres régionaux : RP001 1.0.3 révision A ;
   - identifiant de l'équipement : valeur relevée au § 4.2 ;
   - identifiant d'application : valeur proposée par la console ;
   - clé d'application : faire générer.
4. Valider l'enregistrement.
5. Pour chacune des trois valeurs, afficher le format **MSB** (bouton de changement de représentation) et les conserver dans cet ordre.

#### 4.4.2 Préparation de l'environnement de compilation

1. Installer **Git**, **VS Code**, puis l'extension **PlatformIO IDE**.
2. Récupérer le projet correspondant au programme embarqué.
3. Ouvrir le dossier du projet dans VS Code.

#### 4.4.3 Fichiers de configuration

Dans le projet, chaque fichier de configuration se crée en copiant son modèle, puis en renommant la copie. Vérifier qu'aucun modèle ne reste sans copie :

| Modèle | Copie à créer |
|---|---|
| `platformio_orig.ini` | `platformio.ini` |
| `shared/paxcounter_orig.conf` | `shared/paxcounter.conf` |
| `shared/loraconf_sample.h` | `shared/loraconf.h` |
| `shared/ota_sample.conf` | `shared/ota.conf` |

#### 4.4.4 Paramétrage

1. Dans `platformio.ini`, renseigner la cible matérielle correspondant à la révision de la carte (fichier du dossier `shared/hal`).
2. Dans `platformio.ini`, figer la version de la bibliothèque de pilotage des LED en indiquant un numéro de version explicite dans la liste des dépendances ; sans cela, la version la plus récente est téléchargée et peut être incompatible avec le cadre logiciel du projet.
3. Dans `shared/lmic_config.h`, vérifier que la région européenne est active et qu'aucune autre région n'est décommentée.
4. Dans `shared/loraconf.h`, reporter les trois valeurs au format **MSB**, en respectant strictement la correspondance des champs :

| Champ du fichier | Valeur de la console |
|---|---|
| `DEVEUI` | identifiant de l'équipement |
| `APPEUI` | identifiant d'application |
| `APPKEY` | clé d'application |

#### 4.4.5 Compilation et téléversement

1. Fermer tout moniteur série occupant le port de la carte.
2. Lancer la tâche **Upload** de l'environnement USB.
3. En cas d'échec de connexion, consulter le § 5 (mode programmation).
4. Attendre l'indication de succès et la vérification de l'empreinte.

#### 4.4.6 Contrôle du fonctionnement

1. Ouvrir le moniteur série à **115200** bauds et appuyer sur **RST**.
2. Suivre la séquence attendue : demande de connexion, émission, ouverture des fenêtres de réception, puis connexion établie.
3. Dans la console du serveur, onglet de suivi en direct de l'équipement : vérifier l'acceptation de la demande de connexion, puis l'arrivée régulière des messages applicatifs.

Résultat obtenu : connexion établie et messages reçus par le serveur.

### 4.5 Diagnostic d'une connexion qui n'aboutit pas

1. Augmenter le niveau de journalisation dans `platformio.ini`, recompiler, puis relire le journal de démarrage.
2. Interpréter la séquence :

| Journal de la carte | Suivi de la passerelle | Interprétation |
|---|---|---|
| Aucune trace d'émission | — | La pile radio ne démarre pas : vérifier la cible matérielle et la région |
| Émission puis attente sans réponse | Aucune trame reçue | Problème de portée, d'antenne ou de bande |
| Émission puis attente sans réponse | Trame reçue | Valeurs d'identification non conformes à l'enregistrement |

3. Dans ce dernier cas, comparer octet par octet les trois valeurs du fichier de configuration avec celles de la console, au format MSB, en vérifiant qu'aucune n'a été reportée dans le mauvais champ.

## 5. Limites et précautions

- Ne jamais mettre la carte sous tension sans antenne vissée.
- La variante de la carte, celle de l'antenne et celle du réseau visé doivent correspondre : vérifier la bande imprimée sur le module et sur l'antenne.
- Les broches du processeur numérotées à partir de 33 sont en entrée seule : en tenir compte lors du choix des extensions.
- Un grand nombre de broches est déjà affecté à la radio, à l'afficheur et au lecteur de carte : vérifier la table de brochage de la révision exacte avant tout raccordement, les affectations différant d'une révision à l'autre.
- Le téléversement d'un nouveau programme efface celui déjà présent : relever au préalable les informations utiles affichées au démarrage.
- La réglementation européenne limite la puissance et le taux d'occupation du canal : régler la puissance d'émission et l'intervalle entre messages en conséquence.
- Un programme de comptage d'appareils à proximité traite des données issues d'équipements personnels : en usage réel, vérifier le cadre réglementaire applicable à la protection des données.
- La carte ne comporte ni bouton de programmation ni bouton utilisateur : seuls un interrupteur d'alimentation et un bouton de réinitialisation sont présents. Le passage en mode programmation est normalement automatique ; en cas d'échec, relier brièvement la broche d'entrée **IO0** à la masse pendant la réinitialisation, puis relâcher. La même broche sert à faire défiler les pages de l'afficheur en fonctionnement normal.
- Les dépendances logicielles du projet doivent être figées à une version compatible : une version plus récente qu'au moment de la publication du projet peut interrompre la compilation.
- Les avertissements de l'analyseur de code de l'éditeur concernant des types non reconnus dans les fichiers de configuration sont sans effet sur la compilation.
- Le lecteur de carte mémoire signale son absence au démarrage : ce message est normal si aucune carte n'est insérée.

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
| Interrupteur ON / OFF | Alimentation de la carte |
| Broche IO0 (connecteur latéral) | Mode programmation par mise à la masse, et navigation dans les pages de l'afficheur |
| Micro-USB | Alimentation, programmation, liaison série à 115200 bauds |
| Connecteur SMA / IPEX | Antenne radio |

## 7. Ressources

- Documentation constructeur : https://wiki.lilygo.cc/products/t3-series/t3-lora32/
- Guide de démarrage : https://wiki.lilygo.cc/products/t3-series/t3-lora32/quick-start.html
- Dépôt des exemples : https://github.com/Xinyuan-LilyGO/LilyGo-LoRa-Series
- Bibliothèque radio RadioLib : https://github.com/jgromes/RadioLib
- Configuration ESPHome de référence : https://devices.esphome.io/devices/lilygo-t3-lora32-v1.6.1
