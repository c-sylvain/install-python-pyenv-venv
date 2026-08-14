# install-python-pyenv-venv

Journal d'exécution de la procédure [`Python-Ubuntu-Setup.md`](Python-Ubuntu-Setup.md).

- **Date d'exécution** : 2026-08-14
- **Machine** : Ubuntu 24.04.4 LTS (Noble Numbat), noyau 7.0.0-28-generic
- **Utilisateur** : `sylvain`
- **Résultat** : ✅ procédure complète, vérification finale conforme

## État initial

| Élément | Avant |
|---|---|
| Python système | 3.12.3 (`/usr/bin/python3`) |
| pyenv | absent |
| Config `~/.bashrc` | aucune ligne pyenv |
| Dépendances de build | 15 des 19 paquets manquants |

## Étapes réalisées

### 1. Dépendances système

Les 19 paquets de la procédure ont été vérifiés un à un avec `dpkg-query`. 15 manquaient. Disponibilité confirmée dans les dépôts noble avec `apt-cache` avant installation.

Installation effectuée par l'utilisateur (sudo interactif, mot de passe requis) en deux passes :

```bash
sudo apt update
sudo apt install -y build-essential curl git libssl-dev zlib1g-dev libbz2-dev \
  libreadline-dev libsqlite3-dev wget llvm libncursesw5-dev xz-utils
sudo apt install -y tk-dev libxmlsec1-dev liblzma-dev python3-openssl python3-pip python3-venv
```

Note : `libncursesw5-dev` n'existe plus comme paquet réel sur noble — c'est un paquet virtuel fourni par `libncurses-dev`, qu'apt a installé à sa place. `libxml2-dev` et `libffi-dev` sont arrivés comme dépendances automatiques de `llvm`.

### 2. Installation de pyenv

```bash
curl -fsSL https://pyenv.run | bash
```

→ pyenv **2.8.4** installé dans `~/.pyenv` (avec les plugins `pyenv-virtualenv`, `pyenv-update`, `pyenv-doctor`).

### 3. Configuration du shell

Quatre lignes ajoutées à `~/.bashrc` (ajout conditionnel, pour rester idempotent en cas de relance) :

```bash
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init --path)"
eval "$(pyenv init -)"
```

L'`exec $SHELL` de la procédure a été remplacé par un chargement explicite de l'environnement dans chaque commande : un `exec` remplacerait le processus shell et interromprait la session non interactive.

### 4. Vérification de pyenv

```bash
pyenv --version   # → pyenv 2.8.4
```

### 5. Python 3.12.9

```bash
pyenv install 3.12.9
pyenv global 3.12.9
python --version   # → Python 3.12.9
which python       # → /home/sylvain/.pyenv/shims/python
```

Compilé avec GCC 13.3.0, sans avertissement de module manquant. Contrôle supplémentaire des modules qui dépendent des bibliothèques de l'étape 1 — ce sont ceux qui échouent silencieusement si un `-dev` manque au moment du build :

```bash
python -c "import ssl, sqlite3, lzma, bz2, ctypes, readline, zlib"   # OK
python -c "import tkinter"                                           # OK
```

### 6. Projet et environnement virtuel

```bash
mkdir -p ~/projects/ansible-lab
cd ~/projects/ansible-lab
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip     # 24.3.1 → 26.2.1
python -m pip install ansible           # ansible 14.3.1 / ansible-core 2.21.3
```

## Vérification finale

```
which python  → /home/sylvain/projects/ansible-lab/.venv/bin/python
which ansible → /home/sylvain/projects/ansible-lab/.venv/bin/ansible
```

```
ansible [core 2.21.3]
  ansible python module location = /home/sylvain/projects/ansible-lab/.venv/lib/python3.12/site-packages/ansible
  executable location = /home/sylvain/projects/ansible-lab/.venv/bin/ansible
  python version = 3.12.9 (main, Aug 14 2026, 20:56:49) [GCC 13.3.0]
  jinja version = 3.1.6
  pyyaml version = 6.0.3 (with libyaml v0.2.5)
```

Les trois commandes pointent bien vers `~/projects/ansible-lab/.venv/bin/` — critère de la procédure rempli.

## Versions installées

