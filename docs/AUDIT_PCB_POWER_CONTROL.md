# Audit du PCB Power Control — 2026-11-18

Audit critique avant commande JLCPCB, fait en lecture seule sur `pcb_power_control/` (aucun fichier du projet modifié). Ce document liste tous les points relevés et les corrections proposées.

**Conventions**
- Coordonnées **abs** = coordonnées KiCad en mm. **rel** = abs − (60 ; 50), c'est-à-dire depuis le coin haut gauche de la carte.
- Statut : **[C]** confirmé (commande, netlist, datasheet), **[H]** hypothèse, **[M]** à mesurer.
- Gravité : **Bloquant** (à corriger avant commande), **Important**, **Mineur**, **Suggestion**.

## Suivi des corrections (commencé le 2026-11-26)

Statuts : **Fait** (corrigé dans le schéma et/ou le PCB, vérifié par ERC, DRC et parité), **Partiel**, **À faire**, **Banc** (ne se règle qu'avec une mesure), **Firmware**, **Devis** (se règle à la commande JLC). Le détail de chaque correction est dans la section « Journal des corrections » en fin de document.

| Point | Gravité | Sujet | Statut |
|---:|---|---|---|
| 1 | Bloquant | Repères + / − des XT60 | Partiel : sérigraphie corrigée ; polarité moulée du XT60 à vérifier sur la pièce |
| 2 | Bloquant | Fusibles des feux | Fait |
| 3 | Important | Découplage ESP32, RC d'EN | Partiel : C35 et C36 ajoutés ; RC d'EN laissé en place (pas de place près de la broche 3) |
| 4 | Important | Boucles des bucks | Partiel : bootstrap, VCC et vias thermiques faits ; C12 (entrée de U5) laissé en place |
| 5 | Important | Kelvin du shunt, C_ISCP | Fait (boucle ≈13 mm² au lieu de ≈153) |
| 6 | Important | Rayons thermiques GND des XT60 | Fait |
| 7 | Important | Condensateur tampon PACK_RAW | Fait (C37) ; marge des TVS à confirmer au banc |
| 8 | Important | Isolement BMS / ESP32 | Fait |
| 9 | Important | TVS CAN 24 V | Fait : D1 non monté (empreinte gardée) |
| 10 | Important | Trous de fixation, CAN sous H1 | Fait |
| 11 | Important | Protection thermique matérielle (DIODE) | Fait |
| 12 | Mineur | Vias « buried » | Fait |
| 13 | Mineur | Empilement non défini | Fait (2 oz extérieur : choix au devis) |
| 14 | Mineur | AMS1117 et céramique | Fait (C28 tantale 10 µF) |
| 15 | Mineur | Classe de nets HV | Fait (règle à 0,3 mm, voir journal) |
| 16 | Mineur | Bootstrap et grilles du TPS48111 | Partiel : R61 (0 Ω) sur la grille de Q1 ; C10 laissé en place (pas de place près de U1) |
| 17 | Mineur | Sérigraphie | Partiel : textes ajoutés, traits à 0,15 mm, références déplacées ; hauteur laissée à 0,8 mm |
| 18 | Mineur | UART DALY : vitesse, veille | Banc / Firmware |
| 19 | Mineur | Retour du 3,3 V en USB seul | Banc |
| 20 | Mineur | Annotation, nom de feuille | Fait |
| 21 | Mineur | Divers (H1/J10, prix, cavalier J7) | Fait (documentation : `BOM.md`, rondelle DIN 433 en H1) |
| 22 à 28 | Suggestion | Diode série bucks, surtension VSW, faisceau, terminaison, mécanique, thermique Q1, chien de garde | 25, 26 (trou H5 M2,5) et 27 faits ; 22, 23 et 28 écartés (voir lot 9) ; 24 en attente d'une mesure [M] |
| 29 | Important | Empreinte du shunt | Fait ; résistance effective à mesurer au banc |
| 30 | Important | Pastille thermique LMR36510 | Fait |
| 31 | Mineur | Vias dans les pastilles | Fait, sauf R46.2 (via en bord de pastille, zone saturée, via bouché) |
| 32 | Mineur | Notes du schéma périmées | Fait (note des feux à finir avec le point 2) |
| 33 | Mineur | BOM et placement JLC (BZ1) | Fait |
| 34 | Mineur | Modèles 3D | À faire : téléchargement des STEP à valider |
| 35 | Suggestion | Diviseur de retour du buck 3,3 V | Écarté (gain ≈ 5 µA, voir lot 9) |
| 36 | Mineur | Impulsions des GPIO au démarrage | Fait : R58 / R59 sur FLT, filtre RC R60 / C40 sur STB (brochage du firmware inchangé) |
| 37 | Important | Court-circuit franc en sortie | Fait (C37) ; à valider au banc avec alimentation limitée |
| 38 | Mineur | Mesures ADC | Firmware |
| 39 | Mineur | Job Gerber, contour | Fait |
| 40, 41 | Suggestion | Détection de coupure, GPIO libres | 40 : couvert par VSW_ADC + C37 (firmware) ; 41 : fait (JTAG sur TP31 à TP34) |
| 42 | Mineur | Empreinte SOP-4 des EL357N | Fait |
| 43 | Mineur | Cuivre de l'AMS1117 | Fait |
| 44 | Suggestion | Points de test | Fait (TP26 à TP29) |
| 45 | Mineur | Composants au ras du bord | Devis |

## Base vérifiée

| Contrôle | Résultat |
|---|---|
| `sch erc` | 0 erreur, 0 avertissement |
| `pcb drc --severity-all --schematic-parity` | 0 violation, 0 élément non connecté, 0 écart de parité |
| DRC avec les 8 contrôles ignorés réactivés | 45 avertissements : 18 chevauchements de sérigraphie, 13 sérigraphies sur pastille, 6 sérigraphies coupées par le bord, 8 pistes non centrées sur un via. Aucun courtyard manquant, aucun type d'empreinte incohérent. |
| BOM exportée vs `docs/BOM_BionX_Power_Control.csv` | Identique (seul le regroupement des références diffère). Toutes les pièces assemblées ont MPN et LCSC. |

**Brochages et empreintes vérifiés contre les datasheets** : brochages conformes sauf le XT60 (point 1) ; deux empreintes aux dimensions non conformes, le shunt (point 29) et les LMR36510 (point 30).

| Pièce | Source | Résultat |
|---|---|---|
| TPS48111 DGX19 (empreinte maison) | Table 5-1 p. 5 ; motif TI 4226944/A | Conforme : pastilles 1,45 × 0,3 mm, pas 0,5 mm, rangées à 4,4 mm, broche 16 absente |
| HYG017N10NS1TA (TOLL) | Fiche HUAYI p. 1 et p. 7 | Conforme : 1 = grille, 2–8 = source, languette = drain ; pas 1,2 / 1,225 mm compatible avec PG-HSOF-8-1 |
| 5N10-MS | Fiche MSKSEMI | 1 = G, 2 = S, 3 = D, conforme |
| AMS1117-3.3 | Fiche AMS | 1 = GND, 2 = VOUT (languette), 3 = VIN, conforme |
| EL357N | Fiche Everlight | 1 = anode, 2 = cathode, 3 = émetteur, 4 = collecteur ; les deux voies sont non inverseuses |
| TCAN3413 | Fiche TI | 1 TXD, 2 GND, 3 VCC, 4 RXD, 5 VIO, 6 CANL, 7 CANH, 8 STB, conforme |
| LMR36510 | Table 5-1 p. 4 | 1 PGND, 2 VIN, 3 EN, 4 PG, 5 FB, 6 VCC, 7 BOOT, 8 SW, PAD, conforme |
| ESP32-S3-WROOM-1 | Fiche Espressif | 1 GND, 2 3V3, 3 EN, 13/14 USB D−/D+, 24/25 IO47/48, conforme |
| USBLC6-2SC6, PESD1CAN | Fiches ST, BORN | Conformes |
| SRG0630-220-M sur `L_Changjiang_FXL0630` | Fiche JIERR | Pastilles 8,4 / 3,7 / 3,5 mm conformes ; Isat 3,2 A |
| D2, D3 (TVS), D4 (LED) | Netlist + empreintes | Cathode sur la pastille 1, conforme |
| Shunt R1 (Kelvin) | Netlist + empreinte | Polarité conforme (KP côté PACK_RAW, KN côté SENSE) ; **dimensions non conformes : voir point 29** |
| LMR36510 (pastille thermique) | Dessin TI DDA0008B | **Non conforme : voir point 30** |
| USB-C TYPE-C-31-M-12 | Empreinte KiCad (fiche en image) | CC1/CC2 à 5,1 kΩ, D+/D− sur les deux rangées ; non vérifié contre la fiche |
| XT60PW-M | Empreinte KiCad de série | **Non conforme : voir point 1** |

---

## Bloquant

### 1. Repères « + / − » des XT60 inversés par rapport aux nets
- **Catégorie** : puissance, sérigraphie.
- **Où** :
  - J2 : broche 1 = PACK_RAW en (79 ; 67,4) abs, broche 2 = GND en (79 ; 60,2) abs. Le « + » est en (81,5 ; 57,9), à 3,4 mm de la broche GND ; le « − » est en (81,5 ; 70,2), à côté de PACK_RAW.
  - J3 : broche 1 = PACK_SW en (153 ; 67,4), broche 2 = GND en (153 ; 74,6). Le « − » est à côté de PACK_SW, le « + » à côté de GND.
- **Constat** : l'empreinte de série `AMASS_XT60PW-M_1x02_P7.20mm_Horizontal` marque la pastille 1 « − » en (−2,8 ; 2,5) et la pastille 2 « + » en (9,5 ; 2,5). Le schéma câble l'inverse. La fiche Amass (LCSC C98732) ne donne pas la polarité.
- **Statut** : [C] incohérence sérigraphie / nets ; [M] marquage moulé du boîtier réel.
- **Risque** : une fiche câblée selon la carte ou selon le boîtier inverse la polarité.
  - Côté pack : D2 court-circuite le pack, le fusible de 30 A fond, les LMR36510 sont détruits (cas de R1).
  - Côté moteur : le contrôleur est alimenté à l'envers.
  - Le détrompage du XT60 ne protège que si les fiches sont câblées comme le PCB.
- **Correction** :
  1. Avec un XT60PW-M en main, figer la convention (repérer le « + » moulé).
  2. Soit supprimer les textes « + / − » de l'empreinte et poser des repères « +42 V » et « GND » sur les bonnes pastilles (coût nul).
  3. Soit permuter les broches 1 et 2 de J2 et J3 dans le schéma (reprise des polygones de puissance, environ 7,2 mm de décalage).
  4. Câbler les fiches pack et moteur en conséquence, et l'écrire dans la doc.

### 2. Fusibles réarmables F1 et F2 utilisés au-delà de leur courant de coupure
- **Catégorie** : protection, sécurité.
- **Où** : F1 (75 ; 86) abs et F2 (75 ; 98) abs, sur PACK_RAW non commuté, vers J8 et J11 qui sortent de la carte.
- **Constat** : fiche C462518, SMD1812P050TF/60 :
  - Vmax = 60 V ;
  - **Imax = 10 A** (contre 100 A pour les versions 15 V et 30 V) ;
  - Rmin = 0,15 Ω.

  Un court-circuit franc du fil de feu à 42 V donne `42 / (0,15 + câblage)`, soit plusieurs dizaines d'ampères.
- **Statut** : [C] valeurs nominales ; [M] seuil de court-circuit du DALY et courant des feux.
- **Risque** : fil de feu pincé sur le cadre, fusible qui se fissure ou brûle. Les sorties J8 et J11 sont sous tension pack en permanence.
- **Correction** (au choix) :
  - résistance série 4,7 Ω en 2512 avant chaque fusible : Icc ≤ 42 / 4,85 ≈ 8,7 A ; chute de 0,5 à 0,8 V et 0,05 à 0,14 W pour 0,1 à 0,17 A (courant des feux à mesurer) ;
  - ou fusible CMS 1 A, ≥ 63 V DC, pouvoir de coupure ≥ 50 A, à la place du fusible réarmable (référence LCSC à choisir).
- **Coût** : 2 pièces, empreintes à adapter.

---

## Important

### 3. Pas de découplage au pied du 3,3 V de l'ESP32 ; RC d'EN éloigné
- **Catégorie** : alimentation.
- **Où** : broche 2 de U4 (3V3) en (167,25 ; 123,09) abs ; broche 3 (EN) en (167,25 ; 121,82) abs.
- **Constat** :
  - C22 (10 µF) et C21 (100 nF), les plus proches, sont en (150–153 ; 98,8), à 28–30 mm.
  - L'alimentation arrive par une seule piste de 0,3 mm en B.Cu à x = 171 (1 mm du bord), de y 99 à 121,9, avec un via en (167,6 ; 95,6) et un en (169,8 ; 123,09).
  - Le schéma des périphériques de la fiche ESP32-S3-WROOM-1 (§6) prévoit 22 µF + 0,1 µF au pied du module.
  - C23 (EN, 1 µF) est en (157,5 ; 98,8), à 26 mm de la broche EN. Le net MCU_EN fait 93 mm au total (3 vias).
- **Statut** : [C] routage ; [H] effet.
- **Risque** : reset par chute de tension pendant une rafale Wi-Fi ou BLE (environ 350 mA), bruit ADC, resets parasites sur EN.
- **Correction** :
  - 10 µF 0805 (C15850, déjà en BOM) + 100 nF 0603 (C14663) entre x 168 et 171,5, y 118 et 124 abs, hors zone d'antenne (qui commence à y 125,85), chacun avec son via GND ;
  - élargir l'arrivée du 3,3 V à 0,5 mm ou la doubler en F.Cu ;
  - déplacer C23 (et R26) contre la broche 3.
- **Coût** : 2 pièces Basic, placement.

### 4. Boucles des deux bucks
- **Catégorie** : CEM.
- **Constat** :
  - **Condensateur d'entrée de U5** : C12 (2,2 µF) est en (115 ; 116,5) abs, à environ 17 mm de VIN (95,5 ; 113,4), relié par B.Cu. Seul C13 (100 nF) est proche, à 6 mm. TI demande 2,2 µF + 220 nF collés à VIN et PGND (LMR36510 §8.5.1, p. 26). U3 est acceptable : C2 à environ 3,5 mm, C1 à 6 mm.
  - **Condensateurs de bootstrap** : C4 (114 ; 95,5) et C15 (114 ; 112,5) sont à 13,5–14 mm des broches BOOT et SW. Le nœud SW passe donc par 2 vias et 10,4 mm (U3) ou 4,4 mm (U5) de B.Cu.
  - **Voisinage du nœud SW** : IMON_ADC et U5-FB passent à moins de 1,5 mm du nœud SW en B.Cu.
  - **Pastille thermique** : aucun via sous la pastille 9 de U3 et U5. Le via FB de U3 est sous le boîtier en (97,67 ; 102,02).
- **Statut** : [C].
- **Risque** : surtensions de commutation sur VIN alors que la marge est faible (70 V max) ; rayonnement près de l'entrée ADC du courant ; échauffement inutile.
- **Correction** :
  - C12 à gauche de U5, contre les broches 1 et 2, comme C2 pour U3 ; le 100 nF au plus près ;
  - C4 et C15 contre les broches 7 et 8, sans via sur SW ;
  - 4 à 6 vias de 0,3 mm dans chaque pastille thermique vers In1 ;
  - éloigner IMON_ADC des nœuds SW.
- **Coût** : placement et routage seulement.

### 5. Mesure Kelvin du shunt
- **Catégorie** : signaux, protection. Aggrave le point 1 de `PCB_POWER_CONTROL.md` (« Points à relire »).
- **Constat** :
  - SHUNT_KP (49 mm en F.Cu) part de R1.2 à x = 87,9, descend à y = 73 et traverse en leur milieu les polygones SENSE et FET_SRC de x 98 à 120, ce qui les coupe en deux morceaux chacun.
  - SHUNT_KN (31 mm) passe en B.Cu par deux vias.
  - La boucle KP / CS+ / KN fait **≈ 153 mm²**, juste à côté du chemin de 30 A.
  - R9 (RSET) et R10 (RISCP) sont en (127–131 ; 80,5), à 12–15 mm des broches 18 et 19 de U1.
  - Pas de condensateur de 1 nF entre ISCP et CS− (recommandé par TI, §9.2.2.4 et §9.5).
- **Statut** : [C] routage ; [M] effet.
- **Risque** : déclenchements intempestifs de la protection court-circuit (58,7 A, réponse en 1,2 µs), bruit et offset sur IMON.
- **Correction** :
  - KP et KN en paire serrée (0,2 / 0,2 mm) sur une seule couche, de R1 jusqu'à U1, hors des polygones de puissance ;
  - R9 et R10 contre les broches 18 et 19 ;
  - un condensateur 1 nF 0603 entre ISCP et CS−, près de U1 (empreinte, montage au choix).
- **Coût** : 1 condensateur, reroutage d'environ 50 mm.

### 6. Broches GND des XT60 reliées par des rayons thermiques
- **Catégorie** : puissance.
- **Où** : J2.2 (79 ; 60,2) et J3.2 (153 ; 74,6) abs.
- **Constat** : les zones GND sont en « relief thermique pour les traversants ». Largeur de cuivre qui traverse l'anneau de dégagement :

  | Broche | F.Cu | In1 | In2 | B.Cu |
  |---|---:|---:|---:|---:|
  | J2.2 GND | 1,1 mm | 2,2 mm | 1,1 mm | 2,2 mm |
  | J3.2 GND | 1,1 mm | 2,2 mm | **0** | 2,2 mm |
  | J2.1 PACK_RAW (connexion pleine) | 13,5 mm | — | 13,5 mm | — |

  À 30 A, environ 190 A/mm² dans les rayons extérieurs.
- **Statut** : [C].
- **Risque** : point chaud sur la broche de retour, échauffement inégal des deux contacts du XT60. Peu de pertes, les rayons étant courts (≈ 0,03 mΩ).
- **Correction** : forcer une connexion pleine sur ces deux pastilles (au moins sur In1 et B.Cu), ou 4 rayons de 1,5 mm ; reconnecter In2 sur J3.2. Prévoir un fer puissant ou un préchauffage au soudage.
- **Coût** : nul.

### 7. Marge des TVS face au LMR36510, et pas de capacité tampon sur PACK_RAW
- **Catégorie** : protection. Aggrave B7 et la réserve 1 de `BOM.md`.
- **Constat** :
  - SMCJ48A : avalanche 53,3–58,9 V, écrêtage max 77,4 V à 19,4 A (fiche C19077611). L'écrêtage dépasse les **70 V absolus** du LMR36510 (§6.1 p. 5) vers 11 A environ.
  - PACK_RAW n'a que des céramiques : C1, C12 (2,2 µF / 100 V), C2, C13 (100 nF), soit environ 2,5 µF effectifs à 42 V.
  - Coupure sur surintensité ou court-circuit avec 1 à 3 µH de câble pack : ½·L·I² = 1,7 à 5,2 mJ à 58,7 A, soit **56 à 78 V** sur le rail avant ou pendant la conduction de la TVS.
  - Branchement à chaud sur céramiques seules : dépassement possible jusqu'à 2 × 42 V, limité par la TVS.
  - TI recommande 20 à 100 µF électrolytique en parallèle quand le câble d'entrée est long (LMR36510 §8.2).
- **Statut** : [H] calcul ; [M] inductance réelle du câble et forme d'onde au branchement (TP15).
- **Aggravation (passe 3, point 37)** : sur un court-circuit franc en sortie, la pointe d'entrée peut dépasser 100 V sans capacité tampon. Ce condensateur devient fortement recommandé.
- **Correction** :
  - 47 à 100 µF / 63 V aluminium sur PACK_RAW près de U3 et U5 (CMS 10 × 10 ou radial soudé main). Avec 100 µF, 5,2 mJ ne font monter le rail que de 1,3 V. Étincelle au branchement du XT60 : environ 88 mJ ;
  - option combinée avec le point 22 : diode série + TVS locale pour les bucks.
- **Coût** : 1 à 3 pièces.

### 8. L'isolement BMS / ESP32 n'existe pas dans le cuivre
- **Catégorie** : isolement.
- **Où** : zone BMS de (72,8 ; 108,5) à (96,3 ; 129,2) abs : J12, U12, U13, R42 à R45.
- **Constat** :
  - Les nets côté BMS (BMS_3V3, BMS_TX, BMS_RX, BMS_GND, anodes de U12 et U13) sont entourés par la GND de la carte sur les 4 couches : 368 mm² en F.Cu, 457 en In1 et In2, 383 en B.Cu dans cette zone.
  - Isolement minimal 0,18 mm (B.Cu, près de (80,8 ; 110,0)), 0,2 mm en F.Cu, 0,25 mm en In1 et In2.
  - Les plans passent sous les EL357N : l'isolement de 3750 Vrms du composant est court-circuité par 0,2 mm de carte.
  - B− (masse du DALY) et P− (masse de la carte) s'écartent jusqu'à la tension pack quand le DALY ouvre ses MOSFET.
- **Statut** : [C].
- **Risque** : tient électriquement sous vernis (IPC-2221B B4 : 0,13 mm pour 31–50 V), sans marge pour humidité ou crasse (B2 sans vernis : 0,6 mm). Couplage capacitif inutile, et l'intention « isolé » n'est pas tenue.
- **Correction** :
  - bande sans cuivre ≥ 1 mm sur toutes les couches sous U12 et U13, entre les broches 1-2 et 3-4 ;
  - côté BMS, pas de plan de la carte : soit rien, soit un îlot BMS_GND ;
  - classe de nets BMS avec 0,6 mm d'isolement.
- **Coût** : routage seulement.

### 9. TVS CAN PESD1CAN limitée à 24 V
- **Catégorie** : ESD, CAN.
- **Où** : D1 (158 ; 81,5) abs.
- **Constat** :
  - PESD1CAN : tension de veille 24 V, avalanche 26,7–31 V (fiche C343993 p. 2).
  - Le TCAN3413 tient ±58 V de défaut sur le bus et ±10 kV IEC 61000-4-2 sur CANH/CANL (fiche TI).
  - Le câble moteur porte 42 V à côté de CANH et CANL.
  - D1 est à environ 28 mm de J10 par la piste (10 mm de J6).
- **Statut** : [C] valeurs ; [H] scénario de défaut.
- **Risque** : un câble moteur écrasé met CANH ou CANL au pack, D1 conduit et brûle (souvent en court-circuit), le bus est mort. Sans D1, le TCAN3413 survivrait.
- **Correction** : TVS CAN bidirectionnelle avec une tension de veille ≥ 42–48 V, ou D1 non montée en s'appuyant sur la robustesse du TCAN3413.
- **Coût** : 0 ou 1 changement de pièce.

### 10. Trous de fixation : GND sous les têtes de vis, CAN sous H1
- **Catégorie** : mécanique, sécurité.
- **Constat** :
  - Le cuivre GND arrive à environ 0,2 mm des quatre trous M3 non métallisés, en F.Cu et B.Cu.
  - CANH et CANL passent en B.Cu à 2,5 et 2,85 mm du centre de H1 (167 ; 55) abs, sous l'entretoise (Ø 5,5) ou la rondelle (Ø 7).
- **Statut** : [C] géométrie ; [H] conséquence.
- **Risque** :
  - une vis ou une entretoise métallique relie P− au boîtier ou au cadre. Si B− touche aussi le cadre, le DALY, qui coupe côté négatif, est court-circuité ;
  - usure du vernis sous l'entretoise : CANH ou CANL à la masse.
- **Correction** : choisir un mode de fixation :
  - montage isolé : pas de cuivre dans un rayon de 3,5 mm autour des trous en F.Cu et B.Cu, entretoises plastique ;
  - ou masse franche : trous métallisés reliés à GND, en connaissance de cause.

  Dans les deux cas, éloigner la paire CAN à plus de 4 mm de H1.
- **Coût** : nul.

### 11. Pas de protection thermique matérielle des MOSFET
- **Catégorie** : ce qui manque.
- **Constat** : la broche DIODE (U1.10) est à la GND, donc FLT_T ne s'active jamais. GPIO18, R21 et TP6 sont pourtant câblés pour ce signal. La datasheet TPS4811 (§6.5) donne un arrêt à 150 °C avec un MMBT3904 ; layout §9.6.1.
- **Statut** : [C].
- **Risque** : la protection thermique repose sur le firmware et la sonde externe J13, dont l'emplacement n'est pas décidé (R5).
- **Correction** : MMBT3904 (Basic) collé au drain de Q1 (le plus chaud, à côté du shunt), base et collecteur sur DIODE, émetteur en Kelvin vers la GND de U1, plus 1 nF en parallèle.
- **Coût** : 2 pièces Basic.

---

## Mineur

### 12. 207 vias déclarés « buried » de F.Cu à B.Cu
- [C] Le fichier de perçage exporté les met bien dans le PTH (vérifié). Une sortie IPC-2581 ou ODB++, ou le contrôle DFM, pourrait les annoncer comme vias borgnes ou enterrés.
- **Correction** : les convertir en vias traversants (sélection, propriétés). Coût nul.

### 13. Empilement non défini dans le fichier
- [C] Pas de section `(stackup)` dans `BionX_Power_Control.kicad_pcb`.
- Sections de cuivre du chemin de 30 A, en F.Cu : PACK_RAW ≥ 11,05 mm, SENSE ≥ 17,6, FET_SRC ≥ 9,9, PACK_SW ≥ 10,0 mm. Les copies In2 font 12,6 à 19,5 mm.
- Échauffement estimé (IPC-2221, piste longue, donc pessimiste), section minimale de 9,9 mm à 30 A :

  | Cuivre extérieur / intérieur | F.Cu seul | F.Cu avec partage In2 |
  |---|---:|---:|
  | 35 µm / 17,5 µm | ≈ 72 °C | ≈ 18 °C (In2 : ≈ 78 °C selon IPC-2221, valeur très pessimiste) |
  | 70 µm / 17,5 µm | ≈ 23 °C | ≈ 10 °C |

- Pertes cuivre du chemin (≈ 43 mm, ≈ 11 mm de large) : ≈ 1,9 mΩ, soit ≈ 1,7 W à 30 A en 35 µm, la moitié en 70 µm.
- Vias F.Cu ↔ In2 suffisants : FET_SRC 36 × Ø 0,3 (≈ 19 mm de piste équivalente), PACK_SW 38 × Ø 0,4 (≈ 27 mm), PACK_RAW 24 × Ø 0,4 (≈ 17 mm).
- Confirme le [H] de la doc : correct pour les pointes de 30 A, marge faible en continu.
- **Correction** : préciser à la commande cuivre extérieur ≥ 1 oz (2 oz si le surcoût est faible), cuivre intérieur 0,5 oz par défaut chez JLC.

### 14. AMS1117 avec une céramique en sortie
- [H] La datasheet AMS1117 (§Stability) exige 22 µF tantale pour la stabilité. C28 est une céramique X5R de 22 µF.
- [M] Vérifier USB_3V3 à l'oscilloscope en USB seul.
- **Correction** : tantale 22 µF en 1206, ou régulateur stable sur céramique.

### 15. Classes de nets haute tension
- [C] FET_SRC est dans la classe « Rail » (isolement 0,2 mm) bien qu'il porte 30 A au potentiel du pack. MAIN_GATE, PRE_GATE, BST (pack + 12 V), PU, SHUNT_KP/KN, CS+, ISCP, VS_FILTERED et Q3-D sont en classe par défaut (0,2 mm).
- Pastilles exposées : écart minimal HV ↔ BT de 0,64 mm, hors la pastille thermique des HSOP-8 (0,35 mm, propre au boîtier).
- **Correction** : classe « HV » à 0,4–0,5 mm d'isolement pour ces nets.

### 16. Commande de grille et bootstrap du TPS48111
- [C] C10 (BST) est en (123,5 ; 83), à environ 7 mm de U1.12 ; R7 (PU) est à 12 mm. MAIN_GATE fait 47,6 mm avec 5 vias. Pas de résistance de grille individuelle par MOSFET (TI §9.6.1).
- Risque faible : les commutations sont rares et l'extinction passe par PD, direct.
- **Correction** : C10 et R7 contre U1 ; empreintes 0 Ω en série sur chaque grille.

### 17. Sérigraphie
- [C] Toutes les inscriptions font 0,8 mm de haut et 0,12 mm de trait (148 textes). JLC conseille au moins 1,0 / 0,15 mm.
- Il manque :
  - nom de la carte, version, date ;
  - avertissement « 42 V » ;
  - « + » de J8 et J11 (tension pack permanente) ;
  - brochage de J6, J12 (3V3 TX RX GND), J13, J10 ;
  - fonction de J7 (terminaison).
- **Correction** : 1,0 / 0,15 mm minimum, ajouter ces textes, nettoyer les 18 chevauchements.

### 18. UART du DALY : vitesse et états de veille
- [M] EL357N : temps de montée et descente de 18 µs spécifiés avec une charge de 100 Ω. Avec 2,2 kΩ de rappel, la descente est plus lente : à mesurer à 9600 bauds (déjà noté dans la doc).
- [C] (corrigé en passe 3) Au reset et en veille sans maintien, GPIO47 est une entrée sans rappel (fiche ESP32-S3, tableau 2-1) : la LED de U13 reste éteinte. Le seul cas à éviter est un firmware qui laisse GPIO47 en sortie à l'état bas pendant la veille (environ 4,5 mA dans U13).
- [M] Si le TX du DALY reste bas quand le DALY dort : 1,5 mA perdus dans R43 sur le 3,3 V permanent, plus 4,5 mA sur le port du DALY. À relever dans B5.
- **Correction** : empreinte pour passer R43 et R45 à 1 kΩ si besoin ; règle firmware de veille.

### 19. USB seul : retour du 3,3 V vers PACK_RAW
- [C] Mécanisme : PWR_3V3 → L1 → SW de U3 → diode interne du MOSFET haut → PACK_RAW (environ 2,6 V). Confirme le [H] de `BOM.md`. [M] courant à mesurer.

### 20. Annotation et nommage
- [C] Références de symboles d'alimentation en double (#PWR005, #PWR006, #PWR019 à #PWR025 ×2, #FLG904 ×4) : d'où l'avertissement « erreurs de numérotation » à l'export de la netlist. Sans effet sur la BOM.
- La feuille 03 s'appelle encore « Alimentations accessoires 12 V et 5 V ».
- **Correction** : réannoter les symboles d'alimentation, renommer la feuille.

### 21. Divers
- [H] La rondelle d'une vis M3 en H1 touche presque le boîtier de J10 (le connecteur finit vers x 163,4, l'empreinte du trou commence à 163,6).
- [C] Prix et lignes JLC (jlcsearch, stock non renseigné) : Q1/Q2 à 2,07 USD au lieu de 1,25 USD dans `BOM.md` ; 17 lignes Extended uniques (16 assemblées, BZ1 étant soudé main) ; environ 21,8 USD de pièces par carte.
- [C] J7 (cavalier de terminaison) : le cavalier lui-même n'est pas dans la BOM.

