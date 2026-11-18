# Phase 4 — PCB Power Control (2026-11-16)

État : carte routée, **DRC à 0** avec `kicad-cli pcb drc --severity-all --schematic-parity` (0 violation, 0 liaison non connectée, 0 écart de parité avec le schéma). ERC du schéma à 0. Vues : `docs/images/power_control_face_avant.png` et `power_control_face_arriere.png`.

## Caractéristiques

| Élément | Valeur |
|---|---|
| Dimensions | 112 × 82 mm, 4 trous M3 |
| Couches | 4 : F.Cu (signaux + puissance), In1 (masse), In2 (alimentations + masse), B.Cu (signaux + masse) |
| Empreintes | 117 + 4 trous de fixation |
| Pistes | 1385 segments (1020 face avant, 365 face arrière), aucune sur les couches internes |
| Vias | 872 (794 de 0,6/0,3 mm ; 78 de 0,8/0,4 mm sur le chemin de puissance) |
| Classes de nets | Défaut 0,2 mm ; Rail 0,3 mm (3,3 V, 5 V, VBUS) ; Puissance 0,6 mm, isolement 0,3 mm |

## Chemin 30 A

- En ligne droite en haut de carte : J2 (pack) → R1 (shunt) → Q1 → Q2 → J3 (moteur), les deux broches + alignées.
- Polygones pleins sur F.Cu, doublés sur In2 par des vias de couture : `PACK_RAW`, `SENSE`, `FET_SRC`, `PACK_SW`. Largeur 12 à 15 mm.
- Retour de masse par deux plans complets (In1 et B.Cu) entre les broches − de J2 et J3.
- [H] Avec du cuivre 35 µm dehors et 17 µm dedans, ces largeurs conviennent pour 30 A en pointe ; la marge en continu est faible. L'option cuivre 70 µm externe est à chiffrer en phase 6.

## Méthode

1. Carte générée par l'API Python de KiCad à partir de la netlist (identifiants de symboles conservés, la mise à jour depuis le schéma reste possible).
2. Placement par blocs fonctionnels, connecteurs en bord de carte, module ESP32 dans le coin bas-droit avec une zone sans cuivre sous l'antenne.
3. Signaux routés par Freerouting 2.5.0, limité aux deux faces externes pour garder les plans internes intacts.
4. Vias de couture de masse et échappements ajoutés par script, puis corrections manuelles.

## Points à relire avant fabrication

Le routage est automatique : il respecte les règles électriques, mais personne ne l'a relu piste par piste.

1. **Mesure du shunt** : les nets `SHUNT_KP` et `SHUNT_KN` partent bien des prises Kelvin, mais leur tracé n'est pas en paire serrée. À reprendre à la main si la mesure IMON s'avère bruitée.
2. **Boucles des deux bucks** : condensateurs d'entrée et nœuds de commutation placés près des régulateurs, tracé non optimisé.
3. **USB** : D+ et D− ne sont pas routés en paire différentielle contrôlée. Acceptable en USB pleine vitesse (12 Mbit/s) sur quelques millimètres.
4. **Via dans la pastille de R34** (masse) : seule solution trouvée pour relier cette pastille. Risque mineur de manque de soudure.
5. **Sérigraphie** : trois contrôles sont désactivés dans le projet (`silk_overlap`, `silk_over_copper`, `silk_edge_clearance`). Les références se chevauchent par endroits ; sans effet électrique, à nettoyer si vous voulez une sérigraphie lisible.
6. **Mécanique** : aucun boîtier n'a été pris en compte. Vérifier la position des connecteurs et des trous dans la vue 3D.
7. **Thermique** : les MOSFET TOLL dissipent ≈1,75 W chacun à 30 A, évacués par les polygones. Non simulé.

## Outils installés pour cette phase

- OpenJDK (Homebrew) et Freerouting 2.5.0 dans `~/Documents/KiCad/freerouting/`.

## Passe de nettoyage après relecture de Théo (2026-11-17)

Point de départ : la version relue par Théo, sauvegardée avant modification dans `../sauvegarde_pc_relu_theo_2026-11-17.kicad_pcb`. État final : **DRC à 0** (`--severity-all --schematic-parity`, sérigraphie exclue comme avant), **ERC à 0**.

### Pistes

| Défaut | Avant | Après |
|---|---:|---:|
| Changements de largeur en cours de piste | 46 | 1 |
| Angles droits hors pastilles | 32 | 5 |
| Micro-segments (< 0,05 mm) | 30 | 1 |
| Extrémités posées au milieu d'une autre piste | 7 | 0 |
| Doublons superposés, allers-retours | 15 | 0 |

