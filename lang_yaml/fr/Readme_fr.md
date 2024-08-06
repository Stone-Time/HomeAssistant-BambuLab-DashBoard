![Aperçu](https://github.com/Stone-Time/HomeAssistant-BambuLab-DashBoard/blob/main/lang_yaml/fr/screenshot.jpg?raw=true)

# 1. Composants HACS nécessaires
- Mushroom
- card-mod
- Stack In Card
- Bambu Lab
- Google Dark Theme

# 2. Importer l'imprimante dans les appareils BambuLab
- Configuration du Cloud Bambu, pas LAN !
- Europe
- Se connecter directement à l'imprimante au lieu de passer par le Cloud Bambu <--- activer

# 3. Importer l'imprimante dans les appareils BambuLab
-> Paramètres -> Appareils & Services -> Intégration

-> Créer une caméra générique

| Option                            | Paramètre                                      |
| ----------------------------------| -----------------------------------------------|
| URL de l'image                    | (laisser vide)                                 |
| URL de la source du flux          | rtsps://<printer-ip>/streaming/live/1          |
| Protocole RTSP                    | TCP                                            |
| Authentification                  | basic                                          |
| Nom d'utilisateur                 | bblp                                           |
| Mot de passe                      | <lan-access-code>                              |
| Fréquence d'images                | 30                                             |
| Vérifier le certificat SSL | désactiver                                            |

# 4. Transférer le contenu du dossier www
Soit via SambaShare ou File editor un par un 
Le dossier www devrait exister, les autres doivent être créés

# 5. Déterminer les données à remplacer dans le tableau de bord

| Imprimante                   | EXEMPLES              |
| ------------------------- | ----------------------- |
| <BAMBULABPRINTER_ID> 			|	x1c_00m00a000000000     |
| <BAMBULABPRINTER_AMS_ID> 	|	x1c_00m00a000000000_ams |

| Interrupteur d'alimentation               | EXEMPLES                                    |
| --------------------------- | --------------------------------------------- |
| <POWER_SWITCH_ID>					  | switch.powerprinterrack1_2                    | 
| <POWER_POWERMETER_ID>				| sensor.powerprinterrack1_2_power              | 
| <POWER_VOLTAGEMETER_ID>			| sensor.powerprinterrack1_2_voltage            |
| <POWER_CURRENTMETER_ID>			| sensor.powerprinterrack1_2_current            |
| <POWER_DAYENERGYMETER_ID>		| sensor.powerprinterrack1_2_total_daily_energy | 
| <POWER_TOTALENERGYMETER_ID> | sensor.powerprinterrack1_2_energy             | 

| Caméra    | EXEMPLES        |
| --------- | ----------------- |
| <CAM_ID>  | camera.10_10_5_91 |

# 6. Créer des compteurs comme assistants 
-> Paramètres -> Appareils & Services -> Assistants
-> Créer un assistant -> Compteur

| Champ           | Valeur              |
| -------------- | ----------------- |
| Nom           | printer_bambulab_<BAMBULABPRINTER_ID>_maintenance_tiges_carbone |
| Icône           | mdi:wrench-clock  |
| Valeur minimale     | 0 |
| Valeur de départ    | 0 |
| Taille de l'étape   | 1 |

| Champ           | Valeur              |
| --------------  | ----------------- |
| Nom             | printer_bambulab_<BAMBULABPRINTER_ID>_maintenance_tiges_yz |
| Icône           | mdi:wrench-clock  |
| Valeur minimale     | 0 |
| Valeur de départ    | 0 |
| Taille de l'étape   | 1 |

| Champ           | Valeur              |
| --------------  | ----------------- |
| Nom             | printer_bambulab_<BAMBULABPRINTER_ID>_maintenance_vis_z |
| Icône           | mdi:wrench-clock  |
| Valeur minimale     | 0 |
| Valeur de départ    | 0 |
| Taille de l'étape   | 1 |

| Champ           | Valeur              |
| --------------  | ----------------- |
| Nom             | printer_bambulab_<BAMBULABPRINTER_ID>_maintenance_appareil_principal |
| Icône           | mdi:wrench-clock  |
| Valeur minimale     | 0 |
| Valeur de départ    | 0 |
| Taille de l'étape   | 1 |

# 7. Créer une automatisation
Paramètres -> Automatisations & Scènes

-> Créer une automatisation -> Nouvelle automatisation

-> En haut à droite, 3 points -> Modifier en YAML

```yaml
alias: printer_bambulab_<BAMBULABPRINTER_ID>_maintenance_count
description: ""
trigger:
  - platform: state
    entity_id:
      - sensor.<BAMBULABPRINTER_ID>_gesamtnutzung
    alias: printer_bambulab_<BAMBULABPRINTER_ID>_maintenance_count
condition:
  - condition: template
    value_template: |
      {% set t_new = trigger.to_state.state|default(0)|int %}
      {% set t_from = trigger.from_state.state|default(0)|int %}
      {{ t_new >= t_from }}
action:
  - service: counter.increment
    metadata: {}
    data: {}
    target:
      entity_id:
        - counter.printer_bambulab_<BAMBULABPRINTER_ID>_maintenance_tiges_carbone
        - counter.printer_bambulab_<BAMBULABPRINTER_ID>_maintenance_appareil_principal
        - counter.printer_bambulab_<BAMBULABPRINTER_ID>_maintenance_tiges_yz
        - counter.printer_bambulab_<BAMBULABPRINTER_ID>_maintenance_vis_z
mode: single
```

-> Sauvegarder -> Renommer -> 

# 8. Créer la page et remplacer toutes les <IDs> dans le YAML

Modèle YAML de la page selon ce qui est présent dans les entités Bambu

Si le "Slot actif" a pour ID d'entité "active_tray" à la fin -> en_yaml

Si le "Slot actif" a pour ID d'entité "aktiver_slot" à la fin -> de_yaml

# 9. Traduction du type de Buse

Allez dans la configuration de votre fichier "configuration.yaml" sur Home Assistant.

Ajoutez ces lignes dans le fichier :

```yaml
sensor:
  - platform: template
    sensors:
      <BAMBULABPRINTER_ID>_nozzle_type_fr:
        value_template: >
          {% set nozzle_type = states('sensor.<BAMBULABPRINTER_ID>_nozzle_type') %}
          {% if nozzle_type == 'hardened_steel' %}
            Acier trempé
          {% elif nozzle_type == 'stainless_steel' %}
            Acier inoxydable
          {% else %}
            {{ nozzle_type }}
          {% endif %}
```

Remplacez `<BAMBULABPRINTER_ID>` par l'identifiant de votre imprimante Bambu.

Enregistrez le fichier.

Redémarrez Home Assistant.

Si tout fonctionne correctement, vous aurez une nouvelle entrée nommée "sensor.<BAMBULABPRINTER_ID>_nozzle_type_fr".

Il vous suffira de remplacer, dans le fichier fr_yaml.txt, `sensor.<BAMBULABPRINTER_ID>_nozzle_type` par `sensor.<BAMBULABPRINTER_ID>_nozzle_type_fr`.

Copiez et collez le YAML modifié dans la configuration YAML de votre page d'accueil pour votre imprimante.