# Phase 3 — BOM, références et empreintes (2026-11-16)

Fichiers : `docs/BOM_BionX_Power_Control.csv` et `docs/BOM_BionX_Display.csv` (export `kicad-cli`, champs MPN, LCSC, DNP, Montage). Prix et stocks relevés le 2026-11-16 sur LCSC (fiche produit) et sur l'index JLCPCB `jlcsearch.tscircuit.com`. Le stock d'assemblage JLCPCB lui-même n'a pas pu être lu : à recontrôler dans le devis JLCPCB.

Aucune ligne sans empreinte. ERC à 0 sur les trois projets.

## Synthèse par carte (prix unitaires JLCPCB, hors frais)

| Carte | Pièces assemblées | Lignes Basic | Lignes Extended | Lignes à souder à la main |
|---|---:|---:|---:|---:|
| Power Control | 28,77 USD | 23 | 21 | 9 |
| Display | 7,22 USD | 16 | 6 | 7 |

Les deux MOSFET CSD19536KTT (9,37 USD), le module ESP32-S3 (4,92 USD), les deux LMR36510 (3,56 USD) et le TPS48111 (3,27 USD) font les trois quarts du coût de Power Control.

## Valeurs modifiées pour rester en pièces Basic

Ces valeurs remplacent celles de `PHASE2_SCHEMAS.md`.

| Repère | Avant | Maintenant | Effet |
|---|---|---|---|
| R11 (surintensité) | 31,6 kΩ | **30 kΩ** | `11,9 × 100 / (30 000 × 0,001)` = **39,7 A** |
| R10 (court-circuit) | 3,4 kΩ | **3,3 kΩ** | `(3300 + 464) × 15,6 µA / 1 mΩ` = **58,7 A** |
| R2 / R3 (buck 3,3 V) | 46,4 kΩ / 20 kΩ | **47 kΩ / 20 kΩ** | 3,35 V |
| R24 / R25 (buck 5 V) | 100 kΩ / 24,9 kΩ | **82 kΩ / 20 kΩ** | 5,10 V |
| C2, C13 (entrée des bucks) | 220 nF / 100 V | **100 nF / 100 V** | même pièce que C9 ; [H] découplage HF suffisant avec les 2,2 µF |

## Changements de composants

| Repère | Choix | LCSC | Stock LCSC | Raison |
|---|---|---|---:|---|
| U2 | **INA237AIDGSR** au lieu de l'INA228 | C2864837 | 6628 | L'INA228AIDGSR est à **0 en stock** (C2887910, 6,79 USD). L'INA237 a le même brochage, tient 85 V, coûte 1,84 USD. Il mesure sur 16 bits et n'a pas les accumulateurs de charge et d'énergie : le firmware intègre lui-même. |
| D1, D101 | PESD1CAN (BORN) | C343993 | 50 740 | Le PESD2CANFD24V-T n'est pas au catalogue JLCPCB. Même boîtier SOT-23, même brochage, 24 V. |
| D2, D3 | **SMCJ48A** | C19077611 | 5445 | La SMCJ43A n'est pas au catalogue JLCPCB. Veille 48 V, écrêtage max 77,4 V à 19,4 A. |
| Q3, Q4, Q6 | 5N10-MS (100 V, 5 A, SOT-23) | C5140036 | 195 740 | Précharge (0,4 A) et feux en tension pack. Q3 passe du TO-263 au SOT-23. |
| U12, U13 | EL357N(C)(TA)-G | C42379244 | 172 725 | Optocoupleurs de l'UART DALY, boîtier SOP-4 au pas de 2,54 mm. |
| L1, L2 | SRG0630-220-M (22 µH, 3 A) | C53462993 | 3435 | Une seule référence pour les deux bucks. |
| R1 | RLP25FEGR001 (1 mΩ, 3 W, 1 %, ±50 ppm) | C459678 | 18 160 | Shunt 2512 à deux bornes sur l'empreinte projet `R_Shunt_2512_Kelvin` (prises de mesure séparées). |
| R18, R39 | RC2512JK-07220RL (220 Ω, 1 W) | C137008 | 2810 | Voir la réserve ci-dessous. |
| F1, F2 | SMD1812P050TF/60 | C462518 | 8950 | Fusibles réarmables des feux. |
| BZ1 | TMB12A05 (5 V actif, traversant) | C96093 | 90 335 | À souder à la main. |