- Largeurs : une seule largeur par tronçon (0,2 mm signaux, 0,3 mm alimentations et masse, 0,6 mm puissance). Les rétrécissements à 0,15 mm sont limités au pied des broches au pas de 0,5 mm (TPS48111, USB-C).
- Les 5 angles droits restants n'ont pas la place pour un chanfrein sans violer l'isolement.
- Chaque modification est vérifiée contre l'isolement ; celles qui ne passent pas sont annulées.

### Désignateurs

Les empreintes des connecteurs avaient été renommées (BATTERY, MOTOR, DISPLAY, BMS, REAR/FRONT LIGHT, BOOT, EN). Cela casse le lien avec le schéma, la BOM et le fichier de placement JLCPCB. Les désignateurs d'origine (J2, J3, J10, J12, J8, J11, SW1, SW2) sont remis sur F.Fab, et vos étiquettes sont conservées telles quelles en texte de sérigraphie. Les valeurs modifiées (« Moteur », « FEU AVANT », « Flash » sur J9) sont revenues à celles du schéma : elles alimentent la BOM.

### CAN

- Avant : CANH (159 mm) et CANL (151 mm) suivaient deux chemins séparés de 1 à 4 cm, en boucle autour des convertisseurs.
- Maintenant : **paire couplée** (0,2 mm, écart 0,2 mm) en face arrière entre J10 (faisceau Display) et J6 (moteur), sans via. Courts tronçons locaux en face avant vers R29, J7, D1 et U7.
- Pour y arriver, **J6 est tourné de 180°** (broche 1 CANH en haut) et décalé de 0,5 mm. Le brochage ne change pas, seule l'orientation du boîtier change ; à vérifier côté sortie de câble.
- À 125 kbit/s, l'adaptation d'impédance n'a pas d'effet. L'intérêt de la paire est la compatibilité électromagnétique : petite surface de boucle, couplage identique des perturbations de l'étage de puissance sur les deux fils, donc rejetées par le récepteur différentiel.
- Contrainte restante : la paire longe l'étage de puissance (y ≈ 77–79 mm) faute d'autre couloir libre. Rapprocher J10 de J6 raccourcirait fortement le bus ; c'est un choix mécanique de faisceau.

### Composants déplacés

- **R46, R47, C31** (réveil de secours) étaient à 20 mm de l'ESP32 et à 90 mm de J10. Les pistes WAKE faisaient 124 et 85 mm et coupaient le passage du CAN. Ils sont regroupés près de l'ESP32 : 63 et 29 mm.
- **C25** (découplage VIO du TCAN3413) est placé juste sous la broche 5, ce qui libère la sortie de CANL.

### Plans de masse

| Couche | Morceaux | Surface GND |
|---|---:|---:|
| In1 | 1 (plan plein) | 8246 mm² sur 9184 |
| B.Cu | 20 | 7039 mm² |
| F.Cu | 31 | 5027 mm² |
| In2 | 23 (autour des polygones de puissance) | 5068 mm² |

- **Le plan de masse de la face arrière (GND_B) avait disparu** dans la version relue. Il est recréé : les pistes de la face arrière ont une référence de masse proche, et la masse a un second chemin entre les deux XT60. À supprimer de nouveau si c'était voulu.
- Aucun morceau isolé (suppression des îlots activée), 603 vias de couture GND.
- In1 reste un plan plein sous tout le circuit logique. Sous le chemin de puissance, il est percé par les rangées de vias des polygones `PACK_*`, avec des ponts d'au moins 0,25 mm.

### Ce qui reste à relire

- La paire CAN et les nouvelles pistes WAKE ont été tracées par un routeur sur grille écrit pour l'occasion. Elles sont propres (0/45/90°, DRC à 0) mais méritent un coup d'œil, en particulier la sortie de J6 et le passage entre R29, J7 et D1.
- Étiquette « Flash » de J9 : si c'était une étiquette de sérigraphie voulue, elle est à remettre en texte.

## Points de test, sonde de température et LED d'état (2026-11-17)

État final : **ERC à 0**, **DRC à 0** avec parité schéma (`--severity-all --schematic-parity`). Le schéma et le PCB sont synchronisés.

### Points de test (P4.4)

Ce sont 25 pastilles cuivre rondes de 1,5 mm (`TestPoint_Pad_D1.5mm`). Elles ne figurent ni dans la BOM ni dans le fichier de placement, donc ne coûtent rien à l'assemblage. Chaque pastille est posée sur une piste ou dans un polygone de son net, sinon reliée par une courte piste, à au moins 2,8 mm de la plus proche pour laisser passer deux pointes de sonde. Coordonnées en mm depuis le coin haut gauche de la carte.