---

## Suggestions

### 22. Diode série et TVS locale pour les bucks (R1, point 7)
- Une diode 100 V 1 A en boîtier SMA entre PACK_RAW et l'entrée de U3 et U5, plus une SMAJ/SMBJ43A locale après la diode.
- Protège les bucks contre l'inversion de polarité ; avec une petite résistance série, limite aussi l'énergie des transitoires vus par les LMR36510. Chute de 0,7 V sans conséquence pour quelques dizaines de mA.
- Ne protège ni le TPS48111 (VS limité par R8) ni les feux.

### 23. Coupure matérielle en surtension sur VSW (R2, B7)
- Comparateur (par exemple TLV7031 avec référence) sur VSW qui force INP à l'état bas, donc PD ouvre Q1/Q2 en quelques µs.
- Le firmware via l'ADC de GPIO7 réagit en millisecondes. Avec ≈ 1 mF côté moteur et 10 A de régénération, le bus monte d'environ 10 V/ms : la TVS est atteinte avant.

### 24. Protéger le 3,3 V et le 5 V exportés vers le Display (J10)
- PTC ou résistance série de 1 Ω, plus une petite TVS, pour qu'un court-circuit du faisceau ne fasse pas tomber l'ESP32 maître (3,3 V partagé).

### 25. Topologie de la terminaison CAN (B1)
- Power Control est au milieu du bus : Display (J10) — Power Control — moteur (J6). J7 doit rester ouvert ; la terminaison va au Display et au moteur, selon la mesure B1.

