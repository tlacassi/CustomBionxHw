# Budget de veille — BionXHW

Objectif provisoire : ≤100 µA **mesurés au pack** en veille système. Courants ci-dessous issus des fiches techniques ou de calculs, sans mesure matérielle. Le BMS DALY, ses communications, l'écran, les fuites et les périphériques restent à comptabiliser.

| Bloc | Rail | Mode veille | Courant propre |
|---|---|---|---:|
| U1 TPS48111LQDGXRQ1 | Pack | EN/UVLO <0,3 V, INP/INP_G bas | 1,6 µA typ.; 5,36 µA max (48 V, −40 à 125 °C), 2,65 µA max jusqu'à 85 °C |
| U3 LMR36510ADDAR | Pack | PFM, état sans commutation selon TI | 26 µA typ.; 36 µA max, hors conversion des charges |
| U2 INA228 | 3V3 | Mode `SHUTDOWN` configuré | ≤5 µA max; mesure active 640 µA typ. |
| ESP32-S3-WROOM-1-N8 | 3V3 | Deep-sleep RTC, Wi-Fi/BT arrêtés | 7–8 µA typ. |
| SN65HVD230QDR (carte écran) | 3V3 | Écoute CAN, RS haut | 370 µA typ.; 600 µA max |
| R2/R3 retour du buck | 3V3 | Alimentées en permanence | 50 µA à 3,32 V / 66,4 kΩ |
| R4–R6 I²C/ALERT | 3V3 | Lignes hautes | 0 idéal; 706 µA par ligne maintenue basse |
| R21/R22 défauts | 3V3 | Sorties ouvertes en veille à vérifier | 0 idéal; 33,2 µA par défaut maintenu bas |
| R15–R17 entrées de commande | 3V3 | GPIO bas ou haute impédance | 0 idéal; ~33 µA par commande haute |
| Q1–Q3, R19/R20, shunt et R18 | Pack | MOSFETs arrêtés, VGS ≈0 | Fuites des MOSFETs à chiffrer à chaud et à mesurer; résistances ≈0 idéal |
| C1–C11, L1 | Rails respectifs | Polarisés sans charge supplémentaire idéale | Fuites diélectriques et pertes à mesurer |

Le LM5069 retiré prélevait à lui seul 480 µA typiques en arrêt. Les courants propres **U1 + U3** sont désormais **27,6 µA typiques** (41,36 µA en additionnant les maxima de datasheet, sous leurs conditions respectives). Les termes de charge et de fuite s'ajoutent. L'architecture ne possède ni LED d'alimentation ni diviseur permanent entre le pack et GND.

## Calcul d'ordre de grandeur

`I_pack ≈ Iq_U1 + Iq_U3 + (V_3V3 × I_3V3)/(V_pack × rendement) + autres courants pack`.

Pour 36 V pack, 3,32 V de sortie et un rendement **hypothétique** de 80 % à charge très faible :

| Mode | Charges comptées sur 3V3 | Estimation pack hors fuites, écran et BMS |
|---|---:|---:|
| CAN en écoute (PHY actuel) | 8 + 370 + 5 + 50 = 433 µA | ≈77,5 µA |
| PHY coupé (option à concevoir, sans réveil CAN) | 8 + 5 + 50 = 63 µA | ≈34,9 µA |
| PHY TCAN3413 en veille (option à vérifier) | 8 + 15 + 5 + 50 = 78 µA | ≈36,6 µA |

Ces estimations mélangent des valeurs typiques et les maxima explicitement indiqués. Le rendement à quelques centaines de µW, les salves PFM, l'état de FLT/ALERT, la tension basse du pack et la température peuvent faire dépasser 100 µA. La variante TCAN3413 requiert un changement de brochage et de rails; elle n'est pas installée. Le DALY peut dominer le courant pack si son propre réveil n'est pas maîtrisé.

## Mesure et mise à jour

