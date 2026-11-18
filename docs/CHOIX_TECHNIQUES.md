# Dossier de justification des choix techniques — BionXHW

**État :** dossier de conception préliminaire, synchronisé avec les schémas consultés le 3 novembre 2026. Les pièces placées sont des choix de départ; aucune carte n'est encore validée pour fabrication ou raccordement au pack.

## Architecture et règles de décision

Le projet comprend deux cartes séparées : `pcb_display` (interface utilisateur, ESP32 et CAN) et `pcb_power_control` (commutation/précharge, mesure courant/tension et interfaces accessoires). La logique de contrôle doit rester en basse tension; le trajet du pack 10S et son courant doivent rester sur une zone de puissance dimensionnée séparément.

Le pack est annoncé comme 10S avec un fusible de 30 A. Sa chimie, sa tension maximale réelle, les courants continus/crête du contrôleur, ses capacités d'entrée et les caractéristiques des lumières restent à confirmer. La valeur « 42 V max » n'est valable que pour un pack lithium-ion conventionnel chargé à 4,2 V par cellule.

## Composants placés ou retenus comme base

### ESP32-S3-WROOM-1-N8 — U1, carte écran

- **Fonction :** MCU de l'interface; gère l'écran, les boutons, les échanges de données et le contrôleur CAN intégré TWAI.
- **Pourquoi ce module :** le TWAI prend en charge le CAN classique (trames 11/29 bits, jusqu'à 1 Mbit/s); il évite le MCP2515 et son bus SPI sur cette nouvelle carte. Le module inclut le quartz, 8 MB de flash et l'antenne PCB, ce qui réduit le nombre de composants RF à placer.
- **Limites :** ce n'est pas un transceiver CAN; il faut un PHY externe. Le module a besoin d'une alimentation 3,0–3,6 V correctement dimensionnée, d'un traitement EN/straps/USB adapté et d'une zone sans cuivre sous l'antenne. Le Wi-Fi/Bluetooth devront être arrêtés en veille profonde. Le choix d'un ESP32-S3 est une nouvelle architecture de carte; le firmware historique HUZZAH32 + MCP2515 devra être porté ou validé séparément.
- **Veille :** Espressif donne typiquement 7–8 µA au niveau du module en deep-sleep dans les modes RTC correspondants; ce chiffre ne comprend pas le régulateur, les pull-ups, l'écran ni les autres composants.
- **Approvisionnement :** ESP32-S3-WROOM-1-N8, JLCPCB **C2913198**, pièce répertoriée comme *Extended* au contrôle du 2026-11-03. Recontrôler stock et statut PCBA avant commande.
- **Sources :** [datasheet module Espressif](https://documentation.espressif.com/esp32-s3-wroom-1_wroom-1u_datasheet_en.pdf), [fiche JLCPCB C2913198](https://jlcpcb.com/partdetail/Espressif-ESP32_S3_WROOM_1/C2913198).

### SN65HVD230QDR — U2, carte écran — choix actuel provisoire

- **Fonction :** couche physique CAN entre TX/RX du TWAI et CANH/CANL.
- **Pourquoi il a été retenu :** alimentation 3,3 V compatible directement avec le domaine logique ESP32-S3; CAN conforme à ISO 11898-2; débit jusqu'à 1 Mbit/s, suffisant pour CAN classique; boîtier SOIC-8 simple à assembler et à inspecter. C'est un choix sobre pour le premier prototype.
- **Limite de veille importante :** son mode `RS` haut est une écoute seule, récepteur toujours actif; courant typique 370 µA, maximum 600 µA. Il permet de voir le bus et de réveiller le MCU, mais empêche à lui seul une veille de quelques dizaines de microampères.
- **Approvisionnement :** JLCPCB **C16468**, *Extended*, relevé le 2026-11-03.
- **Décision à revoir avant routage :** si le réveil par CAN doit fonctionner tout en respectant le budget de veille, évaluer le **TI TCAN3413DR** (JLCPCB/LCSC **C22433320**, *Extended*). Son mode veille écoute un motif de réveil CAN; la fiche TI donne jusqu'à 2 µA sur VCC et 13 µA sur VIO à 85 °C (15 µA combinés), avec VCC commutable. Il fonctionne avec le CAN classique et accepte des E/S 1,7–3,6 V. Le brochage/les rails diffèrent du SN65HVD230 : ce n'est pas un remplacement à faire sans modifier et vérifier le schéma. Si le réveil CAN n'est pas nécessaire, une veille profonde avec le PHY coupé peut être étudiée, sous réserve de maîtriser les courants de retour par TXD/RXD et l'état des lignes bus.
- **Sources :** [datasheet TI SN65HVD230](https://www.ti.com/lit/ds/symlink/sn65hvd230.pdf), [fiche JLCPCB C16468](https://jlcpcb.com/parts/2nd/Interface/CAN_Transceivers_3093), [datasheet TCAN3413](https://www.ti.com/lit/ds/symlink/tcan3413.pdf), [fiche JLCPCB TCAN3413DR](https://jlcpcb.com/partdetail/TexasInstruments-TCAN3413DR/C22433320).

### LM5069MM-1/NOPB — ancien choix, retiré du schéma

- **Fonction :** contrôleur hot-swap haute tension; pilote un MOSFET série externe pour contrôler l'appel de courant/précharge, régler limite de courant et puissance, surveiller les défauts et se verrouiller après défaut avec la variante `-1`.
- **Pourquoi il a été retenu :** plage VIN de 9–80 V, donc marge par rapport aux 42 V d'un pack 10S Li-ion classique; seuil de limitation défini par une résistance de mesure; contrôle du MOSFET externe pour limiter l'inrush vers le contrôleur BionX. Il centralise le démarrage contrôlé et une partie des protections.
- **Limites :** ce composant ne constitue pas à lui seul une protection complète. Q1 doit être sélectionné selon VDS, SOA pendant toute la rampe d'appel de courant, RDS(on), pertes et refroidissement; les seuils/timer, UVLO/OVLO et capacités d'entrée doivent être calculés. Le circuit ne remplace ni le fusible ni la validation thermique.
- **Courant de veille critique :** VIN reste directement alimenté par le pack même si UVLO désactive le contrôleur : 480 µA typiques, 650 µA maximum sur VIN, hors diviseurs UVLO/OVLO et autres fuites. La broche UVLO peut désactiver la commutation, mais ne transforme pas le circuit en composant à courant nul.
- **Approvisionnement :** JLCPCB **C3177598** (*Extended*, fiche JLC consultée le 2026-11-03); correspondance LCSC **C486026** observée. Vérifier le marquage et la variante `LM5069MM-1/NOPB` lors du choix BOM; ne pas confondre avec la variante `-2` à redémarrage automatique.
- **Sources :** [datasheet TI LM5069](https://www.ti.com/lit/ds/symlink/lm5069.pdf), [fiche JLCPCB C3177598](https://jlcpcb.com/partdetail/TexasInstruments-LM5069MM1/C3177598), [fiche LCSC C486026](https://www.lcsc.com/product-detail/New-Arrivals_Texas-Instruments-Texas-Instruments-LM5069MM-1-NOPB_C486026.html).

### INA228AIDGSR — U2, carte contrôle puissance

- **Fonction :** mesure numérique bidirectionnelle du courant sur le shunt, de la tension de bus et de la température; transmet les mesures sur I²C et peut déclencher ALERT.
- **Pourquoi il a été retenu :** entrée de mesure haute tension jusqu'à 85 V, alimentation 2,7–5,5 V, interface I²C et grande résolution. Il convient à un pack 10S même si la chimie exacte reste à confirmer. Le symbole est actuellement représenté avec la base de brochage INA226, tandis que la valeur/MPN indiquent l'INA228; confirmer le symbole, le footprint VSSOP-10 et le land pattern TI dans KiCad avant le layout.
- **Courant :** 640 µA typiques en mesure active; arrêt par registre `SHUTDOWN` jusqu'à 5 µA maximum. L'arrêt logiciel suppose que le rail 3V3 reste présent; un load-switch peut couper son alimentation, avec validation des fuites par les broches I²C/ALERT.
- **Approvisionnement :** JLCPCB/LCSC **C2887910**, *Extended*, VSSOP-10-0,5 mm, relevé le 2026-11-03.
- **Sources :** [datasheet TI INA228](https://www.ti.com/lit/ds/symlink/ina228.pdf), [fiche JLCPCB C2887910](https://jlcpcb.com/partdetail/TexasInstruments-INA228AIDGSR/C2887910).

### R1 — shunt Kelvin 4 bornes — nouvelle base 1 mΩ avec TPS48111

- **Fonction :** shunt commun de télémétrie INA228 et de protection TPS48111. Les bornes 1/4 portent le courant; 2/3 sont les prises Kelvin. Valeur de départ 1 mΩ, puissance visée ≥3 W avec déclassement à vérifier.
- **Pertes :** à 30 A, `P = 30² × 0,001 = 0,9 W`; chute de tension 30 mV. La plage ±40,96 mV de l'INA228 permet environ ±40,96 A à ce gain; pour observer les pointes de court-circuit, employer la plage ±163,84 mV et calibrer Rshunt.
- **Limites :** MPN, TCR, tolérance, courant admissible et empreinte restent à sélectionner. Aucun stock de shunt n'est prétendu vérifié.

### U3 — LMR36510ADDAR — conversion pack vers 3V3, réseau externe câblé

- **Fonction :** abaisser le pack 10S vers un rail logique 3V3 pour la carte écran et le moniteur INA228.
- **Pourquoi ce candidat :** plage d'entrée 4,2–65 V, donc compatible avec un pack Li-ion 10S conventionnel (42 V chargé) avec marge; sortie 1 A; mode PFM et courant de repos typique d'environ 26 µA adaptés à une veille sobre. Cette valeur seule n'est pas le courant système et ne garantit pas l'objectif de veille.
- **État du schéma :** U3 reste la référence initiale LMR36510ADDAR. C1 = 2,2 µF / 100 V et C2 = 220 nF / 100 V sur VIN; C3 = 1 µF / 16 V sur VCC; C4 = 100 nF / 16 V entre BOOT et SW; L1 = 22 µH (saturation visée ≥2 A, MPN/empreinte à choisir); C5–C7 = 3 × 22 µF / 25 V en sortie; R2 = 46,4 kΩ et R3 = 20 kΩ, 1 %. Le calcul donne `Vout = 1 V × (1 + 46,4/20) = 3,32 V`. Ces valeurs viennent du réseau d'application TI 400 kHz / 3,3 V. EN est câblé sur VIN; PG est volontairement inutilisé. Les rails utilisent des symboles KiCad de puissance nommés PACK_RAW / PACK_SW / PWR_3V3 et des symboles GND, avec PWR_FLAG aux sources identifiées.
- **Qualification restante :** capacité effective sous tension, courant de saturation et pertes de L1, transitoires pack, câblage inversé, rendement à faible charge et PCB thermique. Les références fabricant/LCSC des passifs sont encore à sélectionner; le schéma n'est pas une BOM prête à assembler. C8 = 100 nF découple l'INA228; R4–R6 = 4,7 kΩ tirent SCL/SDA/ALERT au rail logique. Ces trois lignes doivent être hautes en veille.
- **Approvisionnement :** LCSC **C1858393** (LMR36510ADDAR); statut de stock et classe JLC à vérifier à la date de commande.
- **Sources :** [datasheet TI LMR36510](https://www.ti.com/lit/ds/symlink/lmr36510.pdf), [fiche LCSC C1858393](https://lcsc.com/product-detail/DC-DC-Converters_Texas-Instruments-LMR36510ADDAR_C1858393.html).

### Q1 — MOSFET série de puissance — non sélectionné

- **Fonction prévue :** interrupteur externe piloté par le LM5069; sa rampe de conduction réalise le démarrage contrôlé/précharge du contrôleur aval.
- **Pourquoi aucun MPN n'est encore défendable :** il manque la capacité d'entrée du contrôleur, les limites de tension/transitoires, le temps de précharge, le courant d'appel, le profil 30 A, l'aire de cuivre et le refroidissement. Choisir uniquement sur RDS(on) ou courant drain nominal serait insuffisant; il faut valider l'aire de fonctionnement sûre (SOA) durant l'appel.
- **Approvisionnement :** à choisir avec le boîtier/empreinte et le statut JLC/LCSC après calcul; le symbole `Q_NMOS_GDS` présent est un placeholder, pas un composant sélectionné.

### J1 — liaison I²C / mesure vers la carte écran — interface provisoire

- **Fonction :** transporte 3V3, GND, SCL, SDA et ALERT de la carte écran vers l'INA228 sur la carte puissance.
- **Pourquoi :** l'I²C évite une seconde liaison CAN pour une seule mesure locale; la ligne ALERT peut demander une mesure ou signaler un seuil.
- **Approvisionnement / limites :** connecteur générique 1×5 sans MPN ni footprint. Type, verrouillage, brochage mécanique, pull-ups, longueur/parasites et protection ESD restent à définir. Vérifier les pull-ups I²C sur un seul emplacement et ne pas mettre en parallèle des valeurs trop faibles.

## Protocole BMS DALY : choix à confirmer sur la référence réelle

La documentation DALY décrit CAN, UART/485 et Modbus; elle donne 9600 bit/s pour UART/485 et indique que l'accès varie selon la série de BMS. L'option privilégiée pour éviter un second CAN est le port UART si le modèle réel le fournit. « UART/485 » peut désigner une interface TTL ou une couche électrique RS-485 : ne jamais raccorder directement les fils BMS au GPIO ESP32 sans modèle, pinout, tension et couche physique confirmés. Si le port est RS-485, ajouter un transceiver approprié; si TTL-UART, vérifier le niveau logique, le sens TX/RX, la masse et les conditions d'isolation.

- **Source :** [protocoles DALY](https://www.dalybms.com/news/daly-three-communication-protocols-explanation/).

## Fonctions accessoires demandées, composants non choisis

Le fichier `TODO.md` suit le buzzer sur la carte puissance, le feu arrière côté contrôleur arrière et le feu avant raccordé au faisceau avant. Aucun courant/MPN n'est estimé tant que leurs tensions et appels de courant ne sont pas connus. En veille, ils devront être réellement coupés par un étage de puissance ou démontrer un courant résiduel négligeable; l'état « GPIO bas » seul n'est pas une preuve de zéro consommation.

## Décisions avant layout

1. Fixer la chimie/tension maximale du pack et les exigences de courant/démarrage du contrôleur BionX.
2. Décider si le réveil par CAN est exigé. Si oui, comparer le SN65HVD230 avec le TCAN3413 et mettre à jour schéma, brochage et BOM ensemble.
3. Fixer un objectif chiffré de courant en veille mesuré **à l'entrée pack**, puis choisir la stratégie de coupure qui le permet.
4. Choisir Q1, le shunt et les composants de seuil/timer après calcul SOA et thermique.
5. Fixer les rails 3V3/5V, convertisseur, courants de feux/buzzer, interfaces DALY et connecteurs.
6. Recontrôler les références JLC/LCSC en stock et le classement Basic/Extended au moment de la BOM.


## U1 — TPS48111LQDGXRQ1 — remplacement appliqué après accord de Théo

Le LM5069 est retiré et remplacé dans `Power_Switch.kicad_sch`, sous-feuille de la carte puissance. Le symbole projet `BionX:TPS48111LQDGXRQ1` reprend les 19 broches de la table 5-1 TI Rev E; la position 16 est absente. L'empreinte DGX19 reste à dessiner/qualifier : aucune empreinte 20 broches approximative n'a été attribuée.

- **Motif :** 3,5–80 V en fonctionnement (100 V absolu), pilote de précharge indépendant G/INP_G, pilotage de MOSFETs dos à dos, seuils surintensité et court-circuit réglables, variante L à verrouillage. Le shutdown EN bas consomme 1,6 µA typiques, 5,36 µA max de −40 à 125 °C (2,65 µA max jusqu'à 85 °C), aux conditions de la datasheet. Il évite les 480 µA typiques du LM5069 désactivé.
- **Commande :** J4 expose POWER_ENABLE, PRECHARGE_ON, MAIN_ON, deux défauts et GND. R12–R14 = 1 kΩ série; R15–R17 = 100 kΩ vers GND pour l'arrêt par défaut. Aucune résistance ne tire EN depuis le pack. Le BMS assure les limites de cellules; le TPS48111 n'a pas l'entrée OV de la variante TPS48110. Un défaut de firmware ou de BMS reste à traiter dans la stratégie système.
- **Alimentation :** R8 = 100 Ω et C9 = 100 nF/100 V filtrent VS. R9 = 100 Ω définit RSET sur CS+. C10 = 1 µF/25 V entre BST et SRC; référence SRC commune aux trois sources de MOSFET. Il faut confirmer la marge de charge de grille aux tolérances/températures.
- **Réglages préliminaires :** R11 = 39,2 kΩ donne `IOC ≈ 11,9×100/(39 200×0,001) ≈ 30,36 A`; R10 = 2,74 kΩ donne `ISC ≈ (2740+464)×15,6 µA/1 mΩ ≈ 50 A`; C11 = 68 nF donne environ 1 ms de temporisation de surcharge selon l'exemple TI. Ces seuils nominaux ne sont pas validés comme coordination du fusible 30 A; intégrer tolérances, TCR, SOA et courbes du fusible avant fabrication.
- **Diagnostics :** IMON est relié à GND conformément au cas inutilisé TI, car l'INA228 assure la télémétrie; DIODE est à GND et désactive la surveillance thermique externe. FLT_I/T ont chacun 100 kΩ vers 3V3; ils doivent être lus et leur état en veille mesuré. Le modèle ERC utilise IMON passif (sortie de courant autorisée à GND), PU/PD/G trois états et SRC entrée de référence : cela évite de déclarer à tort un court-circuit entre deux sorties de tension.
- **Approvisionnement :** [LCSC C17558994](https://www.lcsc.com/product-detail/C17558994.html); présence au catalogue vérifiée, stock et service PCBA JLC à recontrôler. [Datasheet TI](https://www.ti.com/lit/ds/symlink/tps4811-q1.pdf).

## Q1/Q2/Q3 — CSD19536KTT — base de MOSFET 100 V

Q1 et Q2 ont leurs sources communes et leurs grilles principales pilotées par PU/PD via R7 = 10 Ω à l'allumage; ils bloquent les deux sens lorsqu'ils sont arrêtés. À l'état passant ils autorisent les deux sens : la gestion de la régénération BionX/BMS ne découle pas automatiquement de ce montage. Q3 est le MOSFET de précharge et partage la référence SRC; il contourne Q1 via R18. Pendant la précharge, le courant rejoint la sortie par la diode intrinsèque de Q2. Cette diode doit tenir le profil de précharge.

Le CSD19536KTT est une référence TI active 100 V, VGS ±20 V, RDS(on) max 2,4 mΩ à 10 V et Qg typique 118 nC. Les pertes nominales de la paire à 30 A sont `2×30²×2,4 mΩ = 4,32 W` à la résistance indiquée; elles augmentent à chaud. Il faut donc du cuivre/refroidissement adaptés et vérifier SOA et énergie d'avalanche. R19/R20 = 1 MΩ ramènent les grilles à SRC sans courant pack permanent idéal à l'arrêt. Empreinte de départ TO-263-2 à vérifier contre le plan KTT, avec drain sur le tab, grille 1 et source 3.

Sources : [TI CSD19536KTT](https://www.ti.com/product/CSD19536KTT), [LCSC C2687963](https://www.lcsc.com/product-detail/MOSFETs_Texas-Instruments_C2687963.html). Les stocks et l'éligibilité JLC ne sont pas garantis pour la commande.

## R18 — précharge résistive, dimensionnement final en attente de Ccharge

R18 = 100 Ω est une valeur de travail, sans empreinte ni puissance définitive. À 42 V : courant initial 0,42 A, puissance initiale 17,64 W. `E = Ccharge × Vpack² / 2`, `t99% ≈ 4,6 × R × Ccharge`. Par exemple pour 1 mF : 0,882 J et environ 460 ms, en l'absence de charge active. Ce n'est pas une capacité mesurée du contrôleur. La résistance doit être choisie sur sa courbe de surcharge impulsionnelle et sa répétition; une simple résistance CMS 100 Ω ordinaire n'est pas acceptable.

La séquence firmware devra autoriser EN, précharger, vérifier réellement la montée de PACK_SW avec l'INA228 et un timeout, activer MAIN puis arrêter PRECHARGE. Sur timeout/défaut, arrêter les commandes sans répétition illimitée. Le firmware correspondant n'est pas implémenté ici.

## Points ouverts avant PCB / pack

Capacité réelle et tension minimale/chimie du pack, coordination fusible, régime de régénération, dimensionnement thermique/SOA, choix des protections TVS/diode et amortissement des câbles (la tenue 65 V du buck est la contrainte), connecteurs 30 A, MPN du shunt et des passifs, empreinte DGX19, qualification des bibliothèques et test réel de veille. Aucun ERC ne valide ces propriétés physiques.

## Électronique locale du PCB Power Control

La carte puissance et la carte écran ont chacune leur microcontrôleur. U4 est le module ESP32-S3-WROOM-1-N8 avec flash 8 Mo, quartz et antenne intégrés. Le TWAI du module fonctionne en CAN classique via U7. Prévoir la zone sans cuivre de l'antenne au routage. J9 assure la programmation USB native. J5 expose TX, RX, EN, BOOT et GND pour un adaptateur UART à logique 3,3 V. Le module reçoit le 3,3 V sélectionné entre U3 et la branche USB. J1 et J4 servent à la mise au point locale. Ne pas piloter J4 depuis un second contrôleur en parallèle avec U4.

| Fonction | GPIO U4 |
|---|---:|
| CAN TX / RX / STB | 4 / 5 / 6 |
| INA228 SDA / SCL / ALERT | 7 / 8 / 9 |
| TPS48111 EN / précharge / principal | 10 / 11 / 12 |
| 12 V enable / feu arrière / buzzer | 13 / 14 / 15 |
| TPS48111 FLT_I / FLT_T | 17 / 18 |
| UART0 TX / RX | 43 / 44 |

U7 TCAN3413DR reçoit 3,3 V sur VCC et VIO. STB haut au reset sélectionne la veille. Le signal RXD arrive sur GPIO5, compatible avec la logique RTC pour le réveil par motif CAN, à implémenter et vérifier sur le bus BionX. J6 porte CANH, CANL et GND. Installer le cavalier J7 seulement si la carte est en bout de bus. D1 PESD2CANFD24V-T protège les deux lignes contre les décharges. Le débit et la terminaison finale dépendent du faisceau réel.

| Bloc | Alimentation | Dimensionnement |
|---|---|---|
| U3, 3,3 V permanent | Pack brut | Alimente U4, U7 et INA228. Budget de courant de pointe du module à vérifier avec les appels radio éventuels. |
| U5, 12 V accessoires | Pack brut | LMR36510ADDAR, 47 µH, 3 × 22 µF, diviseur 100 kΩ / 9,09 kΩ selon TI. GPIO13 avec rappel bas commande EN. |
| U6, source 5 V côté pack | 12 V commuté | L78M05ABDT-TR. À 30 mA, environ 0,21 W de dissipation de charge, plus la consommation propre du régulateur. |
| Q4, feu arrière | 12 V commuté | AO3400A en commutation côté masse. J8 pin 1 = 12 V, pin 2 = retour commuté. Charge supposée être un feu 12 V à courant régulé de 200 mA, soit 2,4 W. À 48 mΩ, perte MOSFET d'environ 1,9 mW hors commutation. Vérifier les pointes au branchement et la tension au drain avec le faisceau réel. |
| Q5, buzzer | 5 V sélectionné entre USB et pack | AO3400A pour buzzer actif piézo 5 V, consommation de conception ≤30 mA. Référence et empreinte du buzzer à sélectionner selon la pression sonore et l'encombrement. |

U4 et U7 sont listés LCSC C2913198 et C22433320. U6 est listé C58069 et l'AO3400A C20917. Les références des inductances, du buzzer et des connecteurs restent à sélectionner. Les réglages d'assemblage JLC sont à vérifier sur la BOM finale.

Sources : [Espressif ESP32-S3-WROOM](https://documentation.espressif.com/esp32-s3-wroom-1_wroom-1u_datasheet_en.pdf), [TI TCAN3413](https://www.ti.com/lit/ds/symlink/tcan3413.pdf), [TI LMR36510](https://www.ti.com/lit/ds/symlink/lmr36510.pdf), [ST L78M](https://www.st.com/en/power-management/l78m.html), [Nexperia PESD2CANFD24V-T](https://assets.nexperia.com/documents/data-sheet/PESD2CANFD24V-T.pdf), [AOS AO3400A](https://www.aosmd.com/products/mosfets/low-voltage-mosfets-12v-30v/ao3400a).


## USB-C de programmation et alimentation

J9 utilise le port USB natif de l’ESP32-S3 (GPIO19 D−, GPIO20 D+). Les deux broches CC ont chacune leur rappel de 5,1 kΩ vers GND. U8 protège les données contre les décharges. R36/R37 valent 22 Ω. SW1 et SW2 donnent accès à BOOT et RESET. Le routage USB doit suivre les règles Espressif pour la paire différentielle.

U9 AP2112K-3.3 fournit un 3,3 V à partir de VBUS. U10 TPS2116 sélectionne entre ce 3,3 V et le buck du pack. U11 TPS2116 sélectionne entre VBUS et le 5 V auxiliaire fourni par U6 pour alimenter le buzzer actif BZ1. Les sélecteurs sont câblés en mode de sélection de la tension la plus haute avec blocage du courant inverse. Les sources peuvent basculer lorsque leurs tensions se croisent. Vérifier la chute du rail lors d’un branchement ou débranchement USB sous charge.

Sur USB seul, le module, le CAN, l’INA228 et le buzzer sont alimentables. Le feu arrière dépend toujours du 12 V du pack. Le port est câblé comme périphérique USB 5 V. Vérifier la consommation à l’attachement, puis pendant la programmation avec le buzzer actif, selon le courant annoncé par le port hôte.

La vue d’ensemble A4 organise la carte par fonctions, selon la structure du [schéma Softcar de référence](https://github.com/SOFTCARCLEAN/SoftcarHardwareVolant/tree/250203_SIO_TPI). [Datasheet TI TPS2116](https://www.ti.com/lit/ds/symlink/tps2116.pdf) · [Guide USB Espressif](https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/esp32s3/schematic-checklist.html)