## Réserves à lever

1. **TVS SMCJ48A [H]** : son écrêtage maximal (77,4 V au courant d'impulsion nominal) dépasse les 70 V absolus du LMR36510. Aux quelques ampères d'un transitoire de branchement, la tension reste proche de la tension d'avalanche (53–59 V). Si la mesure B7 montre des transitoires énergétiques, monter une SMCJ43A achetée ailleurs (même empreinte).
2. **Résistances de précharge [H]** : la RC2512 est une couche épaisse standard de 1 W, pas la série « HP » de l'exemple TI (non disponible chez JLCPCB). Chaque résistance reçoit au plus 8 W crête et 0,88 J sur l'hypothèse 2 mF. À confirmer sur la courbe d'impulsion Yageo ou remplacer par des CRCW2512…HP soudées à la main. Le firmware doit limiter les précharges répétées.
3. **5N10-MS [H]** : résistance à l'état passant spécifiée à 10 V. Avec une commande en 3,3 V (feux), vérifier sur la fiche que le seuil de 1,5 V typique laisse passer 0,5 A sans échauffement, sinon choisir un autre MOSFET 100 V à seuil logique.
4. **CSD19536KTT** : seulement 253 pièces chez LCSC. Il en faut 10 pour 5 cartes.
5. **Empreintes à contrôler au layout** : inductance SRG0630 sur l'empreinte `L_Changjiang_FXL0630`, INA237 sur `TSSOP-10_3x3mm_P0.5mm`, bouton TS-1187A, USB-C TYPE-C-31-M-12.

## Pièces à souder à la main (non commandées chez JLCPCB)

| Pièce | Référence | Cartes |
|---|---|---|
| Connecteurs pack et moteur | Amass XT60PW-M | Power Control (J2, J3) |
| Connecteurs JST XH 2, 3, 4 et 6 points | B2B/B3B/B4B/B6B-XH-A | Les deux |
| Connecteur de l'écran | JST SM13B-GHS-TB (GH 1,25 mm, 13 points) | Display (J220) |
| Encodeur | Alps EC11E vertical avec poussoir (ou Bourns PEC11R) | Display (SW210) |
| Buzzer | TMB12A05 | Power Control |
| Cavaliers de terminaison | Barrette 2,54 mm 2 points | Les deux |

Non montés : J1, J4, J5 (Power Control) et J201 (Display), barrettes de mise au point.

## Pistes pour réduire les frais de pièces Extended

- Display : 6 lignes Extended incompressibles sauf l'USBLC6 et le TPS22918.
- Power Control : 21 lignes. Les plus faciles à supprimer sont l'AP2112K et le TPS2116 (si on renonce à programmer la carte sans pack) et C11 (68 nF, remplaçable par 100 nF Basic au prix d'un délai de 1,6 ms au lieu de 1,07 ms).

## Révision du 2026-11-16 (soir) — optimisation demandée par Théo

Cette section remplace les lignes correspondantes ci-dessus. Power Control passe à **19,99 USD** de pièces assemblées, **25 lignes Basic, 17 Extended** (au lieu de 28,77 USD, 23 et 21).

| Sujet | Avant | Maintenant |
|---|---|---|
| MOSFET principaux Q1, Q2 | CSD19536KTT, TO-263, 2,4 mΩ max, 4,68 USD, 253 en stock | **HYG017N10NS1TA** (C5205219), boîtier TOLL, 100 V, 1,6 mΩ typ / 1,95 mΩ max, 1,25 USD, 5153 en stock. Pertes à 30 A : `2 × 30² × 1,95 mΩ` = 3,5 W au lieu de 4,3 W. |
| Mesure de courant | INA237 sur I²C | **Supprimée.** Le courant de décharge sort de la broche IMON du TPS48111 vers l'ADC (GPIO2). Le SOC et le courant signé viennent du DALY. |
| Tension de sortie | INA237 (VBUS) | Diviseur 150 kΩ / 10 kΩ sur `PACK_SW` vers l'ADC (GPIO7) : 42 V → 2,63 V. Sert à vérifier la précharge. Aucun courant en veille. |
| Alimentation par USB | AP2112K + TPS2116 (2 lignes Extended) | **AMS1117-3.3 + AO3401A + AO3400A + 3 résistances**, toutes Basic. |
| C11 (temporisation) | 68 nF Extended | 100 nF Basic : délai de surintensité 1,58 ms. |