### 26. Mécanique
- Ajouter un trou de fixation dans le quart bas-droit (zone ESP32 et USB, seulement tenue par H1 et H4 à droite).
- Serre-câble pour les deux XT60.
- Guide de lumière pour D4 ; accès à SW1, SW2 et USB à prévoir dans le boîtier.

### 27. Thermique de Q1
- Q1 est plus chaud que Q2 : son drain (SENSE) est aussi chauffé par le shunt (0,9 W à 30 A), et la copie In2 de SENSE n'a de vias que près du shunt.
- Ajouter des vias autour du drain de Q1 et du cuivre SENSE en B.Cu.

### 28. Chien de garde matériel (facultatif)
- Un chien de garde externe sur EN du TPS48111. Le chien de garde interne de l'ESP32 et les rappels à la masse (R15–R17) couvrent déjà un plantage simple.

---

## Passe 2 (même jour) : empreintes, fabrication, schéma

### 29. Important — Empreinte du shunt R1 inadaptée au RLP25 de 1 mΩ
- **Catégorie** : puissance, mesure.
- **Où** : R1 (91 ; 67,4) abs, empreinte maison `BionX_Power.pretty/R_Shunt_2512_Kelvin`.
- **Constat** :
  - Fiche TA-I RLP25 (C459678, §3.2 et §6) : pour R ≤ 4 mΩ, chaque borne fait **C = 2,0 mm** de long ; pastilles recommandées **3,1 mm de long × 4,0 mm de large, écart 1,3 mm**. Les 3 W nominaux supposent environ 300 mm² de pastilles et de pistes.
  - Empreinte actuelle : pastilles de courant de 1,6 × 2,55 mm, écart **4,6 mm** (centres à 87,9 et 94,1). C'est le motif d'une 2512 classique (R > 4 mΩ).
  - Bornes de la résistance de 87,8 à 89,8 et de 92,2 à 94,2 : seuls 0,9 mm sur 2,0 mm de chaque borne reposent sur la pastille, côté extérieur. Les prises Kelvin sont aussi à l'extérieur.
