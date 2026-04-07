# 🤖 RobotPi IDE

> Interface de programmation visuelle pour Raspberry Pi Pico avec téléversement via ampy

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Electron](https://img.shields.io/badge/Electron-28.3.3-blue.svg)](https://www.electronjs.org/)
[![MicroPython](https://img.shields.io/badge/MicroPython-1.20+-green.svg)](https://micropython.org/)
[![Platform](https://img.shields.io/badge/Platform-Linux%20%7C%20Windows%20%7C%20macOS-lightgrey.svg)](https://github.com/matelec/RobotPi-IDE)

Application desktop pour linux, utilisé avec debian, pour programmer facilement un robot mobile basé sur Raspberry Pi Pico, driver TB6612FNG et capteur de distance VL53L0X.

## ✨ Fonctionnalités

- 🎨 **Interface par blocs Blockly** — Programmation visuelle intuitive
- 🐍 **Génération de code Python** automatique
- 📤 **Téléversement direct** sur le Pico via ampy
- 📂 **Gestionnaire de fichiers** pour explorer le système de fichiers du Pico
- 💾 **Sauvegarde/chargement** de projets au format JSON
- 📡 **Moniteur série** en temps réel pour le débogage
- 🔄 **Détection automatique** des ports série Raspberry Pi Pico
- ⚙️ **Configuration des pins GPIO** personnalisable
- 🖥️ **Multiplateforme** : Linux, Windows, macOS

## 🎯 Blocs Disponibles

### Mouvements
- Initialiser le robot
- Avancer / Reculer (instantané ou temporisé)
- Tourner à gauche / droite (instantané ou temporisé)
- Stopper les moteurs

### Capteurs
- Détecter un obstacle devant
- Lire la distance en cm
- Éviter un obstacle automatiquement

### Lumières (LEDs WS2812B)
- Allumer/éteindre toutes les LEDs ou une LED spécifique
- Changer la couleur (7 couleurs disponibles)
- Faire clignoter les LEDs
- Régler la luminosité (0–100%)

### Bouton
- Démarrer au bouton (`attendre_bouton_start`)
- Arrêter au bouton (`attendre_bouton_stop`)
- Vérifier si le bouton est appuyé
- Arrêter si bouton appuyé (dans une boucle)

### Temporisation
- Attendre X secondes

### Standard Blockly
- Logique (conditions, opérateurs booléens)
- Boucles (répéter, tant que, pour chaque)
- Mathématiques, Texte, Listes
- Variables et Fonctions

## 📋 Prérequis

### Logiciels

- **Node.js** v16+
- **Python 3.7+**
- **ampy** (Adafruit MicroPython Tool) :
  ```bash
  pip3 install --user adafruit-ampy
  ```

### Matériel

- Raspberry Pi Pico avec MicroPython installé
- Driver moteur TB6612FNG
- 2 moteurs DC
- Capteur de distance VL53L0X (I2C)
- LEDs WS2812B (NeoPixels)
- Bouton poussoir
- Châssis de robot mobile
- Batteries (voir [`micropython/README.md`](micropython/README.md) pour le câblage complet)

## 🚀 Installation

### Méthode 1 : Installeur automatique (Debian/Ubuntu — Recommandé)

Un script d'installation complet est fourni dans `installer/robot-ide-installer/` :

copier le dossier robot-ide-installer sur la machine

```bash
# Se placer dans le dossier 
su -
cd robot-ide-installer/
chmod +x install.sh
./install.sh
```

Le script effectue automatiquement :
- Installation des dépendances système (`fuse`, `libfuse2`, `python3`, etc.)
- Installation d'ampy via pip3
- Configuration des règles USB udev pour le Raspberry Pi Pico
- Ajout de l'utilisateur aux groupes `dialout` et `plugdev`
- Installation de l'AppImage dans `/opt/robotpi-ide/`
- Création du lanceur dans le menu des applications

> ⚠️ Nécessite les droits root. Conçu pour Debian 12 / Ubuntu.

Pour désinstaller :
```bash
su -
cd robot-ide-installer/
chmod +x uninstall.sh
./uninstall.sh
```

## 🛠️ Structure du projet

```
ROBOTPI_HACKATHON_LINUX/
│
├── Documents/                       # Documents pdf
│   └── RobotPi - dossier eleves.pdf
│   └── RobotPi_Challenges.pdf
│   └── RobotPi.pdf     
│
├── assets/
│   └── icon.png                   # Icône (512×512)
│
├── micropython/
│   ├── robotPi.py                 # Librairie MicroPython pour le Pico
│   └── README.md                  # Documentation de la librairie
│
├── installer/
│   └── robot-ide-installer/
│       ├── install.sh             # Script d'installation Debian/Ubuntu
│       └── uninstall.sh           # Script de désinstallation
│
├── Electronique/                  # Fichiers de fabrication de la carte
└── Construction/                  # Fichiers de conception mécanique
```

## 📖 Guide d'utilisation

### Première utilisation

1. Connecter le Raspberry Pi Pico via USB
2. Lancer RobotPi IDE
3. Sélectionner le port série dans la barre d'outils
4. Cliquer sur **"Installer Lib"** pour téléverser `robotPi.py` sur le Pico
5. Créer votre programme avec les blocs Blockly
6. Cliquer sur **"Générer Python"** pour voir le code généré
7. Cliquer sur **"Téléverser"** pour envoyer le programme sur le Pico

### Moniteur série

- **"📡 Moniteur série"** — affiche les `print()` du Pico en temps réel
- **"🔄 Reset Pico"** — redémarre le programme en cours

### Gestionnaire de fichiers

- **"📂 Fichiers"** — liste les fichiers présents sur le Pico
- Permet de télécharger ou supprimer des fichiers depuis le Pico

## 🐛 Dépannage

### ampy non trouvé
```bash
pip3 install --user adafruit-ampy
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
ampy --version
```

### Permission refusée sur le port série (Linux)
```bash
sudo usermod -a -G dialout $USER
# Se déconnecter et reconnecter pour que le changement prenne effet
```

### Le Pico n'est pas détecté
1. Vérifier que MicroPython est installé sur le Pico
2. Débrancher / rebrancher le Pico
3. Cliquer sur **🔄 Actualiser les ports**
4. Sur Windows, installer les drivers USB si nécessaire

### Erreur de téléversement
- Fermer le moniteur série avant de téléverser
- Vérifier qu'aucun autre programme n'utilise le port série
- Redémarrer l'application

### AppImage ne se lance pas (Linux)
```bash
sudo apt-get install fuse libfuse2
chmod +x RobotPi-IDE-*.AppImage
```

## 📝 Changelog

### Version 1.0.0 (2025-01-31)
- 🎉 Version initiale
- ✨ Interface Blockly complète
- 📤 Téléversement via ampy
- 📡 Moniteur série temps réel
- 📂 Gestionnaire de fichiers
- 🎨 Support LEDs WS2812B
- 🤖 Support TB6612FNG
- 📏 Support VL53L0X

## 📚 Ressources

- [Documentation MicroPython Pico](https://docs.micropython.org/en/latest/rp2/quickref.html)
- [Datasheet TB6612FNG](https://www.sparkfun.com/datasheets/Robotics/TB6612FNG.pdf)
- [Documentation VL53L0X](https://www.st.com/en/imaging-and-photonics-solutions/vl53l0x.html)
- [Guide WS2812B](https://cdn-shop.adafruit.com/datasheets/WS2812B.pdf)
- [Blockly Developer Tools](https://developers.google.com/blockly/guides/overview)
- [Librairie robotPi.py](micropython/README.md) — documentation complète de la librairie MicroPython

## 📄 Licence

Ce projet est sous licence MIT — voir le fichier [LICENSE](LICENSE) pour plus de détails.

## 👨‍💻 Auteur

**RATTE MATTHIAS**  
GitHub : [@matelec](https://github.com/matelec)  
Projet : [RobotPi-IDE](https://github.com/matelec/RobotPi-IDE)

## 🙏 Remerciements

- [Electron](https://www.electronjs.org/) — Framework desktop
- [Blockly](https://developers.google.com/blockly) — Éditeur visuel
- [Adafruit ampy](https://github.com/scientifichackers/ampy) — Outil MicroPython
- [MicroPython](https://micropython.org/) — Python pour microcontrôleurs
- [Raspberry Pi Pico](https://www.raspberrypi.com/products/raspberry-pi-pico/) — Microcontrôleur RP2040

---

<div align="center">

**⭐ Si ce projet vous aide, n'hésitez pas à lui donner une étoile ! ⭐**

[Signaler un bug](https://github.com/matelec/RobotPi-IDE/issues) · [Demander une fonctionnalité](https://github.com/matelec/RobotPi-IDE/issues) · [Documentation librairie](micropython/README.md)

</div>