# TODO — mise en production des PCB BionXHW

Objectif : passer de deux schémas brouillon et deux PCB vides à un dossier de commande JLCPCB (≈5 cartes, coût minimal, BOM simple). Règle de lecture : **[F]** décision figée, **[H]** hypothèse à valider, **[M]** à mesurer, **[D]** décision ouverte (valeur par défaut indiquée).

## Décisions figées (à jour du 2026-11-16)

- [F] Moteur BionX 350 W, **36 V** (confirmé par Théo ; relire la plaque pour la référence exacte). Pack **10S3P**, cellules **EVE INR18650-33V** (3100 mAh min, 10 A cont. par cellule, 33 cellules commandées chez NKON), fusible 30 A. Pointe moteur max : **30 A**.
- [F] BMS : **DALY Smart 10S Li-ion 40 A, port commun, UART** (modèle exact à confirmer chez le vendeur, voir B5). Le BMS BionX d'origine est mort et retiré.
- [F] Architecture : **Power Control** (près du pack) = ESP32-S3 maître BionX, précharge/commutation (TPS48111), shunt + mesure, UART vers le DALY, feu arrière, buzzer, **frein**, **feu avant** (sortie). **Display** = interface seulement (ESP32-S3, écran rond, encodeur).
- [F] Liaison Display ↔ Power : **le CAN du faisceau existant** (même bus que le moteur). Le Display n'émet que ses propres identifiants, jamais de trame BionX. Seul Power Control parle au moteur. Pas d'ESP-NOW.
- [F] Réveil : clic encodeur → ESP Display → trame CAN → TCAN3413 de Power Control en veille. **Pas de fil de wake**, mais un pin libre sur le connecteur Display en secours.
- [F] Frein : contact tout-ou-rien, normalement fermé (fil coupé = frein actif), **câblé sur le Display** (décision du 2026-11-16).
- [F] Gâchette conservée, Hall 5 V, **câblée sur le Display** (décision du 2026-11-16). Le 5 V vient du faisceau, qui passe à 6 conducteurs : GND, 3,3 V, 5 V commuté, CANH, CANL, WAKE secours. Détail dans `PHASE2_SCHEMAS.md`.
- [F] Précharge : TPS48111 + MOSFET de précharge + résistance (principe actuel). Seuil de surintensité à remonter (voir P2.4).
- [D] INA228 : par défaut **conservé** (mesure indépendante du DALY, comptage de charge). Premier composant à supprimer si on simplifie.
- [F] Production : le moins de cartes possible, budget minimal. La carte Display doit être **la plus petite possible**.

## Phase 1 — Banc, avant toute fabrication (aucune dépendance au PCB)

- [ ] **B1 [M]** Relever la plaque du moteur. Mesurer, hors tension, la terminaison CAN (≈60 Ω si deux 120 Ω) et la tension de la ligne WAKE sur le faisceau d'origine. *Fin : valeurs consignées dans HARDWARE.md.*
- [ ] **B2 [M]** Capacité d'entrée du contrôleur moteur : charger via une résistance connue (ex. 100 Ω) depuis une alim à limite de courant et relever τ = R·C à l'oscilloscope. *Fin : C mesurée. À défaut, hypothèse 0,5–2 mF.*
- [ ] **B3 [M]** Rejouer les captures de démarrage : lister les messages BionX indispensables, la séquence réveil / init / maintien / arrêt, les registres batterie lus par le moteur, et si une **émulation minimale du nœud batterie** (ID 0x10 et réponses sur 0x08) est nécessaire. Fixer les limites charge/décharge du nouveau pack **à partir de la fiche EVE, sans copier les anciennes valeurs** : charge ≈ 9,3 A max (1C), décharge à déraler par rapport aux 30 A du pack. *Fin : tableau des messages et limites validé au banc.*
- [ ] **B4 [M]** Test du réveil CAN : ESP32-S3 devkit + TCAN3413 + bus à 125 kbit/s. Critère : 20 réveils sur 20, sur un bus avec le moteur non alimenté branché. **Si échec : le pin de wake devient obligatoire dans le connecteur.**
- [ ] **B5 [M]** DALY : lire tension, courant signé, SOC, cellules, état des MOSFET. Vérifier le **niveau logique du port UART**, la **masse de référence (B− ou P−)** et le courant de repos réel du BMS. *Fin : décision isolation / pas d'isolation, schéma d'interface UART.*
- [ ] **B6 [M]** Gâchette : type (Hall ratiométrique 5 V ?), plage de tension, compatibilité 3,3 V. Frein : confirmer le contact NC/NO.
- [ ] **B7 [M]** Régénération vs coupure de charge du BMS : mesurer la montée de la tension du bus quand le DALY coupe la charge pendant une régénération (roue levée, puissance limitée). *Fin : dimensionnement de la TVS / de l'écrêtage.*