- **Statut** : [C] géométrie ; [M] erreur de mesure réelle.
- **Risque** : la mesure Kelvin inclut une partie des bornes. La résistance vue par le TPS48111 est supérieure à 1 mΩ et dérive en température. Les seuils baissent d'autant : à 1,2 mΩ effectifs, la surintensité passe de 39,7 à environ 33 A, tout près de la pointe moteur de 30 A. IMON est faussé. Joint de soudure et dissipation réduits.
- **Correction** : refaire l'empreinte selon la fiche (pastilles 3,1 × 4,0 mm, écart 1,3 mm), prises Kelvin sur le bord intérieur des pastilles, au plus près du corps. Mesurer la résistance effective au banc (courant connu, tension entre KP et KN).
- **Coût** : empreinte et reroutage local, pas de pièce.

### 30. Important — Pastille thermique des LMR36510 trop petite
- **Catégorie** : fabrication, thermique.
- **Où** : U3 (98 ; 100) et U5 (98 ; 114) abs, empreinte `HSOP-8-1EP_3.9x4.9mm_P1.27mm_EP2.3x2.3mm`.
- **Constat** : dessin TI DDA0008B (fiche LMR36510, fin du document) : pastille exposée de 2,11–2,71 × 2,8–3,4 mm (nominal 2,41 × 3,1) ; motif TI 2,71 × 3,4 mm avec vias de 0,2 mm au pas de 1,3 mm. L'empreinte utilisée fait 2,3 × 2,3 mm. La bibliothèque KiCad contient la bonne variante `HSOP-8-1EP_3.9x4.9mm_P1.27mm_EP2.41x3.1mm`.
- **Statut** : [C].
- **Risque** : environ 25 % de la pastille du boîtier n'est pas soudée ; contact thermique et masse réduits. Sans gravité à faible charge, mais c'est une erreur d'empreinte.
- **Correction** : passer à l'empreinte EP2.41 × 3.1 de série (le placement ne change pas), avec les vias thermiques du point 4.
- **Coût** : nul.

### 31. Mineur — Vias dans des pastilles CMS
- [C] Vias dans la pastille : U4.12 (TEMP_ADC, pastille crénelée du module) en (167,46 ; 110,6), R46.2 (131,6 ; 118,6), U8.2 GND (105,05 ; 124,45), R34.2 GND (105,17 ; 127,5). La doc ne citait que R34.
- [C] Vias à cheval sur le bord d'une pastille : J9 A1 / B12 en (116,64 ; 123,68), R48.1, C31.2, C23.2.
- **Risque** : la soudure s'écoule dans le via ; joint maigre, surtout sous le module et sur les broches fines de l'USB-C.
- **Correction** : déplacer ces vias hors des pastilles, ou demander à JLC des vias bouchés et recouverts.

### 32. Mineur — Notes du schéma périmées
- [C] Textes encore présents dans les feuilles :
  - « External 12 V lamp with its own current regulation » et « 06 REAR LIGHT / 12 V / 200 mA » (Accessoires) ;
  - « 02 12 V ACCESSORIES » et « 03 5 V BUZZER SUPPLY / FROM SWITCHED 12 V » (Alim_12V) ;
  - « Feu arrière 12 V / 200 mA » et « Le 12 V accessoires est commandé… » (feuille racine), « Le 12 V accessoires reste alimenté par le pack » (USB) ;
  - « MESURE COURANT / INA228 » (Commande) ;
  - « Precharge: 42V / 100R… » (Power_Switch).
- **Risque** : acheter des feux 12 V pour des sorties qui sont en tension pack (42 V permanents).
- **Correction** : réécrire ces notes : « feux 12–60 V, + en tension pack permanente », précharge 2 × 220 Ω, mesure par IMON.

### 33. Mineur — Fichiers BOM et placement pour JLC
- [C] Le fichier de placement exporté contient 121 lignes : J4 et J5 (non montés) sont bien exclus, mais tous les traversants soudés main y figurent (BZ1, J2, J3, J6 à J13).
- [C] BZ1 a un code LCSC (C96093) alors que son montage est « manuel » : s'il reste dans la BOM envoyée à JLC, JLC le fournira et le soudera (frais de traversant).
- **Correction** : générer la BOM JLC en filtrant `Montage = manuel`, ou vider le champ LCSC de BZ1.

### 34. Mineur — Modèles 3D manquants
- [C] Aucun modèle pour U1 et R1 (empreintes maison). Fichiers absents de la bibliothèque KiCad 10 pour : XT60PW-M horizontal (J2, J3), fusible 1812 (F1, F2), USB-C HRO (J9), HSOP-8 EP2.3 (U3, U5), TS-1187A (SW1, SW2).
- **Conséquence** : la vue 3D ne permet pas de vérifier les hauteurs ni l'intégration au boîtier.
- **Correction** : STEP du fabricant (LCSC fournit souvent un modèle EasyEDA) ; le point 30 règle U3 et U5.

### 35. Suggestion — Diviseur de retour du buck 3,3 V
- [C] R2 + R3 = 67 kΩ consomment 50 µA en permanence sur le 3,3 V. TI indique RFBT = 1 MΩ pour le courant de repos minimal en PFM (fiche LMR36510, caractéristiques électriques).
- Gain faible (environ 5 µA côté pack) ; à ne faire que si la veille devient un sujet.

