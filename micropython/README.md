# 🤖 robotPi - Librairie Robot pour Raspberry Pi Pico

Librairie complète pour contrôler un robot mobile basé sur Raspberry Pi Pico avec driver moteur TB6612FNG, capteur de distance VL53L0X et LEDs WS2812B.

## 📦 Fonctionnalités

- ✅ Contrôle de 2 moteurs DC via TB6612FNG
- ✅ Capteur de distance laser VL53L0X (jusqu'à 2m) — intégré directement dans la librairie
- ✅ LEDs RGB WS2812B programmables
- ✅ Fonctions de mouvement avec durée
- ✅ Évitement d'obstacles automatique
- ✅ Support bouton tactile (start, stop, détection)

## 🔌 Schéma de câblage

### ⚡ Alimentation par batterie Li-ion 18650

**Configuration recommandée :**
- **2x batteries 18650 en série** = 7.4V nominal (8.4V chargées, 6V déchargées)
- Support 2x 18650 avec protection intégrée
- Module de charge TP4056 (avec protection) pour chaque batterie
- Interrupteur ON/OFF sur le +

**Schéma d'alimentation :**
```
Batteries 18650 (2S)
    │
    ├─→ Interrupteur ON/OFF
    │
    ├─→ VM (TB6612FNG)  ← Alimente les moteurs (7.4V)
    │
    └─→ VSYS (Pico)     ← Alimente le Pico (via régulateur interne)
         │
         └─→ 3.3V sort automatiquement sur pin 3V3
              │
              ├─→ VCC TB6612FNG
              ├─→ VCC VL53L0X
              └─→ GND commun
    
    VBUS (Pico) → VCC WS2812B  ← 5V généré par le régulateur du Pico
```

⚠️ **Important :**
- Ne jamais brancher USB et batteries en même temps
- Utiliser un support avec protection contre décharge profonde
- Tension minimale : 6V (3V par cellule)
- Courant de pointe moteurs : ~2A par moteur

### TB6612FNG (Contrôleur moteurs)
```
PWMA  → GPIO 0   (PWM moteur gauche)
AIN1  → GPIO 1
AIN2  → GPIO 2
PWMB  → GPIO 3   (PWM moteur droit)
BIN1  → GPIO 4
BIN2  → GPIO 5
STBY  → GPIO 6   (Standby)
VCC   → 3.3V     (du Pico)
GND   → GND      (commun)
VM    → 7.4V     (batteries 18650 2S)
```

### VL53L0X (Capteur distance)
```
VCC → 3.3V
GND → GND
SCL → GPIO 9
SDA → GPIO 8
```

### WS2812B (LEDs RGB)
```
VCC → VBUS (5V du Pico, pin 40)
GND → GND
DIN → GPIO 15
```
⚠️ **Note :** Le Pico génère du 5V sur VBUS uniquement quand alimenté par VSYS (batteries) ou USB

### Bouton tactile (optionnel)
```
Un côté     → GPIO 14
Autre côté  → GND
```
*Note : Utilise le pull-up interne. Le bouton est détecté à l'état HAUT (relâché = 0, appuyé = 1).*

## 🚀 Installation

1. Copiez `robotPi.py` sur votre Raspberry Pi Pico
2. Importez la librairie dans votre code :

```python
from machine import I2C, Pin
import robotPi
```

## 📖 Utilisation de base

### Initialisation simple (moteurs uniquement)

```python
robot = robotPi.RobotPi(
    pwm_g=0, in1_g=1, in2_g=2,    # Moteur gauche
    pwm_d=3, in1_d=4, in2_d=5,    # Moteur droit
    stby_pin=6                     # Standby
)
```

### Initialisation complète (avec LEDs, capteur et bouton)

```python
from machine import I2C, Pin
import robotPi

# Configuration I2C pour le capteur
i2c = I2C(0, scl=Pin(9), sda=Pin(8), freq=400000)

# Création du robot
robot = robotPi.RobotPi(
    pwm_g=0, in1_g=1, in2_g=2,
    pwm_d=3, in1_d=4, in2_d=5,
    stby_pin=6,
    led_pin=15,       # LEDs WS2812B
    nb_leds=4,        # Nombre de LEDs
    pin_bouton=14,    # Bouton tactile
    i2c=i2c           # Bus I2C pour VL53L0X
)
```

> **Note :** La vitesse par défaut est de **95%**. Elle est modifiable via `robot.vitesse_defaut`.

## 🎮 Contrôle des moteurs

### Mouvements continus

```python
# Avancer à vitesse par défaut (95%)
robot.avancer()

# Avancer à vitesse spécifique (0 à 100)
robot.avancer(vitesse=50)

# Reculer
robot.reculer(vitesse=60)

# Tourner à gauche (moteur gauche recule, moteur droit avance)
robot.tourner_gauche(vitesse=70)

# Tourner à droite (moteur gauche avance, moteur droit recule)
robot.tourner_droite(vitesse=70)

# Arrêter
robot.stopper()
```

### Mouvements avec durée

> ⚠️ **Attention à l'ordre des paramètres :** `vitesse` vient avant `duree`.

```python
# Avancer pendant 2 secondes puis s'arrêter
robot.avancer_pendant(vitesse=70, duree=2)

# Reculer pendant 1.5 secondes
robot.reculer_pendant(vitesse=60, duree=1.5)

# Tourner à gauche pendant 0.5 secondes
robot.tourner_gauche_pendant(duree=0.5)

# Tourner à droite pendant 0.8 secondes
robot.tourner_droite_pendant(vitesse=80, duree=0.8)
```

## 💡 Contrôle des LEDs

### Allumer / Éteindre

```python
# Allumer toutes les LEDs en rouge (R, G, B)
robot.allumer_leds(255, 0, 0)

# Allumer toutes les LEDs en vert
robot.allumer_leds(0, 255, 0)

# Allumer toutes les LEDs en bleu
robot.allumer_leds(0, 0, 255)

# Allumer une LED spécifique (index 0 à nb_leds-1)
robot.allumer_led(0, 255, 0, 0)  # Première LED en rouge

# Éteindre toutes les LEDs
robot.eteindre_leds()

# Éteindre une LED spécifique
robot.eteindre_led(0)
```

### Luminosité

```python
# Allumer toutes les LEDs avec une luminosité (0 à 100)
robot.allumer_leds_luminosite(255, 0, 0, luminosite=50)  # Rouge à 50%

# Allumer une LED spécifique avec une luminosité
robot.allumer_led_luminosite(0, 255, 0, 0, luminosite=30)  # LED 0, rouge à 30%
```

### Effets lumineux

```python
# Clignoter en rouge 5 fois avec intervalle de 0.3s
robot.clignoter_leds(255, 0, 0, nb_fois=5, intervalle=0.3)

# Arc-en-ciel (couleur différente par LED, décalage par index)
robot.couleur_arc_en_ciel(0)
```

## 📏 Capteur de distance

### Lecture de distance

```python
# Lire en millimètres
distance_mm = robot.lire_distance()
print(f"Distance: {distance_mm} mm")

# Lire en centimètres (arrondi à 1 décimale)
distance_cm = robot.lire_distance_cm()
print(f"Distance: {distance_cm} cm")
```

Les deux méthodes retournent `None` en cas d'erreur de lecture.

### Détection d'obstacles

```python
# Vérifier si obstacle à moins de 20 cm
if robot.obstacle_detecte(seuil_cm=20):
    print("Obstacle détecté !")
    robot.allumer_leds(255, 0, 0)  # Rouge
else:
    robot.allumer_leds(0, 255, 0)  # Vert

# Éviter automatiquement un obstacle
# (recule 0.5s, tourne à droite 0.5s, puis repart)
# Retourne True si un obstacle a été évité, False sinon
evite = robot.eviter_obstacle(seuil_cm=20, vitesse=70)
```

## 🔘 Contrôle par bouton

```python
# Attendre un appui bouton pour démarrer (bloquant)
robot.attendre_bouton_start()

# Attendre un appui bouton pour stopper (bloquant, arrête moteurs et LEDs)
robot.attendre_bouton_stop()

# Vérifier si le bouton est appuyé à l'instant T (non bloquant)
if robot.bouton_appuye():
    print("Bouton pressé !")

# Arrêter le robot si le bouton est appuyé (non bloquant)
# Retourne True si arrêt déclenché, False sinon
if robot.arreter_si_bouton():
    print("Arrêt déclenché par le bouton")
```

## 🎯 Exemples complets

### Exemple 1 : Parcours en carré

```python
import robotPi

robot = robotPi.RobotPi(0, 1, 2, 3, 4, 5, stby_pin=6)

for i in range(4):
    robot.avancer_pendant(vitesse=70, duree=2)
    robot.tourner_droite_pendant(vitesse=70, duree=0.5)
```

### Exemple 2 : Robot autonome avec évitement

```python
from machine import I2C, Pin
import robotPi
import time

i2c = I2C(0, scl=Pin(9), sda=Pin(8), freq=400000)
robot = robotPi.RobotPi(0, 1, 2, 3, 4, 5,
                        stby_pin=6, led_pin=15, nb_leds=4, i2c=i2c)

while True:
    if robot.obstacle_detecte(seuil_cm=20):
        robot.allumer_leds(255, 0, 0)  # Rouge
        robot.eviter_obstacle()
    else:
        robot.allumer_leds(0, 255, 0)  # Vert
        robot.avancer(70)
    time.sleep(0.1)
```

### Exemple 3 : Démarrage / arrêt par bouton

```python
from machine import I2C, Pin
import robotPi
import time

i2c = I2C(0, scl=Pin(9), sda=Pin(8), freq=400000)
robot = robotPi.RobotPi(0, 1, 2, 3, 4, 5,
                        stby_pin=6, led_pin=15, nb_leds=4,
                        pin_bouton=14, i2c=i2c)

print("Appuyez sur le bouton pour démarrer")
robot.attendre_bouton_start()
robot.allumer_leds(0, 255, 0)

while True:
    if robot.obstacle_detecte(seuil_cm=20):
        robot.allumer_leds(255, 165, 0)  # Orange
        robot.eviter_obstacle()
    else:
        robot.allumer_leds(0, 255, 0)  # Vert
        robot.avancer(70)

    if robot.arreter_si_bouton():
        break

    time.sleep(0.1)
```

### Exemple 4 : Indicateur de distance avec LEDs

```python
from machine import I2C, Pin
import robotPi
import time

i2c = I2C(0, scl=Pin(9), sda=Pin(8), freq=400000)
robot = robotPi.RobotPi(0, 1, 2, 3, 4, 5,
                        stby_pin=6, led_pin=15, nb_leds=4, i2c=i2c)

while True:
    distance = robot.lire_distance_cm()

    if distance is not None:
        if distance < 10:
            robot.allumer_leds(255, 0, 0)      # Rouge < 10cm
        elif distance < 20:
            robot.allumer_leds(255, 165, 0)    # Orange < 20cm
        elif distance < 30:
            robot.allumer_leds(255, 255, 0)    # Jaune < 30cm
        else:
            robot.allumer_leds(0, 255, 0)      # Vert > 30cm

        print(f"Distance: {distance:.1f} cm")

    time.sleep(0.2)
```

## ⚙️ Configuration avancée

### Modifier la vitesse par défaut

```python
robot = robotPi.RobotPi(0, 1, 2, 3, 4, 5, stby_pin=6)
robot.vitesse_defaut = 70  # 70% au lieu de 95%
```

### Utilisation sans composants optionnels

```python
# Sans LEDs, capteur ni bouton
robot = robotPi.RobotPi(0, 1, 2, 3, 4, 5, stby_pin=6)

# Seulement avec LEDs
robot = robotPi.RobotPi(0, 1, 2, 3, 4, 5,
                        stby_pin=6, led_pin=15, nb_leds=4)

# Seulement avec capteur
i2c = I2C(0, scl=Pin(9), sda=Pin(8), freq=400000)
robot = robotPi.RobotPi(0, 1, 2, 3, 4, 5,
                        stby_pin=6, i2c=i2c)

# Seulement avec bouton
robot = robotPi.RobotPi(0, 1, 2, 3, 4, 5,
                        stby_pin=6, pin_bouton=14)
```

## 📚 API Complète

### Constructeur

```python
RobotPi(pwm_g, in1_g, in2_g, pwm_d, in1_d, in2_d, stby_pin,
        led_pin=None, nb_leds=0, pin_bouton=None, i2c=None)
```

| Paramètre | Type | Description |
|-----------|------|-------------|
| `pwm_g` | int | GPIO PWM moteur gauche |
| `in1_g` | int | GPIO IN1 moteur gauche |
| `in2_g` | int | GPIO IN2 moteur gauche |
| `pwm_d` | int | GPIO PWM moteur droit |
| `in1_d` | int | GPIO IN1 moteur droit |
| `in2_d` | int | GPIO IN2 moteur droit |
| `stby_pin` | int | GPIO Standby TB6612FNG |
| `led_pin` | int | GPIO données WS2812B (optionnel) |
| `nb_leds` | int | Nombre de LEDs WS2812B (optionnel) |
| `pin_bouton` | int | GPIO bouton tactile (optionnel) |
| `i2c` | I2C | Bus I2C pour VL53L0X (optionnel) |

### Méthodes de mouvement

| Méthode | Description |
|---------|-------------|
| `avancer(vitesse=None)` | Avancer en continu |
| `reculer(vitesse=None)` | Reculer en continu |
| `tourner_gauche(vitesse=None)` | Tourner à gauche en continu |
| `tourner_droite(vitesse=None)` | Tourner à droite en continu |
| `avancer_pendant(vitesse=None, duree=None)` | Avancer puis stopper après `duree` secondes |
| `reculer_pendant(vitesse=None, duree=None)` | Reculer puis stopper après `duree` secondes |
| `tourner_gauche_pendant(vitesse=None, duree=None)` | Tourner à gauche puis stopper |
| `tourner_droite_pendant(vitesse=None, duree=None)` | Tourner à droite puis stopper |
| `stopper()` | Arrêt complet des deux moteurs |

### Méthodes LEDs

| Méthode | Description |
|---------|-------------|
| `allumer_led(index, r, g, b)` | Allume une LED spécifique |
| `allumer_leds(r, g, b)` | Allume toutes les LEDs |
| `allumer_led_luminosite(index, r, g, b, luminosite)` | Allume une LED avec luminosité (0–100) |
| `allumer_leds_luminosite(r, g, b, luminosite)` | Allume toutes les LEDs avec luminosité (0–100) |
| `eteindre_led(index)` | Éteint une LED |
| `eteindre_leds()` | Éteint toutes les LEDs |
| `couleur_arc_en_ciel(index)` | Arc-en-ciel décalé selon `index` |
| `clignoter_leds(r, g, b, nb_fois=3, intervalle=0.5)` | Clignotement |

### Méthodes capteur de distance

| Méthode | Description |
|---------|-------------|
| `lire_distance()` | Distance en millimètres (ou `None`) |
| `lire_distance_cm()` | Distance en centimètres (ou `None`) |
| `obstacle_detecte(seuil_cm=20)` | `True` si obstacle sous le seuil |
| `eviter_obstacle(seuil_cm=20, vitesse=None)` | Recule + tourne si obstacle, retourne `True`/`False` |

### Méthodes bouton

| Méthode | Description |
|---------|-------------|
| `attendre_bouton_start()` | Bloque jusqu'à appui, puis continue |
| `attendre_bouton_stop()` | Bloque jusqu'à appui, puis stoppe moteurs et LEDs |
| `bouton_appuye()` | Retourne `True` si bouton pressé (non bloquant) |
| `arreter_si_bouton()` | Stoppe le robot si bouton pressé, retourne `True`/`False` |

## 🔧 Dépannage

### Le robot ne bouge pas
- Vérifier que STBY est connecté et à HIGH
- Vérifier l'alimentation VM du TB6612FNG (7.4V batteries)
- Vérifier les connexions des moteurs
- Vérifier la charge des batteries (> 6V)
- Vérifier l'interrupteur ON/OFF

### Les LEDs ne s'allument pas
- Vérifier que les batteries alimentent bien le Pico (VSYS)
- Vérifier que VBUS (pin 40) fournit bien 5V
- Vérifier la connexion DIN sur GPIO 15
- Vérifier le paramètre `nb_leds` lors de l'init
- Les WS2812B ne fonctionnent pas en USB uniquement si VSYS n'est pas alimenté

### Le capteur ne fonctionne pas
- Vérifier les connexions I2C (SCL=9, SDA=8)
- Vérifier l'alimentation 3.3V du capteur
- Tester avec un scanner I2C :
```python
i2c = I2C(0, scl=Pin(9), sda=Pin(8), freq=400000)
print(i2c.scan())  # Devrait afficher [41] (0x29 en décimal)
```

### Le bouton ne répond pas
- Vérifier que `pin_bouton` est bien renseigné lors de l'init
- Le pull-up interne est activé : le bouton doit relier la broche à GND
- La détection d'appui correspond à `value() == 1` (niveau haut après relâchement du GND)

### Autonomie faible
- Vérifier la capacité des batteries (recommandé : 2500–3500mAh)
- Réduire la vitesse par défaut : `robot.vitesse_defaut = 50`
- Réduire la luminosité des LEDs : `robot.allumer_leds_luminosite(r, g, b, 30)`
- Éteindre les LEDs quand inutiles : `robot.eteindre_leds()`

## 🔋 Conseils batteries

### Charge
- Utiliser des modules TP4056 avec protection
- Temps de charge : ~3–4h pour 2500mAh
- Toujours éteindre le robot pendant la charge

### Utilisation
- Tension nominale : 7.4V (2S)
- Tension max : 8.4V (chargées)
- Tension min : 6V (ne pas descendre en dessous)
- Autonomie estimée avec 2500mAh : 45–60 minutes

### Sécurité
- ⚠️ Ne jamais court-circuiter les batteries
- ⚠️ Ne jamais percer ou chauffer les batteries
- ⚠️ Utiliser uniquement des batteries avec protection PCB
- ⚠️ Arrêter le robot si tension < 6V

## 📝 Licence

Cette librairie est distribuée sous licence MIT.

## 🤝 Contribution

Les contributions sont les bienvenues ! N'hésitez pas à proposer des améliorations.

---

**Créé pour Raspberry Pi Pico avec MicroPython** 🐍