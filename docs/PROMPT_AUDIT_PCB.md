# Prompt — audit du PCB Power Control

Copie tout ce qui suit dans une nouvelle conversation ouverte dans `BionxHw/bionxHW`.

---

## Rôle et objectif

Tu es relecteur senior de cartes électroniques : puissance, CEM, fabrication JLCPCB. Fais un **audit critique du PCB Power Control** avant commande. Cherche les erreurs, les risques et ce qui manque, puis rends-moi une **liste détaillée et priorisée**.

**Audit en lecture seule.** Ne modifie aucun fichier du projet (schémas, PCB, docs). Aucun commit, aucun achat, aucune commande. Ne touche ni aux `.history/` ni aux fichiers `~*.lck` : KiCad est peut-être ouvert. Les scripts et exports temporaires vont dans le scratchpad.

## Le projet en bref

- **Vélo** : BionX custom, moteur 350 W 36 V avec contrôleur intégré, CAN propriétaire à 125 kbit/s. Pack 10S3P (EVE INR18650-33V), fusible 30 A, pointe moteur 30 A. BMS **DALY Smart 10S 40 A** en UART.
- **Deux cartes KiCad 10** :
  - **Power Control** (`pcb_power_control/`) : c'est la carte à auditer ;
  - **Display** (`pcb_display/`) : PCB pas encore dessiné.
