# BionXHW — besoins et TODO matériel

Ce fichier regroupe les fonctions souhaitées et les décisions matérielles restant à prendre. Une fonction notée « à définir » n'est pas encore câblée ni validée.

## Fonctions demandées

- [ ] **Buzzer sur la carte de contrôle de puissance** (`pcb_power_control`) pour les avertissements sonores.
  - À définir : buzzer actif ou passif, tension, niveau sonore, commande (GPIO + transistor si nécessaire), motifs sonores et conditions d'activation.
- [ ] **Connecteur de feu arrière sur la carte de contrôle de puissance** (`pcb_power_control`), côté contrôleur arrière / faisceau arrière.
  - À définir : tension de sortie, courant nominal et de démarrage, feu alimenté en continu ou commuté, type et détrompage du connecteur, protections et brochage.
- [ ] **Connecteur de feu avant sur la carte avant / écran** (`pcb_display`), raccordé au faisceau avant.
  - À définir : tension de sortie, courant nominal et de démarrage, commande/variation éventuelle, type et détrompage du connecteur, protections et brochage.

## Intégration et vérifications à faire

- [ ] Reporter ces besoins dans les schémas KiCad des cartes concernées, avec les connecteurs et circuits de commande adaptés après définition des caractéristiques électriques.
- [ ] Vérifier que la commande des feux et du buzzer reste dans les limites des GPIO et utiliser un étage de puissance adapté si nécessaire.
- [ ] Dimensionner les pistes, connecteurs, câbles et protections selon le courant réel; vérifier les retours de masse et les transitoires.
- [ ] Confirmer le connecteur et le brochage avec le contrôleur arrière et le faisceau avant avant de figer le PCB.
- [ ] Valider au banc les tensions, courants, échauffements et comportements de mise sous tension avant tout raccordement au vélo.

## Paramètres connus du projet

- Batterie annoncée : pack 10S.
- Fusible annoncé : 30 A.
- Ces informations ne définissent pas la tension des sorties accessoires, leur courant admissible ni le brochage des faisceaux.
