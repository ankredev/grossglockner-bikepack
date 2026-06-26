# MountainBike — Paper-Plugin

Ein serverseitiges **Paper-Plugin** für den Großglockner-Minecraft-Server: ein craftbares
**Mountainbike**, mit dem man sich schnell fortbewegt — bergauf über Stufen, **bergab mit
Turbo**, dazu Sprung-Boost, einen aufladbaren Turbo mit Anzeige, Fahrgeräusch und Klingel.

Da es ein **Plugin** ist (kein Fabric/Forge-Mod), müssen Mitspieler **nichts installieren** —
außer dem automatischen Resource Pack für das Bike-Aussehen (Pflicht-Download beim Joinen).

## Steuerung

| Aktion | Taste |
|---|---|
| Fahrmodus an/aus | **Rechtsklick** mit dem Bike |
| Klingel 🔔 | **Linksklick** oder **F** |
| Springen (höher & weiter) | **Space** (nur während der Fahrt) |
| Turbo an/aus | **Doppel-Space** |

> Hinweis: Frei wählbare Tasten (z. B. „H") sind serverseitig **nicht** abfangbar — Minecraft
> sendet nur Klicks, Springen, Schleichen, Sprinten, Item-Wechsel und F an den Server.

## Mechanik
- **Tempo:** erhöhte Laufgeschwindigkeit; klettert kleine Stufen/Hänge automatisch (Step-Height).
- **Bergab-Turbo (passiv):** beim Abwärtsfahren am Boden zusätzlicher Schub — je steiler, desto
  schneller (gedeckelt). Wirkt nur am Boden, nicht in der Luft.
- **Sprung:** höher und weiter, aber nur wenn man tatsächlich fährt (Stand-Sprung bleibt normal).
- **Turbo (Doppel-Space):** kräftiger Schub in Blickrichtung. **Verbrauchsabhängig** — eine
  Energieleiste (Bossbar) leert sich nur bei aktivem Turbo (voll → leer in 3 s) und lädt sonst
  wieder auf (leer → voll in 10 s). Kurz antippen = wenig Verbrauch. Leer → Auto-Aus.
  Technik: Doppel-Space wird über das Abfangen des Flug-Toggles erkannt (funktioniert in
  **Survival/Adventure**; im Kreativmodus löst Doppel-Space echtes Fliegen aus → dort kein Turbo).
- **Kein Fallschaden** im Fahrmodus.
- **Sound:** Freilauf-„Klick" beim Fahren (Tempo passend zur Geschwindigkeit, lauter im Turbo),
  Klick beim Sprung, Wind-Whoosh beim Turbo-Start, „Ding" wenn die Energie wieder voll ist,
  Glocke bei der Klingel.

## Bike bekommen
- **Befehl:** `/bike` (gibt das Item ins Inventar)
- **Crafting** (3×3):
  ```
  [    ][Stock][    ]
  [Eisen][Leder][Eisen]
  [Eisen][    ][Eisen]
  ```
  = 1 Stock, 4 Eisenbarren, 1 Leder

## Resource Pack (Aussehen)
Das Bike ist technisch ein `carrot_on_a_stick` mit `CustomModelData = 1`. Ein Resource Pack
ersetzt das Aussehen durch ein Mountainbike-Sprite.
- Pack-Quelle: `../bikepack/` (Textur + `build_pack.py`)
- Gehostet: GitHub-Release **ankredev/grossglockner-bikepack** (v1)
- Eingebunden in `papermc2/server.properties` via `resource-pack` + `resource-pack-sha1`,
  `require-resource-pack=true` (Pflicht-Download)

## Build
Gebaut wird **auf dem DietPi** (dort liegen JDK 25 und das passende `paper-api`):
```bash
# auf dem Pi, im Projektordner /root/bikeplugin
export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java))))
/tmp/apache-maven-3.9.9/bin/mvn -q -DskipTests package
# -> target/MountainBike.jar
```
- **Java:** JDK 25 (Paper 26.1.2 verlangt min. Java 25)
- **Abhängigkeit:** `io.papermc.paper:paper-api:26.1.2.build.7-alpha` (scope `provided`),
  Repo `https://repo.papermc.io/repository/maven-public/`
- **Bytecode-Ziel:** Java 21 (`maven.compiler.release` in `pom.xml`)

## Deploy
```bash
cp target/MountainBike.jar /mnt/dietpi_userdata/papermc2/plugins/
chown papermc:papermc /mnt/dietpi_userdata/papermc2/plugins/MountainBike.jar
systemctl restart papermc2
```
Server: **Paper 26.1.2**, Dienst `papermc2.service`, Port 25565
(öffentlich `kremsers.synology.me:19191`).

## Tuning
Alle Werte als Konstanten oben in `src/main/java/at/kremser/bike/MountainBike.java`:

| Konstante | Bedeutung | Standard |
|---|---|---|
| `RIDE_WALK_SPEED` | Laufgeschwindigkeit im Fahrmodus | 0.42 |
| `STEP_UP` | Stufenhöhe, die hochgefahren wird | 1.1 |
| `DOWNHILL_FACTOR` / `DOWNHILL_MAX_BOOST` / `SPEED_CAP` | Bergab-Turbo | 2.6 / 0.9 / 2.4 |
| `JUMP_UP_BONUS` / `JUMP_FORWARD` | Sprung höher / weiter | 0.18 / 0.45 |
| `DRAIN_PER_TICK` / `RECHARGE_PER_TICK` | Turbo leeren (3 s) / laden (10 s) | 1/60 / 1/200 |
| `TURBO_FORWARD` / `TURBO_CAP` | Turbo-Schub / Topspeed | 0.55 / 3.4 |
| `MIN_START` | Mindestenergie zum Turbo-Start | 0.15 |
| `CLICK_BLOCKS` | ein Klick je x Blöcke | 0.6 |

## Verwendete API (26.1.2, registry-basiert)
- `Attribute.MOVEMENT_SPEED`, `Attribute.STEP_HEIGHT`
- `Sound.BLOCK_TRIPWIRE_CLICK_ON` (Freilauf), `ENTITY_WIND_CHARGE_WIND_BURST` (Turbo),
  `BLOCK_NOTE_BLOCK_BELL` (Klingel), `ENTITY_PLAYER_LEVELUP` (Energie voll)
- `PlayerToggleFlightEvent` (Doppel-Space), `PlayerSwapHandItemsEvent` (F), `PlayerInteractEvent`
- `BossBar` (Energie-Anzeige), `ItemMeta.setCustomModelData`

> Hinweis: In 26.x sind `Sound`/`Attribute`/`PotionEffectType` u. a. von Enums auf Registry-
> Interfaces umgestellt und Konstanten umbenannt. Konstanten daher per `javap` aus dem
> `paper-api`-Jar verifizieren, statt zu raten.

## Versionsverlauf
- **1.0.0** — craftbares Bike, Fahrmodus, Bergab-Turbo, kein Fallschaden, Step-Up
- **1.1.0** — Sprung-Boost, Doppel-Space-Turbo (3 s/3 s) mit Bossbar
- **1.2.0** — verbrauchsabhängiger Turbo (Energieleiste), Sounds
- **1.2.1** — kein Mid-Air-Boost mehr, Aufladen auf 10 s
- **1.2.2** — Freilauf-Klick-Sound beim Fahren
- **1.2.3** — Geschwindigkeit aus echter Positionsänderung (Klick auch beim normalen Fahren),
  Klick lauter bei Turbo/Sprung
- **1.3.0** — Klingel (Linksklick / F)
