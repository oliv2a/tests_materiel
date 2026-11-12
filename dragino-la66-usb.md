# Adaptateur USB LoRaWAN Dragino LA66 (V2, EU868)

| Champ | Valeur |
|---|---|
| Référence | LA66 USB Adapter V2-EU868 |
| Quantité | 1 |
| État | Testé |
| Mise à jour | 12/11/2026 |

## 1. Caractéristiques générales

| Caractéristique | Valeur |
|---|---|
| Module radio | Dragino LA66 (processeur 32 bits, 48 MHz, 256 Ko de flash) |
| Protocole | LoRaWAN v1.0.3, ou LoRa pair-à-pair selon le micrologiciel |
| Bande de fréquence | EU868 (version possédée) |
| Interface | USB, via convertisseur CP210x ; pilotage par commandes AT à 9600 bauds |
| Puissance d'émission | Jusqu'à 22 dBm, sensibilité annoncée −148 dBm |

## 2. Fabricant

- **2.1 Nom** : Dragino Technology Co., Ltd
- **2.2 Site web** : https://www.dragino.com

## 3. Applications

- Mise en service et recette d'un réseau LoRaWAN
- Mesure de couverture radio d'une passerelle
- Remontée de données d'un PC ou d'une carte embarquée vers un serveur LoRaWAN
- Banc de test pour le décodage de charge utile côté serveur
- Nœud de démonstration pour formation

## 4. Procédures de test unitaire

### 4.1 Mise en service

1. Visser l'antenne 868 MHz sur l'adaptateur.
2. Brancher l'adaptateur sur un port USB du PC.
3. Ouvrir le **Gestionnaire de périphériques** → **Ports (COM et LPT)**.
4. Relever le port **CP210x … (COMx)**.
   - Périphérique non reconnu : installer le pilote CP210x de Silicon Labs, puis rebrancher.

### 4.2 Ouverture du terminal

1. Lancer PuTTY → type de connexion **Serial**.
2. Serial line : **COMx** ; Speed : **9600**.
3. Menu **Terminal** : cocher **Implicit CR in every LF** et **Local echo : Force on**.
4. Cliquer **Open**.

### 4.3 Identification du micrologiciel

1. Redémarrer le module : saisir `ATZ` puis `Entrée`.
   - Variante : débrancher et rebrancher l'adaptateur.
2. Observer les messages de démarrage (fréquences d'émission, tentatives de connexion).
3. Saisir `AT+VER=?` → relève la version et la bande.
   - Résultat obtenu : `EU868 v1.3` (micrologiciel LoRaWAN).
4. Saisir `AT+DEUI=?` → relève l'identifiant unique du module.
5. Saisir `AT+APPEUI=?` et `AT+APPKEY=?` → relève les clés nécessaires à l'enregistrement.

### 4.4 Enregistrement sur un serveur LoRaWAN

1. Ouvrir la console du serveur réseau et créer une application.
2. Choisir l'ajout manuel d'un équipement.
3. Renseigner :
   - plan de fréquences : Europe 863-870 MHz, SF9 en fenêtre RX2 ;
   - version LoRaWAN : 1.0.3 ;
   - paramètres régionaux : RP001 1.0.3 révision A ;
   - JoinEUI, DevEUI et AppKey relevés au § 4.3.
4. Valider l'enregistrement.

### 4.5 Test de connexion et d'émission

1. Éloigner l'adaptateur de la passerelle d'au moins 2 m.
2. Dans le terminal, saisir `ATZ`.
3. Dans la console du serveur, onglet de suivi en direct : vérifier l'acceptation de la demande de connexion et l'attribution d'une adresse réseau.
4. Envoyer un message applicatif :
   `AT+SENDB=00,2,4,01020304`
   - `00` : sans accusé de réception ; `2` : port applicatif ; `4` : nombre d'octets ; `01020304` : données en hexadécimal.
5. Vérifier dans la console la réception du message et relever la qualité radio.
   - Résultat obtenu : données `01 02 03 04`, port 2, RSSI −81 dBm, SNR +8,75 dB.

### 4.6 Activation du débit adaptatif

1. Saisir `AT+ADR=?` pour lire l'état (`1` = activé).
2. Le cas échéant, saisir `AT+ADR=1`.
3. Envoyer plusieurs messages espacés d'au moins une minute et vérifier l'évolution du facteur d'étalement.

## 5. Limites et précautions

- Ne jamais mettre l'adaptateur sous tension sans antenne.
- Le module est un équipement terminal : son fonctionnement en LoRaWAN suppose une passerelle à portée et un serveur réseau. Vérifier la couverture avant tout essai.
- Les identifiants et clés sont des données sensibles : les conserver hors de tout document partagé.
- La bande du module doit correspondre à celle du réseau visé ; les variantes régionales ne sont pas interchangeables.
- La réglementation européenne limite le taux d'occupation du canal à 1 % : espacer les émissions, d'autant plus aux facteurs d'étalement élevés où la durée d'un message dépasse la seconde.
- La modulation LoRa est commune à de nombreux modules, mais le format des trames diffère selon le protocole : l'interopérabilité avec un module d'une autre famille n'est pas acquise et doit être vérifiée au cas par cas.
- Le passage du mode LoRaWAN au mode pair-à-pair exige un changement de micrologiciel.

## 6. Interfaces et raccordement

| Interface | Caractéristique |
|---|---|
| USB | Convertisseur CP210x, port série virtuel |
| Port série | 9600 bauds, 8N1, commandes AT |
| Alimentation | 5 V par le port USB |
| Antenne | Connecteur SMA ou antenne ressort selon version |

**Commandes AT utilisées**

| Commande | Fonction |
|---|---|
| `ATZ` | Redémarrage du module |
| `AT+VER=?` | Version du micrologiciel et bande |
| `AT+DEUI=?` | Identifiant unique de l'équipement |
| `AT+APPEUI=?` | Identifiant d'application |
| `AT+APPKEY=?` | Clé d'application |
| `AT+ADR=?` / `AT+ADR=1` | Lecture et activation du débit adaptatif |
| `AT+SENDB=…` | Envoi d'un message en hexadécimal |

## 7. Ressources

- Manuel de l'adaptateur USB : https://wiki.dragino.com/xwiki/bin/view/Main/User%20Manual%20for%20LoRaWAN%20End%20Nodes/LA66%20USB%20LoRaWAN%20Adapter%20User%20Manual/
- Manuel du module LA66 : https://wiki.dragino.com/xwiki/bin/view/Main/User%20Manual%20for%20LoRaWAN%20End%20Nodes/LA66%20LoRaWAN%20Module/
- Exemples de code : https://github.com/dragino/LA66
