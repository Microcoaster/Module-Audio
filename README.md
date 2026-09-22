<div align="center">

<p>
  <img src="docs/langues/fr-on.png" alt="Français, page affichée" width="150" />
  <a href="README.en.md"><img src="docs/langues/en-off.png" alt="Read this page in English" width="150" /></a>
</p>

<img src="docs/banniere.png" alt="Module Audio, lecteur embarqué ESP32" width="100%">

</div>

Module de lecture audio pour les circuits MicroCoaster. Un ESP32 lit des pistes stockées sur carte microSD et les sort en I2S vers un amplificateur. La lecture est déclenchée à distance par le contrôleur, en WebSocket, pour synchroniser le son avec ce qui se passe sur le circuit : départ de train, passage en station, effet sonore d'une figure.

Comme les autres modules, il se configure au premier démarrage par portail captif, puis rejoint le serveur.

**Version 0.0.0**

<img src="docs/sections/s01.png" alt="01 Principe" width="100%">

Le module ne connaît pas le circuit. Il reçoit un nom de piste, la cherche sur la carte, la joue. C'est le contrôleur qui sait à quel moment du parcours un son doit partir.

<img src="docs/schemas/principe.png" alt="Ordre reçu : le contrôleur envoie le nom de la piste à jouer. Piste trouvée : décodage et sortie I2S vers le DAC puis l'amplificateur. Piste absente : signalée au serveur, un son muet et un module perdu se ressemblent trop pour qu'on les confonde." width="100%">

Les fichiers se placent à la racine de la carte microSD.

<img src="docs/sections/s02.png" alt="02 Matériel" width="100%">

Deux bus cohabitent sur la carte. L'I2S porte le son vers le DAC, le SPI va chercher les fichiers sur la carte microSD. Les mélanger sur les mêmes broches est la première cause de lecture hachée.

<img src="docs/schemas/brochage.png" alt="Sortie audio sur bus I2S : GPIO 26 BCLK horloge de bit, GPIO 25 LRC sélection de voie, GPIO 22 DIN échantillons vers le DAC, GPIO 2 LED de statut lecture en cours. Stockage sur bus SPI : GPIO 13 SD CS sélection du lecteur, GPIO 23 MOSI données vers la carte, GPIO 19 MISO données depuis la carte, GPIO 18 SCK horloge du bus." width="100%">

La sortie I2S reste numérique jusqu'au DAC, ce qui évite de faire courir un signal analogique faible à côté des lignes de puissance des autres modules.

<img src="docs/sections/s03.png" alt="03 Protocole" width="100%">

Le module s'authentifie à la connexion, puis échange en JSON, comme les autres modules du circuit.

```json
{
  "type": "module_identify",
  "moduleId": "MC-0001-AU",
  "moduleType": "audio",
  "uptime": 12345
}
```

Le contrôleur envoie le nom de la piste à jouer, le module répond une fois la lecture engagée, ou signale que le fichier est introuvable.

<img src="docs/sections/s04.png" alt="04 Mise en service" width="100%">

Copiez d'abord [`include/env.h.example`](include/env.h.example) en `include/env.h` et renseignez-le : identifiants du portail de secours, identité du module et son secret. Ce fichier n'est pas versionné, et sans lui le firmware ne compile pas.

Nécessite [PlatformIO](https://platformio.org/) dans Visual Studio Code.

```bash
pio run                  # compilation
pio run -t upload        # téléversement du firmware
pio run -t uploadfs      # téléversement du contenu de data/ vers LittleFS
pio device monitor       # console série, 115200 bauds
```

1. Alimenter le module. Il crée un point d'accès WiFi.
2. S'y connecter et ouvrir `http://192.168.4.1`.
3. Renseigner le réseau WiFi de destination.
4. Le module redémarre, rejoint le réseau et s'annonce auprès du serveur.

Les identifiants WiFi restent en mémoire du module, jamais dans le dépôt.

<img src="docs/sections/s05.png" alt="05 Écosystème" width="100%">

```ini
links2004/WebSockets        ; liaison avec le contrôleur
bblanchon/ArduinoJson       ; messages échangés
esphome/ESP32-audioI2S      ; décodage et sortie audio
ayresnet/AyresWiFiManager   ; portail captif et reconnexion
```

Module en développement : la lecture et la liaison WebSocket fonctionnent, restent à traiter le réglage de volume à distance et la mise en file d'attente de plusieurs pistes.

Système de fichiers embarqué : **LittleFS**, il héberge les pages du portail. Le socle commun à tous les modules est le [WiFi Manager](https://github.com/Microcoaster/MicroCoaster_WifiManager), et le pilotage se fait depuis la [WebApp](https://github.com/Microcoaster/MicroCoasterWebApp).

---

<sub>MicroCoaster · Auteur : Cybertrist</sub>
