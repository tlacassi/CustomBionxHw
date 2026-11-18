# Display PCB

Le module ESP32-S3-WROOM-1-N8 et le TCAN3413DR reprennent les références de Power Control. J109 est un USB-C de programmation natif avec BOOT et RESET. Le sélecteur TPS2116 alimente le 3,3 V local depuis le faisceau ou depuis le régulateur USB. Les références Display commencent à 100 pour distinguer les cartes dans le projet d’ensemble.

| Connecteur | Fonction et brochage |
|---|---|
| J101 | 1 = 3,3 V permanent, 2 = 12 V commuté, 3 = GND. Correspondance directe avec J10 de Power Control. |
| J106 | 1 = CANH, 2 = CANL, 3 = GND. Correspondance avec J6 de Power Control. Fermer le cavalier J107 seulement en extrémité du bus. |
| J210 | Encodeur A, B, clic, masse. Clic actif bas sur GPIO7. |
| J211 | Bouton externe actif bas sur GPIO8 et masse. |
| J212 | 3,3 V UI, signal et masse. Contact normalement fermé vers GND ou Hall 3,3 V à sortie compatible. Une entrée ouverte demande la limitation ou l’inhibition. |
| J220 | 3,3 V UI, GND, SPI SCK/MOSI/CS/DC/RST, BL, I²C SDA/SCL, INT/RST tactile. |
| J108 | Phare avant 12 V régulé, pin 1 positif et pin 2 retour commuté par AO3400A. |
| J221 | Gâchette optionnelle, 5 V issu du 12 V, signal analogique et masse. |

L’écran de référence est un module rond GC9A01 + CST816S en SPI/I²C. J220 définit un faisceau interne au projet. L’ordre des fils et la fonction de BL sont à confirmer pour le module réel. U120 coupe le 3,3 V UI en veille. Les rappels I²C et des voies A/B de l’encodeur sont sur ce rail. Le firmware devra mettre les GPIO de l’écran au niveau bas ou en haute impédance avant la coupure pour éviter une alimentation par les signaux. [Module de référence Waveshare](https://www.waveshare.com/1.28inch-touch-lcd.htm) · [TI TPS22918](https://www.ti.com/lit/pdf/slvsd76)

L’entrée limite est prévue pour une demande de coupure ou de limitation traitée par firmware. Une coupure de fil amène l’entrée à l’état haut. Un court-circuit à la masse peut imiter un contact fermé. Les conditions de validation, le seuil de vitesse et la stratégie en cas de perte du CAN restent à définir. La gâchette dispose d’un diviseur 18 kΩ / 22 kΩ et de 100 nF. 5 V donnent 2,75 V à l’ADC. L’entrée est réservée à un capteur compatible. Calibrer sa plage et traiter les valeurs invalides, le démarrage avec gâchette actionnée et la perte de signal avant d’autoriser une commande de couple.

## Réveil prévu

1. Le Display laisse GPIO7 et GPIO8 en réveil EXT1 sur niveau bas. Les deux rappels sont sur le 3,3 V permanent.
2. Power Control garde son module alimenté et son TCAN3413 en standby avec STB haut. RXD arrive sur GPIO5, utilisable comme entrée RTC. Les sorties de puissance restent à l’arrêt pendant le sommeil.
3. Après le réveil du Display, celui-ci passe son transceiver en mode normal et émet une séquence CAN dont le motif satisfait le WUP du TCAN3413 au débit réel du bus. Sur WUP, RXD réveille Power Control. La première trame peut ne pas être reçue par le contrôleur CAN Power Control.
4. Après le démarrage des deux contrôleurs, le Display envoie une demande applicative et attend une réponse explicite. Définir les identifiants, la fenêtre de réponse et le nombre maximal de tentatives dans le firmware. L’échange de réveil laisse les commandes de traction désactivées jusqu’à validation des préconditions.

Le CAN réutilise le faisceau existant. Vérifier le WUP et l’échange au banc, notamment si aucun nœud réveillé n’acquitte la première émission. Une ligne de réveil dédiée reste une option si le bus ne permet pas un réveil suffisamment fiable. [Espressif EXT1](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-reference/system/sleep_modes.html) · [TI TCAN3413](https://www.ti.com/lit/ds/symlink/tcan3413.pdf)

Le faisceau 3,3 V est une hypothèse de conception. Le buck permanent de Power Control fournit les deux modules en service sur pack. Vérifier le courant de pointe cumulé et la chute de tension au connecteur Display avec la longueur de câble réelle, écran allumé. L’USB du Display permet la programmation locale et l’essai de l’écran sans pack. Le phare et la gâchette exigent le 12 V du faisceau. Les références définitives de l’écran, du phare et des capteurs restent à confirmer.