### Mesure par IMON

- [C] `V_IMON = (V_SNS + V_OS) × 0,9 × R_IMON / R_SET`. Avec R48 = 5,1 kΩ et R9 = 100 Ω : gain 45,9, soit **45,9 mV/A**. 30 A → 1,38 V ; 58,7 A → 2,69 V.
- [C] Offset ±200 µV, soit ±0,2 A. Miroir de courant à ±1 %.
- R49 = 10 kΩ et C32 = 100 nF filtrent et protègent l'entrée ADC (IMON peut monter à 6,5 V sur un court-circuit).
- [H] IMON ne mesure que le sens décharge. Le courant de régénération n'est connu que par le DALY, lentement. [M] B5 : cadence et résolution du courant DALY, zone morte à faible courant.
- Les GPIO 8 et 9 de Power Control sont libérés.

### Alimentation USB simplifiée

- Le buck 3,3 V alimente directement le rail. L'AMS1117 (U9) fait le 3,3 V depuis VBUS.
- Q7 (P-MOS) relie la sortie de U9 au rail seulement quand VBUS est présent (Q8 tire sa grille à la masse). Sans USB, Q7 est bloqué et sa diode interne empêche le rail de réalimenter U9 : pas de courant de fuite en veille.
- [H] USB seul, le rail 3,3 V remonte aussi dans la sortie du buck arrêté (quelques dizaines de µA vers l'étage pack). Sans conséquence au banc ; à contrôler à la mise en route.
- USB et pack ensemble : le buck (3,35 V) domine, U9 ne débite pas.

### Réserves levées ou maintenues

- **5N10-MS : levée.** Fiche MSKSEMI : seuil 1,2–1,8 V, 125 mΩ max à 4,5 V. La commande en 3,3 V suffit pour 0,5 A.
- **Résistances de précharge : maintenue.** La fiche Yageo RC2512 ne donne pas de courbe d'impulsion ; seul l'essai de surcharge 5 s à 2,5 × la tension nominale y figure.
- **TVS SMCJ48A : maintenue**, dépend de la mesure B7.

## Révision du 2026-11-17 — sonde de température, LED d'état, points de test

Toutes les pièces ajoutées sont **Basic**, sauf le connecteur, soudé à la main. Surcoût de pièces assemblées ≈ 0,03 USD par carte, sans nouvelle ligne Extended.

| Réf. | Rôle | Pièce | LCSC | Type | Prix JLC |
|---|---|---|---|---|---:|
| R55 | Pont de la CTN | 10 kΩ 1 % 0603 (0603WAF1002T5E) | C25804 | Basic | 0,0024 |
| R56 | Série vers l'ADC | 1 kΩ 1 % 0603 (0603WAF1001T5E) | C21190 | Basic | 0,0017 |
| C34 | Filtre de la sonde | 100 nF 50 V X7R 0603 (CC0603KRX7R9BB104) | C14663 | Basic | 0,0083 |
| R57 | LED d'état | 1 kΩ 1 % 0603 | C21190 | Basic | 0,0017 |
| D4 | LED d'état | KT-0603R rouge 0603 | C2286 | Basic | 0,0077 |
| J13 | Sonde externe | JST B2B-XH-A | — | soudé main | — |
| TP1–TP25 | Points de test | pastilles cuivre | — | hors BOM | 0 |

- La fiche LCSC de C14663 affichait un stock à 0 le 2026-11-17, mais c'est la 100 nF 0603 Basic standard de JLCPCB. À recontrôler au devis ; l'alternative est CL10B104KB8NNNC (C1591, Extended).
- Les GPIO 8, 9 et 16 sont maintenant pris (sonde et LED), contrairement à la mention « libérés » de la section précédente.
- À acheter à part : une sonde CTN 10 kΩ B3950 et un boîtier XH 2 points avec deux contacts.