| TP | Net | Face | x | y | Usage |
|---|---|---|---:|---:|---|
| TP15 | PACK_RAW | avant | 21,6 | 23,6 | Tension pack, côté entrée |
| TP16 | PACK_SW | avant | 79,6 | 21,4 | Sortie contrôleur, rampe de précharge |
| TP19 | FET_SRC | avant | 53,8 | 18,8 | Sources communes de Q1/Q2 (référence de V_GS) |
| TP17 | MAIN_GATE | arrière | 56,2 | 29,9 | Grille Q1/Q2 |
| TP18 | PRE_GATE | arrière | 59,5 | 5,4 | Grille Q3 (précharge) |
| TP20 | IMON | avant | 45,5 | 38,5 | Sortie IMON avant diviseur (45,9 mV/A) |
| TP21 | TMR | avant | 37,5 | 28,5 | Temporisation de surintensité (C11) |
| TP5 | POWER_FLT_I | avant | 60,5 | 39,0 | Défaut surintensité |
| TP6 | POWER_FLT_T | arrière | 61,8 | 44,6 | Défaut thermique / temporisation |
| TP4 | IMON_ADC | arrière | 48,0 | 39,8 | Entrée ADC courant (GPIO2) |
| TP3 | VSW_ADC | avant | 78,6 | 38,5 | Entrée ADC tension de sortie (GPIO7) |
| TP1 | PWR_3V3 | arrière | 90,0 | 51,0 | Rail logique |
| TP25 | PACK_5V_AUX | avant | 42,3 | 54,3 | Buck 5 V (faisceau, buzzer) |
| TP2 | USB_VBUS | avant | 53,4 | 74,5 | Présence USB |
| TP7 | CAN_TX | arrière | 89,4 | 33,3 | GPIO4 → TCAN3413 |
| TP8 | CAN_RX | arrière | 90,4 | 37,7 | TCAN3413 → GPIO5 |
| TP23 | CANH | avant | 107,8 | 30,6 | Bus, côté J6 |
| TP24 | CANL | avant | 96,3 | 34,4 | Bus, près de U7 |
| TP11 | WAKE_SPARE | arrière | 74,9 | 66,2 | Réveil de secours (B4) |
| TP9 | DALY_TX | arrière | 32,0 | 67,0 | GPIO47 côté ESP32 |
| TP10 | DALY_RX | arrière | 21,6 | 58,8 | GPIO48 côté ESP32 (côté BMS : J12) |
| TP12 | TEMP_ADC | arrière | 108,1 | 62,0 | Entrée ADC de la sonde (GPIO8) |
| TP13, TP14, TP22 | GND | avant | 92,0 / 76,4 / 36,8 | 46,0 / 63,6 / 42,0 | Masse de sonde : près de l'ESP32, au centre, près du TPS48111 |

- **11 pastilles sur 25 sont en face arrière** : la face avant n'avait pas de place à moins de 15 mm du composant concerné. La carte se sonde à plat, retournée.
- J4 (EN, INP_G, INP, FLT_I, FLT_T) et J5 (UART0, EN, BOOT) restent des barrettes non montées : elles servent aussi de points de mesure.
- Pas de point sur les prises Kelvin du shunt : sonder directement les plots de R1. Un point de test ajouterait un tronçon à la mesure du TPS48111.

### Sonde de température (J13)

- **J13** : JST XH 2 points vertical (B2B-XH-A, soudé main comme les autres XH), au bord droit au-dessus de l'ESP32. Broche 1 = signal, broche 2 = masse.
- **Montage :**
  - le pont est R55 (10 kΩ 1 %) entre `TEMP_BIAS` (GPIO16) et `NTC_NODE`, avec la CTN de J13 vers la masse ;
  - C34 (100 nF) est sur `NTC_NODE`, au plus près du connecteur ;
  - R56 (1 kΩ) va vers `TEMP_ADC` (GPIO8, ADC1_CH7) et limite le courant injecté par le câble externe (ESD).
- **Pas de courant en veille** : le pont n'est alimenté que pendant la mesure. Avec une alimentation sur le 3,3 V permanent, il consommerait 165 µA à 25 °C.
- **Firmware :**
  - passer GPIO16 à l'état haut ;
  - attendre au moins 5 ms (constante de temps ≈ 0,5 ms) ;
  - lire GPIO8, puis repasser GPIO16 à l'état bas ;
  - interpréter les extrêmes : sonde débranchée, lecture pleine échelle ; sonde en court-circuit, lecture ≈ 0.
- [H] **Sonde** : CTN 10 kΩ à 25 °C, B = 3950. [M] Relever R25 et B de la sonde achetée avant de coder la table.

### LED d'état (D4)

- D4 : LED rouge 0603 KT-0603R (C2286, Basic), avec R57 = 1 kΩ, sur **GPIO9**, active à l'état haut. Courant ≈ 1,3 mA.
- Elle est placée juste au-dessus de la rangée haute de l'ESP32.
- Les LED vertes 0603 en Basic ont une tension de seuil d'environ 3 V, trop proche du 3,3 V. Une verte Extended coûterait des frais de ligne.