| Composant | Version |
|---|---|
| pyenv | 2.8.4 |
| Python (pyenv global) | 3.12.9 |
| pip (dans le venv) | 26.2.1 |
| ansible | 14.3.1 |
| ansible-core | 2.21.3 |
| jinja2 | 3.1.6 |
| cryptography | 50.0.0 |

## Écarts par rapport à la procédure

1. **`exec $SHELL` non exécuté** (étape 3) — incompatible avec une session non interactive. L'environnement pyenv est chargé explicitement à la place. Sans effet sur le résultat : la config est en place dans `~/.bashrc` et s'appliquera aux prochains shells.
2. **`sudo apt` délégué à l'utilisateur** (étape 1) — sudo exige un mot de passe saisi au terminal.
3. **`libncursesw5-dev`** — résolu vers `libncurses-dev` par apt (paquet virtuel sur noble). `dpkg-query` le rapporte comme absent alors que la dépendance est satisfaite ; c'est le champ `Provides` qui fait foi.

## Persistance après redémarrage

Vérifié le 2026-08-14, sans redémarrer la machine : un reboot ne modifie rien sur le disque, ce qu'il met réellement à l'épreuve c'est le chargement de `~/.bashrc` dans un shell neuf. C'est donc ce qui a été testé, via `bash -ic` (terminal graphique) et `bash -lic` (SSH / TTY).

| Contexte | pyenv | python |
|---|---|---|
| Shell interactif (terminal graphique) | 2.8.4 | 3.12.9 via shims |
| Shell de login (SSH / TTY) | 2.8.4 | 3.12.9 via shims |

Activation du venv depuis un shell neuf : `python` et `ansible` résolvent bien vers `.venv/bin/`, `ansible-core 2.21.3` répond, et les imports `ansible`, `ssl`, `sqlite3`, `lzma` passent.

Ce qui garantit la persistance :

- `~/.pyenv/version` contient `3.12.9` — un fichier sur disque, pas une variable d'environnement
- `.venv/bin/python` est un lien symbolique **absolu** vers `~/.pyenv/versions/3.12.9/bin/python`
- `.venv/pyvenv.cfg` référence le même chemin en dur (`home = /home/sylvain/.pyenv/versions/3.12.9/bin`)
- aucune référence à `/tmp` ni à un emplacement éphémère

> ⚠️ Le venv est lié en dur à `~/.pyenv/versions/3.12.9`. Un `pyenv uninstall 3.12.9` le casserait sans avertissement — il faudrait alors le recréer (`python -m venv .venv` puis réinstaller les dépendances).

## Second projet : une autre version de Python

Ajouté le 2026-08-14, hors procédure initiale, pour valider la cohabitation de deux versions de Python sur la même machine.

```bash
pyenv install 3.13.15

mkdir -p ~/projects/python-lab && cd ~/projects/python-lab
pyenv local 3.13.15          # écrit .python-version dans le dossier
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
```

Aucun paquet apt supplémentaire n'a été nécessaire : les dépendances de build de l'étape 1 ont resservi telles quelles. Les modules critiques (`ssl`, `sqlite3`, `lzma`, `bz2`, `ctypes`, `readline`, `zlib`, `tkinter`) sont tous présents sur le 3.13.15.

À noter : `pip` était déjà en 26.2.1 dans le venv neuf — l'`ensurepip` de Python 3.13.15 embarque une version récente, l'`--upgrade` n'avait donc rien à faire.

### Les deux environnements côte à côte

| | `ansible-lab` | `python-lab` |
|---|---|---|
| Python | 3.12.9 | 3.13.15 |
| Sélection de version | global pyenv | `.python-version` (local) |
| Contenu du venv | ansible 14.3.1 | nu, pip 26.2.1 |
| Vérifié en shell neuf | ✅ | ✅ |

`pyenv global` reste à **3.12.9** : `pyenv local` n'écrit qu'un fichier `.python-version` dans le dossier du projet. Entrer dans `~/projects/python-lab` bascule automatiquement sur 3.13.15, en sortir revient à 3.12.9. Vérifié après coup que `ansible-lab` est resté intact.

## Reprise de l'environnement

Dans un nouveau terminal (la config `~/.bashrc` est déjà active) :

```bash
# Projet ansible (Python 3.12.9)
cd ~/projects/ansible-lab && source .venv/bin/activate

# Projet python-lab (Python 3.13.15, sélectionné par .python-version)
cd ~/projects/python-lab && source .venv/bin/activate
```