### Confirmations de la passe 2 (aucune action)
- **Plans de référence** [C] : sous chaque piste de signal, le plan voisin (In1 sous F.Cu, In2 sous B.Cu) est la GND, hors trous de vias. Seules exceptions : MAIN_GATE en B.Cu au-dessus de FET_SRC sur 12 mm et SHUNT_KN au-dessus de SENSE, ce qui est voulu (retour par la source). La paire CAN et l'USB ont la GND en référence sur toute leur longueur.
- **Remplissage des zones à jour** [C] : un DRC avec `--refill-zones` sur une copie donne les mêmes surfaces au mm² près et 0 problème de connexion.
- **Variante du LMR36510** [C] : LMR36510**A**DDAR = variante PFM (fiche, tableau de comparaison p. 3), courant de repos sans commutation de 26 µA typique. Bon choix pour le 3,3 V permanent.
- **Veille estimée** [H] : 3,3 V ≈ 70–80 µA (diviseur 50 µA, ESP32-S3 en sommeil profond environ 10–20 µA, TCAN3413 en veille 1 + 9 µA d'après sa fiche), soit environ 35 µA côté pack ; plus U5 arrêté (5,3 µA), TPS48111 arrêté (1,6 µA) et fuites des TVS (≤ 1 µA). Total ≈ 45 µA, hors DALY et Display [M] : négligeable devant le pack de 9,3 Ah.
- **Ordre de grandeur du TOLL** [C] : Qg = 217 nC par MOSFET, soit 434 nC pour Q1 + Q2 ; règle TI §9.2.2.7 : CBST ≥ Qg total / 1 V = 0,43 µF. C10 = 1 µF X7R 50 V (environ 0,7 µF sous 12 V) convient. Charge de C10 par la pompe de 100 µA : environ 120 ms avant le premier allumage.

---

## Passe 3 (même jour) : GPIO, transitoires, mesures, sorties de fabrication

### 36. Mineur — Impulsions des GPIO de l'ESP32-S3 à la mise sous tension
- **Source** : fiche ESP32-S3 v2.2, tableau 2-2 p. 18. GPIO1 à 14, 17 et 18 sont forcés à l'état bas pendant environ 60 µs à la mise sous tension ; GPIO18 passe aussi 60 µs à l'état haut ; GPIO19 et 20 ont une impulsion haute.
- **Conséquences sur cette carte** :
  - [C] GPIO10 à 14 (EN, précharge, principal, 5 V, feu arrière) : impulsion basse, donc sans danger. Confirme R4.
  - [H] GPIO6 (CAN_STB) et GPIO4 (CAN_TX) bas en même temps : le TCAN3413 peut quitter la veille et envoyer environ 7 bits dominants (60 µs à 125 kbit/s) à chaque mise sous tension ou reset. Effet probable : une trame d'erreur sur le bus moteur.
  - [H] GPIO1 (WAKE_SPARE) bas : impulsion de réveil de 60 µs vers le Display par R46.
  - [H] GPIO5 (CAN_RX) forcé bas contre la sortie RXD du TCAN3413 : conflit de 60 µs.
  - [H] GPIO18 forcé haut contre FLT_T (tirage bas interne de 70 Ω) si un défaut thermique est actif : courant au-delà des 10 mA admis. Peu probable tant que DIODE est à la GND.
- **Correction** :
  - déplacer CAN_STB et CAN_TX sur des GPIO sans impulsion (libres : 21, 39 à 42), ou l'accepter et le vérifier au banc (B4) ;
  - résistances série de 1 kΩ sur FLT_I et FLT_T (TI, tableau 9-3, recommande 1 à 10 kΩ) ;
  - GPIO47 et 48 sont alimentés par VDD_SPI : garder un module à mémoire flash 3,3 V (N8 convient).

### 37. Important (aggrave le point 7) — Court-circuit franc en sortie : surtension d'entrée
- **Calcul** [H] :
  - Courant dans la boucle (câble pack ≈ 1 µH) : 42 A/µs. Déclenchement à 58,7 A, puis 1,2 µs de réaction : environ 110 à 120 A au moment de la coupure.
  - Impédance caractéristique câble / condensateurs : Z = √(L/C) ≈ √(2 µH / 2,5 µF) ≈ 0,9 Ω, soit une pointe théorique de 42 + 120 × 0,9 ≈ 150 V.
  - La SMCJ48A est alors poussée bien au-delà de son courant nominal (19,4 A) : son écrêtage dépasse probablement 90 à 100 V.
  - Composants exposés : TPS48111 (VS, CS+, CS− : 100 V max ; seul VS est filtré par R8 et C9), LMR36510 (70 V), MOSFET (100 V ; avalanche de 1,6 J admise).
  - Avec 100 µF électrolytique sur PACK_RAW : Z ≈ 0,14 Ω, soit environ +17 V : pointe vers 59 V.
- **Statut** : [H] ; [M] inductance réelle du câble.
- **Correction** : le condensateur tampon du point 7 ; en complément, une TVS plus robuste (boîtier SMDJ). Premier essai de court-circuit au banc avec une alimentation limitée en courant, jamais sur le pack.

### 38. Mineur — Mesures ADC
- [H] **IMON** (45,9 mV/A) : 1 A donne 46 mV. L'ADC de l'ESP32-S3 avec 12 dB d'atténuation est imprécis près de 0 V : sous 1 à 2 A, la mesure est peu fiable (offset du TPS48111 ±0,2 A en plus). Firmware : atténuation plus faible pour les petits courants.
- [C] **Sonde CTN** : le pont est alimenté par GPIO16 alors que l'ADC mesure en absolu. Un écart de 1,5 % du rail 3,3 V donne environ 0,7 °C d'erreur à 25 °C (sensibilité ≈ 2,2 %/°C). Acceptable. À 80 °C : environ 0,37 V et 11 mV/°C.
- [C] **VSW** : 42 V donne 2,63 V, 48 V donne 3,0 V. L'ADC sature vers 52 V : suffisant pour la précharge et une alerte de surtension modérée, pas pour mesurer une pointe.

### 39. Mineur — Sorties de fabrication
- [C] Le fichier job des Gerbers annonce l'empilement par défaut de KiCad : 4 × 35 µm, diélectriques de 0,48 mm. Il ne correspond pas à l'empilement JLC (cuivre intérieur 0,5 oz, préimprégné d'environ 0,21 mm). JLC imposera le sien ; ce sont les calculs d'impédance et thermiques qui sont faux. À régler avec le point 13 (empilement JLC dans la configuration de la carte).
- [C] Contour rectangulaire à angles vifs (112 × 82 mm). Suggestion : rayons de 1 à 2 mm.
- [C] Gerbers, perçages et fichier de placement s'exportent sans erreur (11 couches, 2 fichiers de perçage).

### 40. Suggestion — Détecter la coupure d'alimentation pour l'odomètre
- Exigence du firmware (`bionx_console/project_requirements.md`, l. 188–189) : pas de sauvegarde garantie à l'extinction sans détection de coupure et réserve d'énergie matérielles.
- Rien sur la carte ne prévient d'une coupure du DALY ou d'un débranchement du pack, et le 3,3 V n'a que des céramiques.
- **Correction** : le diviseur de R3 (150 kΩ / 10 kΩ sur GPIO3) et le condensateur du point 7 donnent un préavis de quelques millisecondes pour écrire. R3 remonte en priorité.

### 41. Suggestion — GPIO libres non sortis
- GPIO3, 21, 35 à 37, 39 à 42, 45 et 46 ne sont reliés à rien. Deux à quatre pastilles de réserve (par exemple GPIO21 et 39 à 42) permettraient d'ajouter plus tard une seconde sonde ou un frein local de secours sans refaire la carte. Ne pas tirer GPIO45 vers le haut (fiche du module).

### Remarques firmware (sans impact PCB)
- Le banc actuel lit tension et courant dans le nœud batterie BionX (0x10, jauge D3/D4). Ce nœud n'existe plus avec le DALY : B3 doit dire si le moteur l'exige, auquel cas Power Control doit l'émuler.
- Le frein est sur le Display, donc transmis par CAN : la perte des trames du Display doit être traitée comme un freinage (règle « frein absent = assistance interdite » de `project_requirements.md`).
- Précharge : limiter les tentatives (énergie de 0,88 J par résistance à 2 mF) et borner la durée (environ 5τ, soit 1,1 s à 2 mF).

---

## Passe 4 (même jour) : empreintes restantes, AMS1117, points de test, assemblage

### 42. Mineur — Empreinte SOP-4 des EL357N plus courte que le motif Everlight
- **Où** : U12 (77 ; 114) et U13 (86,5 ; 114) abs, empreinte `SOP-4_3.8x4.1mm_P2.54mm`.
- **Constat** [C] : fiche Everlight EL357N-G p. 7 : pattes de 7,00 ± 0,30 mm de bout à bout ; pastilles recommandées 1,5 × 0,6 mm, 4,85 mm d'écart intérieur, 7,85 mm hors tout (centres à ±3,175 mm). Empreinte utilisée : 1,45 × 0,55 mm, centres à ±2,75 mm, soit 4,05 mm d'écart intérieur et 6,95 mm hors tout. Le bout des pattes (jusqu'à 7,3 mm) dépasse les pastilles.
- **Risque** : pas de ménisque en bout de patte, soudure plus fragile aux vibrations.
- **Correction** : empreinte au motif Everlight. Bonus : 0,8 mm de plus entre le côté BMS et le côté ESP32 (point 8).

### 43. Mineur — AMS1117 : charge minimale et dissipation
- [C] Fiche AMS1117 : charge minimale de 5 à 10 mA pour garantir la régulation, courant de repos de 5 à 11 mA, θJA de 90 °C/W en SOT-223 (46 à plus de 90 selon le cuivre).
- [C] Le net USB_3V3 (languette du régulateur) n'a que 17 mm² de cuivre, en F.Cu seulement.
- [H] En USB seul, ESP32 en émission Wi-Fi (pointe de 355 mA d'après la fiche du module) et Display alimenté [M] : jusqu'à environ 0,85 W, soit environ +77 °C. Acceptable au banc, sans marge.
- [H] Pack et USB ensemble : l'AMS1117 et le buck (3,35 V) se partagent la charge ; sans danger, la charge de la carte dépasse la charge minimale.
- **Correction** : polygone USB_3V3 sous la languette en F.Cu et B.Cu avec quelques vias ; voir aussi le point 14 (condensateur de sortie).

### 44. Suggestion — Points de test utiles à la mise en route
- Il n'y a pas de point de test sur :
  - BST du TPS48111 : vérifier que la pompe de charge atteint 12 V par rapport à FET_SRC (TP19) avant le premier allumage ;
  - USB_3V3 : stabilité de l'AMS1117 (point 14) ;
  - nœuds SW des deux bucks, utiles avec une masse de sonde courte à côté.
- **Coût** : 2 à 4 pastilles, hors BOM.

### 45. Mineur — Composants CMS au ras du bord pour l'assemblage JLC
- [C] U4 est au ras du bord bas (le module finit à y 131,9 pour un bord à 132,0), J9 aussi, SW2 à environ 0,8 mm, C34, R55 et R56 à environ 2,9 mm du bord droit.
- [H] Selon ses règles du moment, JLC peut exiger des bords techniques (rails) pour l'assemblage CMS quand des composants sont trop près du bord. À vérifier au devis : surcoût possible.

### Confirmations de la passe 4 (aucune action)
- [C] Repère de broche 1 présent sur toutes les pièces polarisées, y compris l'empreinte maison de U1 (point en (−3,25 ; −2,25), à côté de la broche 1). Barre de cathode sur D2 et D3. R1 n'a pas de contour en sérigraphie, mais il n'est pas polarisé.
- [C] Buzzer TMB12A05 : diamètre 12 mm, hauteur 9,5 mm (fiche), pas de 7,6 mm conforme au nom de l'empreinte.
- [C] Pointe d'émission Wi-Fi de l'ESP32-S3 : 355 mA (802.11b, 20,5 dBm, fiche du module). Cela appuie le point 3.
- Non vérifié : TS-1187A et USB-C contre leur fiche, qui ne sont que des images (empreintes KiCad nommées pour ces références).

---

## Synthèse

| Gravité | Nombre | Points |
|---|---:|---|
| Bloquant | 2 | 1, 2 |
| Important | 12 | 3 à 11, 29, 30, 37 |
| Mineur | 20 | 12 à 21, 31 à 34, 36, 38, 39, 42, 43, 45 |
| Suggestion | 11 | 22 à 28, 35, 40, 41, 44 |

## Vérifications non faites

- Polarité moulée du XT60 réel : il faut la pièce.
- Rotations du fichier de placement JLC : il faut l'aperçu de JLC.
- Stock d'assemblage réel : jlcsearch renvoie un stock vide.
- Boîtier et vue 3D : pas de modèle de boîtier.
- Simulation thermique : seulement des estimations IPC-2221.
- Impédance de la paire USB : empilement non défini.
- Brochage USB-C contre sa fiche : la fiche est une image, l'empreinte KiCad a été reprise.
- Valeurs à mesurer : capacité du moteur, inductance du câble pack, courant des feux, seuils et niveaux du DALY.

## Points R et B modifiés par cet audit

| Point | Effet de l'audit |
|---|---|
| R1 | Confirmé. Correction peu coûteuse : point 22. Le danger le plus probable est le point 1 (repères XT60). |
| R2 | Confirmé (IMON unidirectionnel). Ajout du point 23. |
| R3 | Pour la mesure : priorité basse (MOSFET fermés, VSW_ADC donne déjà la tension pack). Pour détecter une coupure et sauvegarder l'odomètre : priorité remontée (point 40). |
| R4 | Confirmé dans la netlist (R15–R17, R19, R20, R23, R31, R33, R41 ; STB tiré haut par R28). |
| R5, R6, R7 | Inchangés. C14663 est bien Basic chez JLC (jlcsearch). |
| B1 | Précisé par le point 25. |
| B2 | Inchangé ; réserve sur l'énergie des résistances de précharge maintenue. |
| B5 | Élargi : niveau du TX du DALY en veille et consommation induite (point 18). |
| B7 | Aggravé par les points 7 et 37 ; voir aussi le point 23. |
| B4 | À compléter : vérifier qu'un reset de l'ESP32 ne perturbe pas le bus (point 36). |
| P2.4 (seuil de surintensité) | Fragilisé par le point 29 : la résistance effective du shunt n'est pas 1 mΩ avec l'empreinte actuelle ; seuils à revalider après correction de l'empreinte. |
| P3.3 (empreintes à vérifier) | LMR36510 non conforme (point 30), shunt non conforme (point 29) ; TOLL, ESP32, USB-C conformes. |

---

## Journal des corrections

Chaque entrée dit ce qui a été modifié, où, et comment c'est vérifié. Les sauvegardes avant modification sont dans le commit `daace9a`.

### 2026-11-26 — lot 1 (PCB et schéma, sans nouvelle pièce)

Vérification après le lot : ERC 0, DRC 0 (zones recalculées), 0 élément non connecté, 0 écart de parité.

- **Point 1** : sur J2 et J3, les textes « + » et « − » de l'empreinte sont permutés. Le « + » est maintenant à côté de la broche 1 (PACK_RAW sur J2, PACK_SW sur J3), le « − » à côté de la broche 2 (GND). Ajout de « +PACK » à côté de la broche 1 de J8 et J11. **Reste** : vérifier sur un vrai XT60PW-M où est le « + » moulé et câbler les fiches pack et moteur comme la carte.
- **Point 6** : connexion pleine (sans rayons) imposée sur les pastilles J2.2 et J3.2 (GND), sur toutes les couches. In2 n'atteint toujours pas J3.2 (la zone PACK_SW de In2 l'entoure) ; F.Cu, In1 et B.Cu suffisent. Prévoir un fer puissant pour souder.
- **Point 12** : 207 vias convertis de « buried » en traversants.
- **Point 13** : empilement JLC 4 couches ajouté dans la carte : 35 µm extérieur, 17,5 µm intérieur, préimprégné 7628 de 0,2104 mm, noyau de 1,065 mm, épaisseur 1,6 mm. Le passage à 2 oz extérieur se décide au devis.
- **Point 17** :
  - textes ajoutés : « BIONX POWER CONTROL » et « v1.0 2026-11 » (au-dessus de TP16), avertissement « ATTENTION 42 V : + de J8 et J11 sous tension pack permanente » (bas gauche), brochage de J12 (« 3V3 TX RX GND ») et de J10 (« 1 GND 2 3V3 3 5V 4 CANH 5 CANL 6 WAKE ») ;
  - traits de toutes les sérigraphies portés à 0,15 mm (150 textes) ;
  - 31 références déplacées pour ne plus chevaucher de pastille ; il reste 25 sérigraphies coupées par le masque (JLC les rogne) et 18 chevauchements entre sérigraphies, sans effet électrique.
  - Hauteur des références laissée à 0,8 mm : à 1,0 mm, les chevauchements augmenteraient.
- **Point 20** : symboles d'alimentation renumérotés (#PWR001 à #PWR117, #FLG001 et #FLG002) ; l'avertissement « erreurs de numérotation » a disparu. Feuille 03 renommée « Alimentation 5 V accessoires ».
- **Point 30** : U3 et U5 passent sur l'empreinte KiCad `HSOP-8-1EP_3.9x4.9mm_P1.27mm_EP2.41x3.1mm` (schéma et PCB). Le via FB de U3 est descendu de 0,25 mm (97,67 ; 102,27) pour garder 0,2 mm avec la pastille agrandie. La variante avec vias thermiques a été écartée pour l'instant : sa pastille en B.Cu court-circuite les pistes FB et ACCESSORY_EN qui passent sous les circuits. Les vias thermiques seront ajoutés avec la reprise des bucks (point 4).
- **Point 32** : notes réécrites dans les feuilles racine, Accessoires, Alim_3V3, Alim_12V, CAN, USB, Power_Switch et Commande (feux en tension pack, 5 V accessoires, diviseurs 47k/20k et 82k/20k, seuils 39,7 A et 58,7 A, précharge 2 × 220 Ω, TVS SMCJ48A, mesure par IMON, cavalier J7 ouvert).
- **Point 33** : le champ LCSC de BZ1 est vidé ; la référence d'achat est conservée dans un champ « Achat » (schéma et PCB). La BOM JLC ne contient plus BZ1.
- **Point 39** : contour avec coins arrondis de 1,5 mm.

### 2026-11-26 — lot 2 (nouvelles pièces : fusibles, découplage ESP32, condensateur tampon)

Vérification après le lot : ERC 0, DRC 0, 0 élément non connecté, 0 écart de parité.

- **Point 2** : F1 et F2 deviennent des fusibles non réarmables Littelfuse Nano2 **0451001.MRL** (1 A rapide, 125 V, pouvoir de coupure 50 A ; LCSC C3099, Extended, ≈0,32 USD). Symbole `Device:Fuse` à la place de `Device:Polyfuse`, empreinte `Fuse:Fuse_Littelfuse-NANO2-451_453`.
  - F1 décalé de 0,25 mm vers la gauche (74,75 ; 86) et F2 de 0,75 mm (74,25 ; 98) pour dégager Q4 et Q6.
  - Le via du drain de Q4 (J8 broche 2) passe de (78,33 ; 85,93) à (79,56 ; 87,45), sous Q4.
  - L'arrivée PACK_RAW de F1 est reroutée : via déplacé en (75,6 ; 88,0), piste F.Cu vers la broche 1, piste B.Cu vers F2 et D2.
  - Note des feux du schéma mise à jour.
  - **Reste** [M] : courant d'appel des feux à la mise sous tension, pour vérifier qu'un fusible rapide de 1 A ne fond pas.
- **Point 3** : ajout de **C35** (10 µF 0805, C15850) et **C36** (100 nF 0603, C14663) entre PWR_3V3 et GND, au bord droit, contre la broche 2 du module (C36 à environ 2 mm de la broche). Trois vias de couture GND déplacés pour leur faire de la place, un via GND ajouté entre les deux.
  - **Non fait** : rapprocher le RC d'EN (R26, C23). La broche 3 est collée à la broche 2, le corps du module occupe la gauche et la bande de droite est maintenant prise par C35 et C36.
- **Points 7 et 37** : ajout de **C37**, électrolytique aluminium **100 µF / 63 V** CMS 10 × 10,5 mm (CS1J101M-CRG10, LCSC C116430, Extended, ≈0,09 USD), sur PACK_RAW, dans la feuille 02.
  - Placé en (88 ; 57,2) abs, juste au-dessus du polygone PACK_RAW et à côté de J2 : la pastille + est dans le polygone, la pastille − dans la masse F.Cu, avec 4 vias GND.
  - Un via de couture PACK_RAW et 6 vias GND qui tombaient sous les pastilles ont été retirés.
  - Ordre de grandeur : 120 A coupés avec 2 µH de câble donnent environ +17 V au lieu d'environ +100 V sans C37.
  - **Reste** [M] : mesurer au banc la pointe à TP15 au branchement du pack et à la coupure.

### 2026-11-26 — lot 3 (shunt, mesure Kelvin, filtre ISCP)

Vérification après le lot : ERC 0, DRC 0, 0 élément non connecté, 0 écart de parité.

- **Point 29** : nouvelle empreinte `BionX:R_Shunt_2512_Kelvin_RLP25`, d'après la fiche TA-I (pastilles 3,1 × 4,0 mm, écart 1,3 mm). Chaque pastille est partagée entre la force (pastilles 1 et 4, 3,1 × 2,85 mm) et la prise Kelvin (pastilles 2 et 3, 1,65 × 0,85 mm), placée du côté intérieur de la borne, contre le corps de la résistance. R1 reste en (91 ; 67,4).
- **Point 5** :
  - **Paire KP / KN** : les deux prises descendent côte à côte entre les polygones PACK_RAW et SENSE, passent en B.Cu sous la piste PACK_RAW de 0,6 mm (x 90,6 et 91,4, pas de 0,8 mm), puis reviennent en F.Cu en paire serrée (0,2 / 0,2 mm, y 77,2 et 77,6) jusqu'à U1. KP ne traverse plus les polygones SENSE et FET_SRC.
  - **R9 (RSET) et R10 (RISCP)** sont remontés contre U1, en (116,3 ; 78,3) et (116,3 ; 79,9). CS+, ISCP et KN arrivent sur les broches 18, 19 et 17 par l'intérieur du boîtier ; CS+ fait un saut en B.Cu pour croiser ISCP.
  - **C38** : ajout d'un 1 nF 0402 X7R (0402B102K500NT, LCSC C1523, Basic) entre ISCP et CS− (SHUNT_KN), comme le recommande TI (§9.5). Placé en (113,85 ; 81,9), à cheval sur les pistes KN et ISCP, à 2 mm de U1. Dans le schéma, il est relié par une étiquette « ISCP » (le net `Net-(U1-ISCP)` s'appelle maintenant `/01  Commutation et précharge/ISCP`). Constante de temps avec R10 : 3,3 kΩ × 1 nF = 3,3 µs ; la protection contre les courts-circuits répond donc en environ 4 à 5 µs au lieu de 1,2 µs.
  - **Surface de la boucle de mesure** : environ 13 mm² au lieu d'environ 153 mm².
  - **Modifications voisines** :
    - via INP_G descendu de 0,36 mm, en (103,23 ; 78,45) ;
    - via FLT_T déplacé en (112,55 ; 85,6) ;
    - un via GND sous U1 et 7 vias de couture GND du couloir retirés ;
    - deux vias PACK_RAW devenus orphelins sous l'ancien shunt retirés ;
    - **TP21 (TMR) passe en face arrière** en (97,5 ; 78,85), avec un via, faute de place en face avant (12 points de test en face arrière au lieu de 11).
  - **Reste** [M] : mesurer au banc la résistance effective vue entre KP et KN (courant connu), puis revalider les seuils de 39,7 A et 58,7 A.

### 2026-11-26 — lot 4 (boucles des bucks)

Vérification après le lot : ERC 0, DRC 0, 0 élément non connecté, 0 écart de parité.

- **Point 4** :
  - **Bootstrap** : C4 (U3) et C15 (U5) passent en 0402 100 nF 50 V X7R (CL05B104KB54PNC, LCSC C307331, Basic) et sont collés aux broches 7 et 8, en (102,25 ; 98,73) et (102,25 ; 112,73). Ils étaient à environ 14 mm, en (114 ; 95,5) et (114 ; 112,5).
  - **Nœuds SW** : les vias et les pistes B.Cu qui menaient le nœud SW jusqu'à ces condensateurs sont supprimés. Chaque nœud SW se limite maintenant à la broche 8, au condensateur de bootstrap et à la pastille de l'inductance. IMON_ADC n'a plus de nœud SW à proximité en B.Cu.
  - **VCC** : C3 (U3) et C14 (U5) passent en 0402 1 µF 25 V X5R (CL05A105KA5NQNC, LCSC C52923, Basic), collés à la broche 6 en (102,25 ; 101,27) et (102,25 ; 115,27). Leur GND rejoint In1 par un via de 0,2 mm en (103,6 ; 102,3) et (103,6 ; 116,3). C3 était à environ 5 mm et C14 à environ 13 mm.
  - **Vias thermiques** : 4 vias de 0,2 / 0,45 mm dans la pastille thermique de U3 et de U5, vers In1 (GND), placés hors des pistes FB et ACCESSORY_EN qui passent dessous en B.Cu.
  - **Non fait** : rapprocher C12 (2,2 µF d'entrée de U5, toujours en (115 ; 116,5)). La place à gauche de U5 est prise par C13, R23, l'arrivée PACK_RAW et les pistes ACCESSORY_EN, FB, DALY et BMS. Le 5 V ne débite que quelques dizaines de mA ; C13 (100 nF) reste à environ 6 mm de VIN. À reprendre si la CEM pose problème au banc.

### 2026-11-26 — lot 5 (TVS CAN) et arrêt de la session

Vérification : ERC 0, DRC 0, 0 élément non connecté, 0 écart de parité.

- **Point 9** : D1 (PESD1CAN) passe en « non monté », dans le schéma et le PCB (hors BOM). Il n'existe pas au catalogue JLC de TVS CAN en SOT-23 qui tienne 42 V. Le TCAN3413 tient seul ±58 V de défaut sur le bus et ±10 kV IEC 61000-4-2. L'empreinte est gardée : on peut souder une PESD2CAN24 si on estime qu'un court-circuit du câble moteur sur le pack est improbable.

**Reste à traiter** (session interrompue par la limite d'usage), dans cet ordre :
1. Point 11 : MMBT3904 sur DIODE (U1 broche 10, aujourd'hui reliée à la GND par le fil (184,15 ; 175,26)-(184,15 ; 180,34) de la feuille 01), à placer contre Q1.
2. Point 8 et point 42 : isolement BMS (zone sans cuivre sous U12 et U13, classe de nets BMS), empreinte SOP-4 au motif Everlight.
3. Point 10 : zone sans cuivre de 3,5 mm autour des trous et paire CAN éloignée de H1.
4. Points 14, 15, 16, 31, 36, 43, 44, puis les suggestions 22 à 28, 35, 40 et 41.
5. Point 3 (RC d'EN) et point 4 (C12) : laissés en place faute de place.

### 2026-11-26 — lot 6 (protection thermique, isolement BMS, optocoupleurs)

Vérification après le lot : ERC 0, DRC 0 (y compris la nouvelle règle d'isolement BMS), 0 élément non connecté, 0 écart de parité.

- **Point 11** :
  - Schéma, feuille 01 : la broche DIODE de U1 n'est plus à la GND. Ajout de **Q9** (MMBT3904, SOT-23, LCSC C20526, Basic ; base et collecteur sur DIODE, émetteur à la GND) et de **C39** (1 nF 0402, C1523) en parallèle.
  - PCB : Q9 est collé sous le drain de Q1, en (105,3 ; 74,5), entre les polygones SENSE et FET_SRC (Q1 est le MOSFET le plus chaud, à côté du shunt). C39 est sous la broche 10 de U1, en (110,9 ; 89,75), avec un via GND.
  - La ligne DIODE fait un détour d'environ 50 mm (6 vias) : la barrière formée en B.Cu par MAIN_ON et FLT_T sous U1 interdit un chemin direct. C'est acceptable pour cette ligne (courant de 10 / 160 µA, filtrée par C39), mais à reprendre si la lecture de température est bruitée.
  - FLT_T (GPIO18) signale maintenant un dépassement de 150 °C au niveau de Q9, et le TPS48111 coupe Q1 et Q2 de lui-même.
- **Point 42** : nouvelle empreinte `BionX:SOP-4_EL357N_Everlight` (pastilles 1,5 × 0,6 mm, centres à ±3,175 mm, 4,85 mm entre rangées), affectée à U12 et U13.
- **Point 8** : bloc de l'UART DALY entièrement replacé et reroutée :
  - U12 tourné de 90° et U13 de 270° : leurs côtés BMS (LED de U12, transistor de U13) sont maintenant en bas, face à J12, et leurs côtés ESP32 en haut. La barrière passe horizontalement au milieu des boîtiers.
  - J12 déplacé en (78,5 ; 127,8), sous les optocoupleurs ; R42 et R45 (côté BMS) entre les deux ; R43 et R44 (côté ESP32) au-dessus ; TP9 et TP10 (face arrière) au-dessus.
  - Nouvelle classe de nets **BMS** (`*BMS_*`, `Net-(R42-Pad2)`) et règle de conception `BionX_Power_Control.kicad_dru` : **0,5 mm minimum entre un net BMS et tout autre net**, sur toutes les couches. Elle passe sans violation.
  - Zones interdites aux plans (`ISOLEMENT_BMS_F`, `_In1`, `_In2`, `_B`) à 1 mm autour de tout le cuivre BMS : aucun plan de masse de la carte sous le domaine BMS, sur aucune couche.
  - L'arrivée PACK_RAW de U5 (via et piste vers C13) a été décalée pour s'écarter de U13.
  - Textes de sérigraphie déplacés (« 42 V PERMANENT / SUR + DE J8, J11 », « J12 DALY : 3V3 TX RX GND »).
  - Le routage de ce bloc a été fait par un petit routeur A* écrit pour l'occasion (grille 0,1 mm, F.Cu et B.Cu, règles du projet), puis vérifié par le DRC de KiCad.

### 2026-11-26 — lot 7 (trous de fixation, AMS1117, vias, isolement HV, FLT, points de test)

Vérification après le lot : ERC 0, DRC 0 (règles BMS et HV comprises), 0 élément non connecté, 0 écart de parité.

- **Point 10** : zones interdites `FIXATION_H1` à `_H4` (rayon 3,5 mm, F.Cu et B.Cu : ni piste, ni via, ni plan) autour des quatre trous. 15 vias GND retirés. La paire CAN est redescendue en y = 59,0 / 59,4 pour s'écarter de H1.
- **Point 14** : C28 devient un tantale 10 µF 16 V, TAJA106K016RNJ (LCSC C7171, Basic), empreinte `CP_EIA-3216-18_Kemet-A`. Son ESR place l'AMS1117 dans sa zone de stabilité.
- **Point 43** : zone `USB_3V3_F` (priorité 6) en F.Cu, rectangle (123,9 ; 108)-(127,4 ; 113), sous la languette de U9. L'ancienne zone B.Cu et le via qui la reliait sont supprimés.
- **Point 31** :
  - vias retirés ou déplacés hors des pastilles : U8.2 (GND, via sous le boîtier en (106,35 ; 124,5)), R34.2, languette de U9, U4.12 (TEMP_ADC, via déplacé en (170,1 ; 110,39)) ;
  - exception : le via WAKE_RAW de R46.2 (131,6 ; 118,6) reste en bord de pastille. Les pistes DALY_TX, MCU_BOOT, IMON_ADC et FRONT_LED_EN l'entourent, il n'y a pas d'autre place. Les vias sont bouchés (tenting) des deux côtés, donc l'étain ne descend pas dedans ; à surveiller à l'inspection de la première série.
- **Point 15** : nouvelle règle « Isolement HV » dans `BionX_Power_Control.kicad_dru` : **0,3 mm minimum entre un net au potentiel du pack et un net basse tension**, sur toutes les couches. Nets HV : PACK_RAW, PACK_SW, SENSE, SHUNT_KP, SHUNT_KN, J8 / J11, Q3-D, FET_SRC, MAIN_GATE, PRE_GATE, VS_FILTERED, ISCP, BST, CS+, PU.
  - Choix de 0,3 mm et pas 0,4 à 0,5 mm : IPC-2221B, tableau 6-1, demande 0,13 mm pour 31 à 50 V sous vernis (B4) et 0,6 mm sur cuivre nu (B2). Toutes les pistes sont sous vernis, et l'écart minimal entre pastilles nues HV et BT est déjà de 0,64 mm. 0,3 mm donne plus de deux fois la marge sous vernis. À 0,4 mm, 34 conflits imposaient de reprendre tout le placement autour de U1, U3 et U5.
  - 8 conflits corrigés :
    - VS_FILTERED décalé sous C9 (y = 82,15) ;
    - via GND de C9 déplacé en (120,45 ; 79,85) ;
    - INP_G écarté de SHUNT_KN (y = 78,12) ;
    - via FLT_I déplacé en (116,46 ; 89,08) ;
    - TP5 décalé de 0,5 mm le long de sa piste.
  - Les nets à 42 V restent entre eux à 0,2 mm (même domaine).
- **Point 36** (partiel) :
  - Schéma, feuille 04 : **R58** et **R59**, 1 kΩ 0402 (0402WGF1001TCE, LCSC C11702, Basic), en série entre U4 (GPIO17, GPIO18) et les étiquettes POWER_FLT_I / POWER_FLT_T. Le tirage, J4, TP5 / TP6 et U1 restent côté TPS48111. L'impulsion haute de GPIO18 au démarrage est maintenant limitée à 3,3 mA.
  - PCB : R58 et R59 placées entre J4 et le module, en (146,6 ; 110,4) et (147,4 ; 112) ; nouveaux nets `Net-(U4-IO17)` et `Net-(U4-IO18)`.
  - **Reste** : déplacer CAN_STB (GPIO6) et CAN_TX (GPIO4) sur des GPIO sans impulsion au démarrage (21, 39 à 42). Cela change le brochage du firmware : à décider, sinon à vérifier au banc (B4).
- **Point 44** : quatre points de test D1,0 mm, hors BOM :
  - **TP26** BST, face avant, sur la piste en (119,6 ; 86,4) ;
  - **TP27** SW_3V3, **TP28** SW_5V, **TP29** USB_3V3, face arrière, avec un via sur la piste.
  - Les nets anonymes ont reçu un nom (`BST`, `SW_3V3`, `SW_5V`) ; classes de nets et règle HV mises à jour.
- **Point 16** (non traité) : la seule place libre pour C10 est à 2,6 mm de U1.12 (au lieu de 9 mm), mais il faudrait passer BST sous PRE_GATE par deux vias. Le gain est faible pour une pompe de charge qui commute rarement. Les 0 Ω de grille demandent aussi de reprendre le placement de Q1 / Q2. À faire si on reprend la zone U1.

**Reste à traiter** :
- point 36 : choix des GPIO du CAN (firmware) ;
- point 16 : placement autour de U1 ;
- point 34 : modèles 3D (téléchargement des STEP à valider) ;
- point 3 (RC d'EN) et point 4 (C12) : pas de place ;
- point 21 et suggestions 22 à 28, 35, 40, 41.

### 2026-11-27 — lot 8 (suggestions, documentation, BOM)

Vérification après le lot : ERC 0, DRC 0 (règles BMS et HV comprises), 0 élément non connecté, 0 écart de parité.

- **Point 27** (thermique de Q1) : nouvelle zone SENSE en **B.Cu**, copie de celles de F.Cu et In2 (rectangle (92,9 ; 53,2)-(103,3 ; 74,6), priorité 5), et 9 vias SENSE (0,6 / 0,3 mm) en (99,6 / 101,1 / 102,6 ; 59,6 / 61,1 / 73,6), juste à côté de la languette de drain de Q1. Jusqu'ici, les seuls vias SENSE étaient près du shunt. La chaleur du drain de Q1 passe maintenant par les trois couches F, In2 et B.
- **Point 41** (GPIO libres) : **TP31 à TP34** (D1,0 mm, face arrière, hors BOM) sur GPIO39 à 42, c'est-à-dire **le JTAG de l'ESP32-S3** (MTCK, MTDO, MTDI, MTMS). Ils servent de réserve et permettent de déboguer avec une sonde JTAG sans USB. Nets nommés `JTAG_MTCK`, `JTAG_MTDO`, `JTAG_MTDI`, `JTAG_MTMS` (feuille 04).
  - GPIO21 n'a pas de pastille : REAR_LED_EN passe à 0,2 mm au-dessus de sa broche et le module est en dessous, aucune sortie n'est possible.
  - GPIO45 reste libre et sans tirage (broche de configuration).
- **Point 40** (détection de coupure) : pas de nouveau diviseur sur GPIO3. Le cuivre PACK_RAW le plus proche est à environ 50 mm, de l'autre côté de la carte. Le diviseur existant VSW_ADC (PACK_SW, GPIO7) voit déjà la tension s'effondrer quand le pack disparaît, et c'est le seul cas où l'odomètre avance. C37 (100 µF) tient le buck 3,3 V pendant quelques millisecondes. **Le firmware doit** surveiller VSW_ADC (seuil et pente) et écrire l'odomètre dès qu'il voit la chute. Le préavis réel est à mesurer au banc [M].
- **Point 25** (terminaison CAN) : J7 reste ouvert. C'est noté dans `BOM.md` ; le cavalier y est signalé comme pièce à acheter à part.
- **Point 21** :
  - `BOM.md` corrigé (prix de Q1 / Q2 à 2,07 USD, cavalier de J7, rondelle DIN 433 pour la vis de H1) ;
  - **`BOM_BionX_Power_Control.csv` réexporté** depuis le schéma corrigé : fusibles Nano2, C28 tantale, C35 à C39, Q9, R58 / R59, empreintes du shunt, des EL357N et des LMR36510 ; D1, J4 et J5 en DNP.
- **Point 44, reprise** : les vias de TP27 (SW_3V3) et TP28 (SW_5V) touchaient les pastilles de L1 et L2. Ils sont déplacés en (102,9 ; 97,6) et (102,95 ; 112,47), sur la piste SW, hors de toute pastille.
- **Non traités, décision à prendre** :
  - 22 : diode série et TVS locale pour les bucks (composants de puissance à placer) ;
  - 23 : comparateur de surtension sur VSW (circuit nouveau) ;
  - 24 : PTC et TVS sur le 3,3 V et le 5 V exportés vers J10 ;
  - 26 : 5e trou de fixation, serre-câbles, guide de lumière (dépend du boîtier) ;
  - 28 : chien de garde externe ;
  - 35 : diviseur de 1 MΩ (gain d'environ 5 µA seulement).

**Reste à traiter après le lot 8** :
- point 36 : GPIO du CAN (firmware) ;
- point 16 : placement autour de U1 ;
- point 34 : modèles 3D (téléchargement à valider) ;
- points 3 et 4 : faute de place ;
- suggestions 22, 23, 24, 26, 28 et 35 : à décider.

### 2026-11-27 — lot 9 (points 36 et 16, suggestions 22 à 28 et 35)

Vérification après le lot : ERC 0, DRC 0 (règles BMS et HV comprises), 0 élément non connecté, 0 écart de parité.

- **Point 36, CAN** : filtre RC sur la broche STB du TCAN3413, **sans changer le brochage du firmware**.
  - Schéma, feuille 05 :
    - **R60** (10 kΩ 0402, C25744) en série entre l'étiquette CAN_STB (GPIO6, avec son tirage R28) et U7.8 ;
    - **C40** (100 nF 0402 16 V, CL05B104KO5NNNC, C1525) entre U7.8 (nouveau net `STB_RC`) et le 3,3 V.
  - Effet : τ = 1 ms. L'impulsion basse de 60 µs de GPIO6 au démarrage ne fait descendre STB qu'à environ 3,1 V, et le TCAN3413 reste en veille. Comme il ignore TXD en veille, l'impulsion de GPIO4 n'envoie plus de bits dominants sur le bus. Le condensateur est relié au 3,3 V et non à la masse : à la mise sous tension, STB suit le 3,3 V et démarre donc en veille.
  - **Firmware** : après avoir mis STB à l'état bas, attendre environ 3 ms avant d'émettre. Le conflit de 60 µs sur CAN_RX (GPIO5 contre RXD) reste, mais il est sans danger.
  - PCB : R60 à la verticale en (150,5 ; 80,3) et C40 en (149,1 ; 81,0), entre R28 et CANH. Le via CAN_STB est déplacé en (151,2 ; 80,2), hors des pistes B.Cu WAKE_RAW et PACK_5V_AUX.
  - Variante essayée puis abandonnée : passer CAN_TX, CAN_RX et CAN_STB sur GPIO35 à 37 (libres sur le module N8, sans impulsion). Les pastilles de ces GPIO sont dans une poche fermée entre J4 et le module : il fallait rerouter POWER_ENABLE, PRECHARGE_ON et MAIN_ON (environ 80 mm chacun), et le résultat était pire (vias en série, conflits d'isolement HV près de U1).
- **Point 16** :
  - **R61** (0 Ω 0402, C17168) en série sur la grille de Q1 (net `Q1_G`, ajouté à la règle HV), en (113,3 ; 71,88). Elle se trouve dans la boucle grille Q1 – grille Q2, ce qui suffit à l'amortir ; la remplacer par 2,2 à 10 Ω si on voit une oscillation au banc.
  - Pas de résistance sur Q2 : sa grille est sous le courtyard du boîtier TOLL, et la bande entre Q1 et Q2 est occupée par les vias de couture FET_SRC.
  - C10 reste en place (voir lot 7).
- **Suggestion 26** : trou de fixation **H5, M2,5** (perçage 2,7 mm), en (145,25 ; 94,5), entre U7 et J4, avec des zones `FIXATION_H5` sans cuivre (rayon 2,8 mm, F.Cu et B.Cu). C'est le seul disque libre de la moitié droite. Il ne passe pas en M3 : la tête de vis fait 5,5 mm, et il ne resterait que 0,05 mm de marge. Le boîtier doit prévoir un plot M2,5.
  - Restent hors carte : serre-câbles des XT60, guide de lumière de D4, accès à SW1, SW2 et l'USB.
- **Suggestion 22, écartée** : D2 (SMCJ48A, unidirectionnelle) est directement en travers du pack. Si le pack était branché à l'envers, elle conduirait en direct et le court-circuiterait avant toute autre chose. Une diode série sur les bucks ne protège donc de rien tant que D2 est là. C'est le détrompage du XT60 qui protège contre l'inversion. Quant aux transitoires, D2 et D3 les écrêtent déjà.
- **Suggestion 23, écartée** : le TPS48111 n'a pas d'entrée de surtension. Surtout, ouvrir Q1 / Q2 sur une surtension de VSW déconnecterait le pack, c'est-à-dire la seule charge capable d'absorber l'énergie, en pleine régénération du moteur. La tension côté moteur monterait encore plus vite, et D3 devrait tout absorber seule. La limite de régénération relève du contrôleur moteur et du firmware (lecture de VSW_ADC).
- **Suggestion 24, en attente** : il faut le courant actif de la carte Display sur J10.2 (3,3 V) et J10.3 (5 V) [M], absent de `POWER_BUDGET.md`. Une PTC ne protège pas : elle déclenche en secondes, alors que l'ESP32 maître décroche en millisecondes quand U3 entre en limitation. La bonne correction est un interrupteur à limitation de courant (famille TPS2553 / TPS25200), réglé d'après cette mesure. Le 5 V est déjà protégé par la limitation de courant de U5.
- **Suggestion 28, écartée** : quand l'ESP32 redémarre (chien de garde interne ou brown-out), ses broches passent en haute impédance, et R15 à R17 ramènent POWER_ENABLE, PRECHARGE_ON et MAIN_ON à l'état bas : Q1 et Q2 s'ouvrent. Un chien de garde externe n'ajouterait que le cas d'un ESP32 bloqué sans redémarrage, déjà couvert par le chien de garde interne.
- **Suggestion 35, écartée** : environ 5 µA de gain côté pack. Avec un diviseur de 1 MΩ, le nœud FB devient sensible au bruit, et TI recommande alors un condensateur d'avance de phase à dimensionner. Pas rentable.
- `BOM_BionX_Power_Control.csv` réexporté (R60, R61, C40) et `BOM.md` mis à jour.

**Reste à traiter après le lot 9** :
- point 34 : modèles 3D (téléchargement des STEP chez LCSC / EasyEDA à valider) ;
- suggestion 24 : mesure du courant du Display, puis choix de l'interrupteur limiteur ;
- points 3 et 4 (RC d'EN, C12) et C10 (point 16) : faute de place ;
- point 31 : exception de R46.2 (via en bord de pastille).

