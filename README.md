<div align="center">

<img src="docs/banniere.png" alt="Module Audio, lecteur embarqué ESP32" width="100%">

</div>

Module de lecture audio pour les circuits MicroCoaster™. Un ESP32 lit des pistes stockées sur carte microSD et les sort en I2S vers un amplificateur. La lecture est déclenchée à distance par le contrôleur, en WebSocket, pour synchroniser le son avec ce qui se passe sur le circuit : départ de train, passage en station, effet sonore d'une figure.

Comme tous les modules, il s'appuie sur `AyresWiFiManager` : au premier démarrage il ouvre un portail captif, on lui donne le réseau, puis il rejoint le serveur tout seul.

## Matériel

| Élément | Rôle |
|:--|:--|
| ESP32 DevKit | Microcontrôleur |
| Lecteur microSD | Stockage des pistes |
| DAC I2S + amplificateur | Sortie audio |
| Haut-parleur | Restitution |

## Bibliothèques

```ini
links2004/WebSockets        ; communication avec le contrôleur
bblanchon/ArduinoJson       ; messages échangés
esphome/ESP32-audioI2S      ; décodage et sortie audio
ayresnet/AyresWiFiManager   ; portail captif et reconnexion
```

Le système de fichiers embarqué est **LittleFS**, il héberge les pages du portail de configuration.

## Compiler et téléverser

Nécessite [PlatformIO](https://platformio.org/) dans Visual Studio Code.

```bash
pio run                  # compilation
pio run -t upload        # téléversement du firmware
pio run -t uploadfs      # téléversement du contenu de data/ vers LittleFS
pio device monitor       # console série, 115200 bauds
```

## Première mise en service

1. Alimenter le module. Il crée un point d'accès WiFi.
2. S'y connecter et ouvrir `http://192.168.4.1`.
3. Renseigner le réseau WiFi de destination.
4. Le module redémarre, rejoint le réseau et s'annonce auprès du serveur.

Les identifiants WiFi restent en mémoire du module, jamais dans le dépôt.

## Pistes audio

Les fichiers se placent à la racine de la carte microSD. Le contrôleur envoie le nom de la piste à jouer ; le module la cherche et la lit. Une piste absente est signalée au serveur plutôt que d'échouer en silence.

## État

Version `0.0.0`, module en développement. La lecture et la liaison WebSocket fonctionnent ; restent à traiter le réglage de volume à distance et la mise en file d'attente de plusieurs pistes.

---

<sub>MicroCoaster™ · Auteurs : CyberSpaceRS, Yamakajump</sub>