## Phase 2 — Schémas

### Power Control
- [x] P2.1 (fait, frein et gâchette déplacés sur le Display) Ajouter : entrée **frein** (NC vers GND, rappel, filtre RC, ESD), entrée **gâchette** (5 V ou 3,3 V selon B6, diviseur et écrêtage vers l'ADC), sortie **feu avant** (copie du canal de Q4), **interface UART DALY**, connecteur de faisceau unique (GND, 3,3 V, CANH, CANL, pin libre + frein, gâchette, feu avant).
- [x] P2.2 (SMCJ43A posées, à confirmer par B7) **TVS / écrêtage à l'entrée pack** : choisir selon B7 et la tension absolue du LMR36510 (vérifier la datasheet).
- [x] P2.3 (U11 retiré, U9 + U10 gardés pour programmer sans pack) Revoir les sélecteurs TPS2116 (U10/U11) et le chemin USB : garder uniquement ce qui sert la programmation et le buzzer.
- [x] P2.4 (R11 = 31,6 kΩ → 37,7 A ; R11 était court-circuitée) **Seuil de surintensité du TPS48111 : remonter IOC à 35–40 A** (aujourd'hui ≈30,4 A, il déclencherait aux pointes du moteur). Recalculer R10/R11 et C11, laisser le fusible 30 A comme dernière protection.
- [x] P2.5 (2 × 220 Ω 2512 en parallèle sur l'hypothèse 0,5–2 mF ; à revoir après B2) Dimensionner la précharge sur la capacité de B2 : R18, énergie ½·C·V² (≈1,8 J à 2 mF et 42 V), puissance impulsionnelle (17,6 W à 100 Ω), SOA de Q3.
- [x] P2.6 (conservé) Décider INA228 conservé ou supprimé (défaut : conservé).

### Display
- [x] P2.7 Retirer : J108 (feu avant), AO3400A, J221 si la gâchette passe sur Power Control, 12 V / 5 V, U9 AP2112K, U10 TPS2116 et l'alimentation par USB. Garder : ESP32-S3, TCAN3413 + TVS CAN, écran rond, encodeur, bouton, TPS22918 si utile, pastilles de programmation (ou USB-C données seules).
- [x] P2.8 (6 points avec le 5 V) Connecteur faisceau minimal : GND, 3,3 V, CANH, CANL, pin libre. Vérifier la chute de tension (≈40 mV pour 150 mA sur 1,5 m de 24 AWG).
- [x] P2.9 Terminaison : cavalier ouvert par défaut sur les deux cartes. Ne rien ajouter avant B1.

### Commun
- [x] P2.10 (tables dans `PHASE2_SCHEMAS.md`) Réattribuer les GPIO, vérifier les broches de straps/USB de l'ESP32-S3 et la cohérence avec le futur BSP firmware.
- [x] P2.11 (0 violation le 2026-11-16) ERC à zéro sur les deux projets (`kicad-cli sch erc`) après chaque modification.

## Phase 3 — Bibliothèques, empreintes, BOM

- [x] P3.1 (empreinte faite d'après le plan TI 4226944/A ; symbole vérifié contre la table 5-1) **Empreinte DGX19** du TPS48111 (dessin TI) et symbole vérifié broche par broche.
- [x] P3.2 (voir `BOM.md`) Choisir et référencer : **shunt Kelvin 1 mΩ ≥3 W**, résistance de précharge **tenue en impulsion**, inductances (22 µH, 47 µH, Isat ≥2 A), buzzer, connecteurs 30 A pack/contrôleur, porte-fusible, TVS, boutons, connecteurs faisceau Display.
- [ ] P3.3 Vérifier les empreintes déjà posées : CSD19536KTT (TO-263), INA228 (VSSOP-10), LMR36510, USB-C, ESP32-S3-WROOM-1.
- [x] P3.4 (voir `BOM.md` ; stock d'assemblage JLCPCB à recontrôler au devis) Pour chaque composant : MPN fabricant, référence LCSC, prix, stock, **Basic/Extended**, alternative. Viser le moins de pièces *Extended* (frais d'assemblage par référence).
- [x] P3.5 (`docs/BOM_*.csv`, aucune ligne sans empreinte) Exporter la BOM (`kicad-cli sch export bom`) et vérifier qu'aucune ligne n'est sans empreinte.

## Phase 4 — Layout Power Control (pas de contrainte de taille)

- [x] P4.1 (112 × 82 mm, 4 couches ; épaisseur de cuivre à décider en phase 6) Contour, trous de fixation, boîtier ; **empilement 4 couches** à décider ; cuivre épais pour le chemin 30 A.
- [x] P4.2 (voir `PCB_POWER_CONTROL.md`) Chemin de puissance : largeur de piste ou polygone pour 30 A, via thermiques, dissipation de Q1/Q2 (≈4,3 W à 30 A d'après ta doc), distances d'isolement pour 42 V.
- [x] P4.3 (DRC à 0 ; relecture manuelle recommandée, voir `PCB_POWER_CONTROL.md`) **Routage Kelvin du shunt**, zone sans cuivre sous l'antenne du module, paire USB 90 Ω, paire CAN, découplages au plus près.
- [x] P4.4 (25 pastilles, table dans `PCB_POWER_CONTROL.md` ; plus sonde de température J13 et LED d'état D4) Points de test : pack, PACK_SW, 3V3, 5 V, GND, CANH/L, UART, TX/RX ESP.

## Phase 5 — Layout Display (la plus petite possible)

- [ ] P5.1 Mesurer le module d'écran réel (diamètre, nappe, hauteur) et fixer le contour de la carte.
- [ ] P5.2 Placer module ESP32-S3 (antenne en bord, zone dégagée), TCAN3413, encodeur, connecteur faisceau. Viser 2 couches si possible.

## Phase 6 — Vérification, coût, commande

- [ ] P6.1 DRC à zéro, revue 3D, vérification des polarités et du sens des composants (rotation JLC).
- [ ] P6.2 Générer Gerbers, perçages, BOM, CPL (`kicad-cli`) et passer la **DFM JLCPCB**.
- [ ] P6.3 **Coût livré en Suisse** pour ≈5 cartes : fabrication, assemblage, pièces *Extended*, transport, TVA et frais de dédouanement. Comparer 1 carte vs 2 cartes (Display seul, Power seul).
- [ ] P6.4 Mettre à jour `CHOIX_TECHNIQUES.md`, `POWER_BUDGET.md`, `DISPLAY_SCHEMA.md` et `README_KICAD_STARTER.md` (anciennes références au DALY UART, SN65HVD230 et LM5069 à retirer ; budget de veille recalculé avec le courant de repos réel du DALY).
- [ ] P6.5 **Initialiser git** pour le dossier matériel et poser un tag avant commande. Aucun achat ni commande sans accord explicite.

## Phase 7 — Réception et mise en route

- [ ] P7.1 Inspection visuelle, test de continuité, aucun court-circuit pack/GND.
- [ ] P7.2 Première mise sous tension sur alimentation à limite de courant, **sans pack ni moteur** ; vérifier 3,3 V, 5 V, CAN, UART, LED d'état, lecture de la sonde (débranchée = pleine échelle).
- [ ] P7.3 Mesurer la veille réelle au pack, avec et sans DALY.
- [ ] P7.4 Test précharge et commutation sur charge résistive, puis avec le moteur roue levée, puissance limitée, arrêt physique indépendant.
- [ ] P7.5 Valider frein, gâchette, perte de CAN et timeouts avant tout essai roulant.

## Points restants de la revue Power Control (2026-11-17)

- [ ] **R1 [D] Polarité inverse à l'entrée pack.** Défaut : rien à ajouter, car le XT60 est détrompé.
  - [C] Q1/Q2 sont montés dos à dos (sources communes) : la sortie moteur est protégée.
  - [H] Tout ce qui est sur `PACK_RAW` voit l'inversion :
    - D2 conduit en direct et sert de court-circuit, si bien que le fusible 30 A du pack fond ;
    - les LMR36510 (U3, U5) supportent au plus −0,3 V à l'entrée et risquent d'être détruits ;
    - les feux sont aussi exposés.
  - Pour une vraie protection, il faudrait un MOSFET P ou un contrôleur de diode idéale sur l'alimentation des convertisseurs (quelques dizaines de mA) : à décider.
- [ ] **R2 [M] Le courant de régénération n'est mesuré que par le DALY.** IMON (TPS48111) ne voit que la décharge.
  - B5 doit valider la cadence et la résolution du courant signé du DALY.
  - Le firmware doit prévoir un repli si l'UART du DALY se tait : limite de régénération à zéro ou faible.
- [ ] **R3 [D] Pas de mesure de `PACK_RAW` à l'ADC.** Avant la fermeture des MOSFET, la tension pack ne vient que du DALY. Défaut : pas d'ajout. Si besoin, ajouter un diviseur 150 kΩ / 10 kΩ sur GPIO3, l'ADC1 restant libre (0 µA perdu si ramené derrière un MOSFET, sinon ≈ 0,26 mA en permanence).
- [x] **R4 [C] Rappels au reset.** Vérifié dans la netlist le 2026-11-17 : toutes les sorties de puissance sont tirées à l'état bas pendant le démarrage de l'ESP32.
  - R15, R16 et R17 (100 kΩ) sur EN, INP_G et INP du TPS48111 ;
  - R19 et R20 (1 MΩ) sur les grilles de Q1/Q2 et de Q3 ;
  - R31, R41 et R33 sur les grilles des feux et du buzzer ;
  - R23 sur `ACCESSORY_EN`.
- [ ] **R5 [M] Sonde de température.** Relever R25 et B de la CTN achetée (hypothèse 10 kΩ, B3950) et coder la table. Fixer en firmware les seuils d'alerte et de coupure selon l'endroit où la sonde est collée (MOSFET, pack ou moteur).
- [ ] **R6 Stock de C14663** (100 nF 0603 Basic, C34) à recontrôler dans le devis JLCPCB.
- [ ] **R7 Faces des points de test.** 11 points de test sur 25 sont en face arrière. À revoir si la carte doit se sonder montée dans son boîtier.

## Points de sécurité à ne pas oublier

- La régénération dépasse ≈9,3 A (1C) si les limites ne sont pas envoyées au moteur.
- Un CAN à zéro ne garantit pas l'arrêt si le bus est perdu : prévoir timeouts et coupure physique indépendante testés.
- Le DALY est la protection de dernier recours, pas l'ESP.
- Boot neutre : toutes les sorties de puissance à l'arrêt tant que le firmware n'a pas validé les préconditions.
