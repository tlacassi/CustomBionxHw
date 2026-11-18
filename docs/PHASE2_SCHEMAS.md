# Phase 2 — modifications des schémas (2026-11-16)

> Révisions ultérieures (INA supprimé, MOSFET TOLL, USB simplifié, valeurs Basic) : voir `BOM.md`.

État : ERC à 0 sur `BionX_Power_Control`, `BionX_Display` et le projet racine. PCB toujours vides. Ce document prime sur `CHOIX_TECHNIQUES.md`, `POWER_BUDGET.md` et `DISPLAY_SCHEMA.md` tant qu'ils ne sont pas réécrits (P6.4).

Légende : **[C]** confirmé (datasheet ou netlist), **[H]** hypothèse, **[M]** à mesurer.

## Décisions prises le 2026-11-16

- **Frein et gâchette sur le Display** (décision de Théo, remplace la valeur par défaut « sur Power Control »). Conséquence : ces deux signaux passent par le CAN. Power Control doit couper l'assistance si les trames du Display cessent (timeout firmware à valider en P7.5).
- **Gâchette Hall 5 V** [H] : le 5 V est fourni par le faisceau, qui passe de 5 à **6 conducteurs**.
- **UART DALY isolé par deux optocoupleurs** : aucun courant au repos, masse BMS séparée.
- **Écran** : Waveshare 1.28inch Touch LCD (SKU 24155), GC9A01 + CST816S, connecteur GH1.25 13 points, module 39,89 × 38,50 mm.
- **Encodeur soudé sur la carte** : type Alps EC11E vertical avec poussoir (MPN à fixer en phase 3).

## Faisceau Power Control (J10) ↔ Display (J101)

| Pin | Signal | Remarque |
|---|---|---|
| 1 | GND | |
| 2 | 3,3 V permanent | Après le sélecteur U10 : l'USB de Power Control alimente aussi le Display au banc. |
| 3 | 5 V commuté | Sortie du buck 5 V (U5), présent seulement si `ACCESSORY_EN` est actif. Alimente la gâchette. |
| 4 | CANH | Même bus que le moteur (J6). |
| 5 | CANL | |
| 6 | WAKE secours | Actif bas. Display : GPIO40 via 1 kΩ. Power Control : 1 kΩ, rappel 100 kΩ, 100 nF, GPIO1 (RTC). Inutilisé si le test B4 réussit. |

## Power Control

### TPS48111 — protection de surintensité

- [C] **Défaut corrigé** : IWRN (broche 8) était relié à IMON (broche 7) et à GND, R11 court-circuitée. La surintensité était désactivée (datasheet : IWRN à GND = court-circuit seul). IMON reste à GND (inutilisé), IWRN va à GND par R11.
- [C] `R_IWRN = 11,9 × R_SET / (R_SNS × I_OC)` → R11 = **31,6 kΩ** donne `11,9 × 100 / (31 600 × 0,001) = 37,7 A`. L'offset `V_OS_SET` vaut ±200 µV (datasheet), soit ±0,5 % sur 37,7 mV : négligeable.
- [C] `R_ISCP = I_SC × R_SNS / 15,6 µA − 464` → R10 = **3,4 kΩ** donne 60,3 A (au lieu de 50 A), réponse 1,2 µs.
- [C] Verrouillage : **R38 = 100 kΩ en parallèle de C11** (section 8.3.3.2). Sans elle le circuit réessayait seul. Avec C11 = 68 nF : `t_OC = 68 nF × 100 kΩ × ln(1/(1 − 1,2/8,2)) ≈ 1,07 ms`. Réarmement par le firmware (basculer INP ou EN).
- [H] Tolérances cumulées estimées à ±10 % : 34–42 A, donc au-dessus de la pointe moteur de 30 A.

### Shunt Kelvin

- [C] Les prises de mesure de R1 ont leurs propres nets : `SHUNT_KP` (R1.2 → R9, R10, INA228 IN+) et `SHUNT_KN` (R1.3 → CS−, INA228 IN−). Le courant passe par R1.1 (`PACK_RAW`) et R1.4 (`SENSE`). Le routage Kelvin est ainsi imposé par la netlist.

### Précharge

- [C] R18 ∥ R39 = 2 × 220 Ω en boîtier 2512 tenue en impulsion, soit **110 Ω**. C'est le montage de l'exemple TI (2 × CRCW2512220RJNEGHP, 24 W crête).
- [H] Pour 0,5–2 mF à 42 V : crête 0,38 A et 16 W, énergie ≤ 1,76 J répartie sur deux résistances, τ = 55–220 ms, 99 % en 0,25–1 s.
- [M] B2 : capacité réelle et courant absorbé par le contrôleur pendant la précharge (une charge de 50 mA fait chuter 5,5 V dans 110 Ω).
- Le firmware doit borner la précharge par un timeout : une sortie en court-circuit dissipe 16 W en continu.

### Entrée pack et sortie contrôleur

- [C] LMR36510 : 65 V en fonctionnement, **70 V maximum absolu**. C'est le composant le plus fragile (INA228 85 V, TPS48111 et MOSFET 100 V).
- D2 (`PACK_RAW`) et D3 (`PACK_SW`) : TVS unidirectionnelles **SMCJ43A** (veille 43 V, écrêtage 69,4 V max à 21,6 A).
- Ces TVS couvrent les transitoires (branchement du pack, coupure sur défaut). Elles ne tiennent **pas** une régénération entretenue si le DALY coupe la charge : c'est le rôle du firmware (limite de régénération ≤ 9,3 A, alerte de tension INA228) et de la mesure B7.

### UART DALY (J12)