### Modifications du routage existant

- **R27** (rappel BOOT) remonte de 2,7 mm et tourne de 180° pour laisser la place à J13. Ses deux pistes sont refaites à 45°.
- **Le 3,3 V de la broche 2 de l'ESP32** ne descend plus le long du bord droit en face avant. Il passe en face arrière (0,3 mm, x = 171 mm) entre deux vias. La bande libérée reçoit R55, R56, C34 et leurs pistes.
- 17 vias de couture GND qui gênaient sont retirés (603 → 586). Aucun via des polygones de puissance n'est touché : 36 FET_SRC, 24 PACK_RAW, 38 PACK_SW, 12 SENSE, comme avant.
- L'audit des pistes est revenu au niveau d'avant ces ajouts : 5 angles droits, 7 segments hors 45°, 1 changement de largeur, tous antérieurs.

### Affectation des GPIO libres

Les GPIO 8, 9 et 16 sont maintenant utilisés. Il reste **GPIO3** (ADC1, broche de configuration JTAG, utilisable en entrée), GPIO21, GPIO35 à 37, GPIO39 à 42 et GPIO45/46 (configuration).

## Plans internes et rapprochement du CAN (2026-11-18)

État final : **DRC à 0** avec parité schéma ; le schéma n'est pas modifié.

### Plan PACK_RAW de la couche interne 2 réduit

- **Constat.** La zone `PACK_RAW_In2` couvrait tout le quart gauche de In2 (x 1–29, y 9–74 mm depuis le coin haut gauche), et une seconde zone `PACK_RAW_In2_bucks` couvrait le dessous des convertisseurs. Les pistes de la face arrière prennent In2 comme plan de référence : 44 mm de chaque fil CAN, ainsi que ACCESSORY_EN, DALY_RX, les retours de tension des convertisseurs, etc., s'appuyaient donc sur le plan du pack au lieu de la masse.
- **Ce qui a changé.**
  - `PACK_RAW_In2` est réduite au chemin 30 A, alignée sur la zone de la face avant : x 14–29, y 9–25.
  - `PACK_RAW_In2_bucks` est supprimée.
  - La distribution vers les feux (F1/F2), la TVS D2 et les convertisseurs était déjà entièrement routée en pistes de 0,6 mm. Ces zones n'en étaient qu'une doublure.
- **Résultat.**
  - Plus aucun signal de la face arrière n'a un plan de puissance comme référence, hormis ceux de l'étage de puissance lui-même : MAIN_GATE au-dessus de FET_SRC, prises du shunt au-dessus de SENSE.
  - La masse de In2 passe de 23 à 17 morceaux.

### J10 (faisceau Display) déplacé près du transceiver

- **Avant.** J10 était en bas à gauche. La paire CAN traversait toute la carte (147 et 150 mm en face arrière) jusqu'à J6.
- **Maintenant.** J10 est **au bord supérieur, entre les trous de fixation H4 et H1**, tourné de 180°, broche 1 (GND) à droite, à (100,5 ; 4,3) mm. C'était le seul emplacement libre pour un 6 points près de U7. Le brochage du faisceau ne change pas.
- **Paire CAN** : en face arrière, sans via, de J10 directement à J6, par la bande de masse à droite du XT60 moteur (sous son boîtier, x ≈ 108 mm). Elle a la masse comme référence sur toute sa longueur.
  - Longueur totale du réseau CAN, tronçons locaux U7/D1/R29/J7 compris : **72 mm (CANH) et 75 mm (CANL)**, contre 173 et 164 mm auparavant.
  - La paire est routée J10 → J6 → U7 : le transceiver est sur un court tronçon de dérivation, comme sur un bus.
- **3,3 V** : il longe le bord supérieur en face avant, puis le bord droit en face arrière (x = 111 mm), et rejoint la ligne 3,3 V existante qui alimente l'ESP32.
- **5 V et WAKE** :
  - ils descendent en face arrière à côté de la paire CAN ;
  - ils repartent vers l'ouest par l'ancien couloir du CAN (y ≈ 29 mm), au-dessus de la masse ;
  - 5 V : 2 vias, environ 70 mm depuis J10 jusqu'au buzzer ;
  - **WAKE** : 6 vias et un détour par l'ouest autour du buzzer. C'est une ligne de secours statique (RC 1 kΩ / 100 nF), sans enjeu CEM ; elle reste à reprendre à la main si on veut l'embellir.
- 80 vias de couture GND qui gênaient sont retirés dans la bande supérieure et le couloir de droite. 7 vias sont ajoutés pour relier des morceaux de masse que les nouvelles pistes avaient isolés. Total : 505.
- **[M] Mécanique** : la sortie de câble du faisceau Display est maintenant au bord supérieur, à côté du XT60 moteur. À vérifier avec le boîtier.
