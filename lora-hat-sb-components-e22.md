# LoRa HAT pour Raspberry Pi – SB Components (E22-900T22S)

| Champ | Valeur |
|---|---|
| Référence | LoRa HAT 915/868/433 MHz, variante 868 MHz |
| Module radio | Ebyte E22-900T22S (puce SX1262) |
| Quantité | 2 |
| État | Testé |
| Mise à jour | 12/11/2026 |

## 1. Caractéristiques générales

| Caractéristique | Valeur |
|---|---|
| Bande de fréquence | 868 MHz (canal par défaut 18 = 868,125 MHz) |
| Puissance d'émission | Jusqu'à 22 dBm, réglable par registre |
| Interface | UART, via le connecteur 40 broches ou via le convertisseur USB-UART CH340 intégré |
| Adaptation de niveaux | 74HC125V |
| Antenne | Connecteurs SMA et IPEX |

## 2. Fabricant

- **2.1 Nom** : SB Components Ltd (module radio : Chengdu Ebyte)
- **2.2 Site web** : https://shop.sb-components.co.uk — https://www.ebyte.com

## 3. Applications

- Liaison point à point entre deux cartes, sans infrastructure réseau
- Télémesure de capteurs éloignés
- Relais de données en zone sans couverture cellulaire
- Évaluation de portée et de qualité de liaison radio
- Prototypage de protocoles applicatifs sur liaison série radio

## 4. Procédures de test unitaire

### 4.1 Préparation matérielle

1. Mettre le Raspberry Pi hors tension et débrancher son alimentation.
2. Visser une antenne 868 MHz sur le connecteur SMA.
3. Positionner le cavalier de sélection de carte sur **MODE 2** (liaison Raspberry Pi ↔ module).
4. Positionner les cavaliers de mode :
   - **M0 et M1 en place** : mode transmission ;
   - **M0 en place, M1 retiré** : mode configuration.
5. Enficher la carte sur le connecteur 40 broches.
6. Remettre sous tension.

Prérequis côté système : port série matériel activé et accessible sur `/dev/serial0` (voir fiche Raspberry Pi 4 B).

### 4.2 Test de liaison entre deux cartes

1. Sur les deux cartes : cavaliers **M0 et M1 en place**.
2. Éloigner les antennes d'au moins 1 à 2 m.
3. Sur chaque Raspberry Pi, ouvrir le terminal série :
   `minicom -D /dev/serial0 -b 9600`
4. Activer l'écho local : `Ctrl+A` puis `E`.
5. Saisir un texte sur la première carte et vérifier son affichage sur la seconde.
6. Répéter dans l'autre sens.
7. Quitter : `Ctrl+A` puis `X`.

Résultat obtenu : transmission correcte dans les deux sens.

### 4.3 Lecture de la configuration du module

1. Retirer le cavalier **M1** (conserver **M0**), attendre 2 s.
2. Exécuter :

```bash
python3 - <<'EOF'
import serial, time
s = serial.Serial('/dev/serial0', 9600, timeout=1)
s.write(bytes([0xC1, 0x00, 0x09])); time.sleep(0.3)
print(s.read(12).hex(' '))
EOF
```

3. Résultat obtenu en sortie d'usine : `c1 00 09 00 00 00 62 00 12 03 00 00`

| Octet | Registre | Signification |
|---|---|---|
| 00 00 | ADDH / ADDL | Adresse 0x0000 |
| 00 | NETID | Réseau 0 |
| 62 | REG0 | UART 9600 bauds, 8N1, débit radio 2,4 kbit/s |
| 00 | REG1 | Paquet 240 octets, puissance 22 dBm |
| 12 | REG2 | Canal 18 → 868,125 MHz |
| 03 | REG3 | Mode transparent |
| 00 00 | CRYPT | Pas de chiffrement |

### 4.4 Réglage de la puissance d'émission

1. Module toujours en mode configuration.
2. Exécuter :

```bash
python3 - <<'EOF'
import serial, time
s = serial.Serial('/dev/serial0', 9600, timeout=1)
s.write(bytes([0xC0, 0x04, 0x01, 0x02])); time.sleep(0.3)
print(s.read(4).hex(' '))
EOF
```

3. Réponse attendue : `c1 04 01 02` (REG1 = 0x02 → 13 dBm).
4. Remettre le cavalier **M1**, attendre 2 s.
5. Répéter l'opération sur la seconde carte.
6. Refaire le test de liaison (§ 4.2) pour contrôle.

## 5. Limites et précautions

- Ne jamais mettre le module sous tension sans antenne adaptée à sa bande : risque de destruction de l'étage de sortie.
- Vérifier que la bande du module, celle de l'antenne et celle de l'interlocuteur sont identiques avant tout essai ; les variantes 433, 868 et 915 MHz ne sont pas interopérables.
- La réglementation européenne limite la puissance rayonnée dans la bande 868,0–868,6 MHz à 14 dBm, avec un taux d'occupation de 1 % : régler la puissance et l'intervalle d'émission en conséquence.
- À très courte distance, une puissance élevée peut saturer le récepteur : espacer les antennes lors des essais de paillasse.
- Les deux modules doivent partager les mêmes paramètres (adresse, canal, vitesse UART, mode) pour dialoguer.
- La carte occupe le connecteur 40 broches : prévoir un Raspberry Pi dédié ou une extension à interface USB si d'autres cartes sont nécessaires.

## 6. Interfaces et raccordement

| Élément | Repère | Rôle |
|---|---|---|
| Cavalier de sélection de carte | MODE 1 / MODE 2 | MODE 1 : pilotage par USB ; MODE 2 : pilotage par le Raspberry Pi |
| Cavaliers de mode | M0, M1 | Transmission, configuration, veille économe, sommeil profond |
| Port série côté Raspberry Pi | `/dev/serial0` | 9600 bauds, 8N1 par défaut |
| Convertisseur USB-UART | CH340 | Pilotage depuis un PC, sans Raspberry Pi |
| Antenne | SMA ou IPEX | 868 MHz |

**Correspondance des modes**

| M0 | M1 | Mode |
|---|---|---|
| En place | En place | Transmission |
| En place | Retiré | Configuration |
| Retiré | En place | Veille économe (WOR) |
| Retiré | Retiré | Sommeil profond |

## 7. Ressources

- Fiche produit : https://shop.sb-components.co.uk/products/lora-hat-433mhz-868mhz
- Documentation et exemples : https://learn.sb-components.co.uk/LoRa-HAT-for-Raspberry-Pi
- Dépôt logiciel : https://github.com/sbcshop/Lora-HAT-for-Raspberry-Pi