- Forcer POWER_ENABLE, MAIN_ON et PRECHARGE_ON bas; vérifier au reset et en deep-sleep. Configurer l'INA228 en shutdown et relâcher SDA/SCL/ALERT.
- Débrancher USB et programmeur. Mesurer d'abord la carte puissance seule puis la carte écran avec le CAN en écoute, et enfin le BMS. Contrôler les réveils avec et sans trafic CAN.
- Mesurer à plusieurs tensions de pack et températures après stabilisation; intégrer plusieurs cycles PFM. Confirmer le courant des protections transitoires lorsque leurs références seront choisies.
- Pour chaque ajout, consigner rail, mode, courant typique/max, fuite à chaud et impact ramené au pack.

Sources : [TI TPS48111](https://www.ti.com/lit/ds/symlink/tps4811-q1.pdf), [TI LMR36510](https://www.ti.com/lit/ds/symlink/lmr36510.pdf), [TI INA228](https://www.ti.com/lit/ds/symlink/ina228.pdf), [TI SN65HVD230](https://www.ti.com/lit/ds/symlink/sn65hvd230.pdf), [TI TCAN3413](https://www.ti.com/lit/ds/symlink/tcan3413.pdf), [Espressif module ESP32-S3](https://documentation.espressif.com/esp32-s3-wroom-1_wroom-1u_datasheet_en.pdf).

## Révision avec microcontrôleur local sur la carte puissance

Le calcul précédent décrivait l'ancienne architecture. Le 3,3 V de Power Control alimente désormais son propre module U4 et son PHY U7. La carte écran garde son propre budget.

| Ajout | Veille sur Power Control |
|---|---|
| U4 ESP32-S3-WROOM-1-N8 | 7–8 µA typiques sur 3,3 V en deep sleep avec fonctions RTC appropriées. |
| U7 TCAN3413DR | 10 µA typiques, 15 µA max jusqu'à 85 °C en additionnant VCC et VIO, avec STB et TXD hauts aux conditions TI. |
| U5 LMR36510 12 V | EN bas. Ajouter le courant d'arrêt côté pack et les fuites des condensateurs. |
| U6, Q4/Q5 et charges | Sur pack seul, le 12 V coupé retire l’alimentation des accessoires après décharge. Lorsque l’USB est branché, le 5 V du buzzer reste disponible. |

Exemple pour Power Control seul à 36 V et rendement hypothétique de 80 % : 8 + 10 + 5 + 50 = 73 µA sur le 3,3 V. L'équivalent pack vaut environ 8,4 µA, soit environ 36 µA avec les courants propres typiques de U1 et U3, avant le courant d'arrêt de U5, les fuites et les tolérances. Additionner séparément les courants ramenés au pack de la carte écran et du BMS. La cible système de 100 µA reste à mesurer.

En veille sur pack seul, ajouter environ **1,32 µA typique sur le 3,3 V** pour le sélecteur U10, soit environ 0,15 µA ramené à un pack de 36 V au rendement hypothétique de 80 %. U11 est privé de source lorsque l’USB est absent et le 12 V coupé. Mesurer les fuites et la consommation réelle après intégration. [TI TPS2116](https://www.ti.com/lit/ds/symlink/tps2116.pdf)

## Display révisé

Le Display utilise désormais un TCAN3413DR. Sur pack, le rail UI et le 12 V accessoires sont coupés en veille. Prévoir environ 8 µA typiques pour le module, 10 µA typiques pour le CAN en standby et 1,32 µA typique pour le sélecteur de source, tous ramenés au rail 3,3 V. Ajouter environ 10 µA si le contact limite maintient son entrée basse via le rappel de 330 kΩ, ainsi que le courant d’arrêt du TPS22918 et les fuites. Les rappels des voies A/B d’encodeur sont sur le rail UI coupé. Un bouton de réveil maintenu appuyé tire environ 69 µA sur son rappel de 47 kΩ plus la résistance série. Les mesures devront distinguer les deux cartes et le BMS. Les anciennes estimations du Display avec SN65HVD230 sont historiques.