- U12 : TX du BMS → RX ESP (GPIO48). U13 : TX ESP (GPIO47) → RX du BMS. R42/R44 = 470 Ω (≈4,5 mA dans la LED), R43/R45 = 2,2 kΩ de rappel.
- UART au repos = niveau haut = LED éteintes : consommation nulle en veille des deux côtés.
- J12 : 1 = 3,3 V du BMS, 2 = TX BMS, 3 = RX BMS, 4 = masse BMS.
- [H] Le port DALY fournit un 3,3 V et des niveaux 3,3 V. [M] B5 : brochage, tension de la broche d'alimentation, capacité du TX à absorber 4,5 mA. Si l'alimentation du port vaut 12 V, changer R42 et R45.
- [H] Optocoupleur type EL357N : temps de coupure de quelques dizaines de µs pour un bit de 104 µs à 9600 bauds. À valider au banc.

### Autres

- **Feux en tension pack** (décision de Théo) : J8 (arrière) et J11 (avant) ont leur + sur `PACK_RAW` à travers un fusible réarmable (F1, F2 : 0,5 A / 60 V, boîtier 1812) et sont commutés côté masse par Q4 et Q6 (GPIO14 et GPIO38). Les feux fonctionnent moteur coupé.
- Q4 et Q6 doivent être des MOSFET N **100 V à seuil logique** (commande 3,3 V) : référence à fixer en phase 3. L'AO3400A (30 V) ne convient plus pour ces deux canaux ; il reste pour le buzzer (Q5).
- Le + des connecteurs de feu reste à la tension du pack feu éteint. Choisir des feux acceptant au moins 42 V (plage 12–60 V).
- **Le second buck (U5) sort maintenant 5 V** : R25 = 24,9 kΩ avec R24 = 100 kΩ donne 5,02 V ; L2 = 22 µH (table TI à 400 kHz). Le L78M05 (U6) et ses condensateurs sont supprimés. Il n'y a plus de rail 12 V.
- USB : U11 (sélecteur 5 V) et C30 retirés. Le buzzer et le faisceau prennent le 5 V du buck. U9 + U10 conservés : la carte se programme sur USB seul, sans pack sur l'établi.
- Ancien J10 (3,3 V / 12 V / GND) retiré, remplacé par le J10 6 points de la feuille CAN.

### GPIO Power Control

| Fonction | GPIO |
|---|---:|
| WAKE secours | 1 |
| CAN TX / RX / STB | 4 / 5 / 6 |
| Tension de sortie (ADC) | 7 |
| Courant IMON (ADC) | 2 |
| Sonde de température (ADC1_CH7) / alimentation du pont / LED d'état | 8 / 16 / 9 (ajout du 2026-11-17) |
| TPS48111 EN / précharge / principal | 10 / 11 / 12 |
| 5 V enable / feu arrière / buzzer | 13 / 14 / 15 |
| TPS48111 FLT_I / FLT_T | 17 / 18 |
| USB D− / D+ | 19 / 20 |
| Feu avant | 38 |
| DALY TX / RX | 47 / 48 |
| UART0 TX / RX | 43 / 44 |

## Display

- Retirés : phare (J108, Q104, R130, R131), régulateur 5 V (U121, C242), alimentation USB (U109, U110, C128), anciens J101 et J106, connecteur d'encodeur J210.
- Ajoutés : J101 6 points, R138 (1 kΩ, WAKE), SW210 (encodeur soudé), J220 13 points dans l'ordre du module Waveshare (MISO non relié).
- USB-C conservé en **données seules** (USB-Serial-JTAG natif). La carte s'alimente uniquement par le faisceau.
- Frein (J212) : contact NC vers GND, rappel R224 = 10 kΩ sur le 3,3 V commuté (0 µA en veille, 0,33 mA en service), filtre 1 kΩ / 10 nF. Fil coupé = frein actif.
- Gâchette (J221) : 5 V du faisceau, diviseur 18 kΩ / 22 kΩ, 100 nF, ADC1 sur GPIO2.
- J201 (UART de service) marqué non monté.

### GPIO Display

| Fonction | GPIO |
|---|---:|
| Frein | 1 |
| Gâchette (ADC1_CH1) | 2 |
| CAN TX / RX / STB | 4 / 5 / 6 |
| Clic encodeur (réveil) / bouton | 7 / 8 |
| Encodeur A / B | 9 / 10 |
| LCD SCK / MOSI / CS / DC / RST / BL | 11 / 12 / 13 / 14 / 15 / 16 |
| Tactile SDA / SCL / INT / RST | 17 / 18 / 21 / 38 |
| USB D− / D+ | 19 / 20 |
| Alimentation UI (TPS22918) | 39 |
| WAKE secours | 40 |

## Reste à faire avant le layout

1. Empreinte DGX19 du TPS48111 : **faite** d'après le plan TI 4226944/A (20 positions au pas de 0,5 mm, position 16 absente, pastilles 1,45 × 0,3 mm, entraxe des rangées 4,4 mm, pas de plage thermique). Fichier `pcb_power_control/BionX_Power.pretty/VSSOP-19_DGX_3x5.1mm_P0.5mm.kicad_mod`, affectée à U1.
2. Empreintes et MPN manquants : shunt, inductances, buzzer, connecteurs, boutons, optocoupleurs, TVS, résistances de précharge.
3. Q3 (précharge) est un CSD19536KTT pour 0,4 A : un petit MOSFET 100 V suffirait, éventuellement le même que Q4 et Q6.
4. Vérifier le brochage physique du connecteur GH1.25 du module d'écran (le wiki Waveshare était indisponible).