- **Architecture figée** (à ne pas remettre en cause sans le signaler) :
  - ESP32-S3 maître BionX, transceiver TCAN3413 ;
  - commutation et précharge par TPS48111 avec deux MOSFET TOLL dos à dos (HYG017N10NS1TA), MOSFET de précharge 5N10 et résistances 2 × 220 Ω en 2512 ;
  - shunt Kelvin de 1 mΩ ; la mesure de courant passe par la broche IMON du TPS48111 (l'INA a été supprimé) ;
  - buck LMR36510 3,3 V permanent et buck 5 V commuté ;
  - feux avant et arrière en tension pack, commutés côté masse ;
  - buzzer ; UART DALY isolé par optocoupleurs EL357N ;
  - USB-C pour la programmation, avec alimentation simple AMS1117 et deux MOSFET ;
  - sonde de température CTN externe sur J13, LED d'état D4, 25 points de test.
- **Faisceau Display (J10, 6 points)** : GND, 3,3 V, 5 V commuté, CANH, CANL, WAKE de secours. J10 a été déplacé le 2026-11-18 au bord supérieur, près de J6 (CAN moteur) et de U7.
- **Carte** : 112 × 82 mm, 4 couches.
  - F.Cu : signaux et polygones de puissance ;
  - In1 : masse pleine ;
  - In2 : copies des polygones de puissance et masse ;
  - B.Cu : signaux et masse.
- **Classes de nets** : signaux 0,2/0,2 mm ; alimentations 0,3 mm ; puissance 0,6 mm avec 0,3 mm d'isolement.
- **Coordonnées** : la carte va de (60, 50) à (172, 132) en coordonnées absolues KiCad. Les docs utilisent souvent des coordonnées relatives au coin haut gauche ; précise toujours lesquelles tu emploies.

## À lire d'abord

1. `docs/TODO_PRODUCTION.md` : phases, décisions, mesures de banc B1–B7, **points restants R1–R7**.
2. `docs/PCB_POWER_CONTROL.md` : choix de layout et historique des passes, dont la table des points de test et les sections des 2026-11-17 et 18.
3. `docs/PHASE2_SCHEMAS.md` : calculs (seuils du TPS48111, précharge, TVS), GPIO, faisceau.
4. `docs/BOM.md` et `docs/BOM_BionX_Power_Control.csv`.
5. Les schémas `pcb_power_control/*.kicad_sch` et le PCB `pcb_power_control/BionX_Power_Control.kicad_pcb`.
6. Le firmware de référence (clone local) : `/Users/theolacassin/Documents/GitRepos/bionx_console`, en particulier `docs/HARDWARE.md` et `docs/BENCH_FINDINGS.md`.

Les points R1 à R7 et les mesures B1 à B7 sont déjà connus. Ne les recopie pas comme des découvertes : dis seulement si ton analyse les confirme, les aggrave ou les infirme.

## Outils disponibles (macOS)

- `kicad-cli` : `/Applications/KiCad/KiCad.app/Contents/MacOS/kicad-cli`. Utilise `pcb drc --severity-all --schematic-parity`, `sch erc`, les exports SVG, Gerber et position.
- **Python avec pcbnew** : `/Applications/KiCad/KiCad.app/Contents/Frameworks/Python.framework/Versions/3.9/bin/python3`.
  - Piège SWIG : après un `board.Remove()`, les objets du même processus deviennent invalides. Fais un traitement par processus.
  - Piège SWIG : copie `GetStart()` et `GetEnd()` dans des tuples avant de modifier quoi que ce soit.
- **poppler** (`pdftotext`, `pdftoppm`) pour lire les datasheets.
- **Rendu** : export SVG via kicad-cli, puis `qlmanage -t -s 1800` pour obtenir un PNG.
- **Recherche de pièces** : API `jlcsearch.tscircuit.com` et fiche produit LCSC (`wmsc.lcsc.com/ftps/wm/product/detail?productCode=Cxxxx`).

## Ce qu'il faut auditer

Vérifie toi-même plutôt que de croire les docs.

1. **Cohérence schéma ↔ PCB ↔ BOM.**
   - Lance ERC, DRC et parité.
   - Vérifie que chaque composant a son empreinte, son MPN et son LCSC.
   - Vérifie les brochages critiques contre les datasheets :
     - TPS48111 en DGX19 (empreinte maison) ;
     - MOSFET TOLL, 5N10, AMS1117, EL357N, TCAN3413, LMR36510 ;
     - XT60, USB-C, module ESP32-S3-WROOM-1.
   - Vérifie l'orientation des diodes et des TVS, ainsi que la polarité des condensateurs et de la LED.
2. **Chemin 30 A.** Section de cuivre par couche (en supposant 35 µm, puis 70 µm), nombre de vias entre couches, échauffement, goulots d'étranglement, Kelvin du shunt, symétrie des deux MOSFET, dissipation et surface de cuivre des TOLL.
3. **Protections.**
   - TVS : tension d'écrêtage face aux 70 V maximum du LMR36510.
   - Polarité inverse (R1).
   - Régénération et coupure par le DALY.
   - Précharge (énergie, aire de sécurité du MOSFET, timeout firmware).
   - ESD sur tous les connecteurs qui sortent de la carte : CAN, faisceau, feux, sonde, UART DALY, USB.
   - Fusibles des feux.
4. **Isolements.** Distances pour 42 V et plus, isolement réel entre le côté BMS et le côté ESP32 autour des optocoupleurs (cuivre et plans sous les EL357N), distances aux trous de fixation.
5. **CEM et plans.**
   - Plan de référence sous chaque piste rapide ou sensible.
   - Boucles chaudes des deux bucks : condensateur d'entrée, SW, inductance, retour de masse.
   - Nœuds SW près des signaux sensibles.
   - Retour de la paire CAN.
   - Antenne de l'ESP32 : zone sans cuivre sur toutes les couches, dégagement au bord.
   - Morceaux de masse isolés ; densité et utilité des vias de couture.
6. **Signaux.** USB D+/D− (impédance, symétrie, longueur), CAN (terminaison J7/R29 ouverte par défaut, stub vers U7), ADC (IMON, VSW, sonde : filtrage, référence de masse), straps de l'ESP32 (GPIO0, 3, 45, 46), boutons BOOT et RESET.
7. **Alimentations.** Découplages au plus près, chute de tension sur le 3,3 V et le 5 V du faisceau, comportement USB seul, pack seul et les deux ensemble, courant de veille (le 3,3 V est permanent sur le pack).
8. **Fabrication JLCPCB.** Règles minimales (piste, isolement, perçage, anneau, bord), sérigraphie sur pastilles, références lisibles, repères de polarité, rotations CPL (empreintes maison), points de test, pièces Extended, assemblage simple face, composants traversants à souder à la main.
9. **Mécanique.** Trous de fixation, position des connecteurs et sortie des câbles (J10 est maintenant au bord haut, à côté du XT60 moteur), hauteurs, accès aux boutons et à l'USB, LED visible.
10. **Ce qui manque.** Toute fonction, protection, mesure ou marquage utile qui n'existe pas encore : sérigraphie des connecteurs, version de carte, logo de tension, etc.

## Format de rendu attendu

Une liste triée par gravité : **Bloquant** (à corriger avant commande), **Important**, **Mineur**, **Suggestion**. Pour chaque point :

- **Titre court** et **catégorie** (puissance, CEM, fabrication, etc.) ;
- **Où** : référence de composant, net, couche et coordonnées en précisant absolues ou relatives ;
- **Constat** avec la preuve : sortie de commande, calcul, extrait de datasheet avec numéro de page, capture ;
- **Statut** : **[C] confirmé**, **[H] hypothèse** ou **[M] à mesurer** ;
- **Risque concret** si on ne fait rien ;
- **Correction proposée** et son coût : pièces, place, effet sur la BOM ou le routage.

Termine par :
- un tableau de synthèse avec le nombre de points par gravité ;
- les vérifications que tu n'as pas pu faire, et pourquoi ;
- la liste des points R1–R7 et B1–B7 que ton analyse modifie.

N'invente jamais une valeur mesurable : capacité du moteur, tensions du faisceau, niveaux du DALY. Si une conclusion en dépend, classe-la en [M].
