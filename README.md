# install-python-pyenv-venv

Mise en place d'un environnement Python complet sur Ubuntu vierge, avec **pyenv** et **venv**, puis deux projets de test.

Ce dépôt contient deux choses :

- une partie **pédagogique** (concepts, démarrage rapide, erreurs classiques) — à lire si Python est nouveau pour vous ;
- un **journal d'exécution** détaillé de l'installation réellement effectuée, avec les versions, les vérifications et les écarts constatés.

| | |
|---|---|
| **Date d'exécution** | 2026-08-14 |
| **Machine** | Ubuntu 24.04.4 LTS (Noble Numbat), noyau 7.0.0-28-generic |
| **Procédure source** | [`Python-Ubuntu-Setup.md`](Python-Ubuntu-Setup.md) |
| **Résultat** | ✅ procédure complète, vérification finale conforme |

## Sommaire

- [Les concepts en deux minutes](#les-concepts-en-deux-minutes)
- [Pour démarrer](#pour-démarrer)
- [Vocabulaire](#vocabulaire)
- [Journal d'exécution](#journal-dexécution) — le détail de l'installation
- [Second projet : une autre version de Python](#second-projet--une-autre-version-de-python)
- [Test fonctionnel : un playbook sur les deux versions](#test-fonctionnel--un-playbook-sur-les-deux-versions)

---

# Les concepts en deux minutes

## Le problème que ça résout

Ubuntu est livré avec son propre Python (ici 3.12.3), dont le système lui-même se sert. Deux difficultés en découlent :

1. **On ne veut pas y toucher.** Installer ou supprimer des paquets dans le Python du système peut casser des outils d'Ubuntu qui en dépendent.
2. **Un projet a rarement les mêmes besoins qu'un autre.** L'un demande Python 3.12 et une vieille version d'une bibliothèque, l'autre Python 3.13 et la dernière. Un seul Python partagé ne peut pas satisfaire les deux.

D'où deux outils, qui règlent deux problèmes distincts et se complètent :

| Outil | Répond à la question | Fichier qui le pilote |
|---|---|---|
| **pyenv** | *Quelle **version de Python** ?* | `.python-version` |
| **venv** | *Quels **paquets** installés ?* | `requirements.txt` |

C'est la distinction à retenir avant tout le reste. La confusion entre les deux est la source d'erreur la plus fréquente au début.

## Comment ça s'empile

```
Système : /usr/bin/python3  (3.12.3)   ← appartient à Ubuntu, on n'y touche pas
   │
pyenv : ~/.pyenv/versions/              ← plusieurs versions de Python installées côte à côte
   ├── 3.12.9      ← version choisie par défaut (« global »)
   └── 3.13.15     ← version choisie dans un dossier précis (« local »)
   │
venv : un dossier .venv/ par projet     ← les paquets, isolés projet par projet
   ├── ~/projects/ansible-lab/.venv     → s'appuie sur 3.12.9  + ansible
   └── ~/projects/python-lab/.venv      → s'appuie sur 3.13.15 + ansible
```

Chaque étage s'appuie sur celui du dessus. Un venv n'embarque pas Python : il pointe vers une version installée par pyenv et n'ajoute que les paquets.

## Ce que fait « activer un venv »

`source .venv/bin/activate` ne fait rien de magique : la commande met simplement `.venv/bin/` en tête du `PATH`, de sorte que `python` et `pip` trouvés en premier soient ceux du projet. C'est visible :

| | `pip` utilisé | Les paquets s'installent dans |
|---|---|---|
| **Sans activation** | 24.3.1 | `~/.pyenv/versions/3.12.9/lib/python3.12/site-packages` |
| **Avec activation** | 26.2.1 | `~/projects/ansible-lab/.venv/lib/python3.12/site-packages` |

Le nom du venv apparaît aussi dans l'invite du terminal, et la commande `deactivate` revient à l'état précédent.

---

# Pour démarrer

## Travailler sur un projet existant

Dans un terminal neuf — la configuration est déjà chargée par `~/.bashrc` :

```bash
# Projet ansible (Python 3.12.9)
cd ~/projects/ansible-lab && source .venv/bin/activate

# Projet python-lab (Python 3.13.15, sélectionné par .python-version)
cd ~/projects/python-lab && source .venv/bin/activate
```

Pour sortir de l'environnement, à la fin :

```bash
deactivate
```

## Le cycle de travail habituel

```bash
cd ~/projects/mon-projet
source .venv/bin/activate          # 1. activer — toujours en premier

python -m pip install requests     # 2. installer ce dont on a besoin
python mon_script.py               # 3. travailler

python -m pip freeze > requirements.txt   # 4. figer les versions
deactivate                                # 5. sortir
```

L'étape 1 conditionne tout le reste : sans elle, les étapes 2 et 3 s'adressent au mauvais Python.

## Créer un nouveau projet de zéro

```bash
mkdir -p ~/projects/mon-projet && cd ~/projects/mon-projet

pyenv local 3.13.15                # (facultatif) fixer la version de Python ici
python -m venv .venv               # créer l'environnement virtuel
source .venv/bin/activate          # l'activer
python -m pip install --upgrade pip
```

`pyenv local` est facultatif : sans lui, le projet utilise la version globale (ici 3.12.9). Il devient utile dès qu'un projet a besoin d'une version différente des autres.

## `requirements.txt` : refaire l'environnement ailleurs

Le dossier `.venv/` **ne se recopie pas** d'une machine à l'autre : il contient des chemins absolus vers `~/.pyenv/versions/...`. Ce qui se transporte, c'est la *liste* des paquets.

```bash
python -m pip freeze > requirements.txt      # figer l'état actuel
python -m pip install -r requirements.txt    # le restaurer ailleurs
```

Règle qui va avec : on versionne `requirements.txt` dans git, **jamais** le dossier `.venv/` (lourd, et inutilisable sur une autre machine). Un fichier `.gitignore` contenant `.venv/` suffit.

Les deux projets décrits ici vivent dans `~/projects/`, hors de ce dépôt. Une copie de leurs `requirements.txt` y est versionnée pour référence :

- [`projets/ansible-lab/requirements.txt`](projets/ansible-lab/requirements.txt)
- [`projets/python-lab/requirements.txt`](projets/python-lab/requirements.txt)

Les deux sont strictement identiques :

```
ansible==14.3.1        cryptography==50.0.0    packaging==26.3
ansible-core==2.21.3   Jinja2==3.1.6           pycparser==3.0
cffi==2.1.1            MarkupSafe==3.0.3       PyYAML==6.0.3
                                               resolvelib==1.2.1
```

Attendu : seul l'interpréteur diffère entre les deux environnements, pas les paquets.

Pour recréer l'un de ces environnements sur une autre machine :

```bash
pyenv install 3.12.9                 # ou 3.13.15 pour python-lab
mkdir -p ~/projects/ansible-lab && cd ~/projects/ansible-lab
python -m venv .venv && source .venv/bin/activate
python -m pip install -r /chemin/vers/projets/ansible-lab/requirements.txt
```

> Ces fichiers sont des **copies** prises le 2026-08-14, les originaux restant dans les dossiers de projet. Après tout `pip install` dans un venv, il faut régénérer puis recopier, sinon les deux divergent silencieusement :
>
> ```bash
> python -m pip freeze > requirements.txt
> ```

## Les erreurs classiques

| Symptôme | Cause probable | Correction |
|---|---|---|
| `ModuleNotFoundError` alors que le paquet vient d'être installé | Installé hors venv, ou venv non activé au lancement | `source .venv/bin/activate`, puis réinstaller |
| `pip install` réclame les droits root | Le venv n'est pas activé — pip vise une installation partagée | Activer le venv ; **jamais** de `sudo pip install` |
| `python` reste sur l'ancienne version après `pyenv install` | La version n'a pas été sélectionnée | `pyenv global <version>` ou `pyenv local <version>` |
| `pyenv: command not found` dans un terminal neuf | Les lignes de configuration manquent dans `~/.bashrc` | Voir [étape 3](#3-configuration-du-shell) |
| Le venv casse après un `pyenv uninstall` | Le venv pointe en dur vers la version supprimée | Recréer le venv, puis `pip install -r requirements.txt` |

Le point le plus important de ce tableau : **`sudo pip install` n'est jamais la solution**. Un pip qui réclame les droits root signale presque toujours un venv non activé, et l'utiliser ainsi installe dans le Python du système — exactement ce que ce montage cherche à éviter.

## Aide-mémoire

```bash
pyenv versions              # versions installées ; * marque celle qui est active
pyenv install --list        # versions disponibles au téléchargement
pyenv global 3.12.9         # version par défaut, partout
pyenv local 3.13.15         # version pour ce dossier (écrit .python-version)
pyenv version               # version active ici, et pourquoi

python -m venv .venv        # créer un environnement virtuel
source .venv/bin/activate   # l'activer
deactivate                  # en sortir

which python                # quel python est réellement utilisé — en cas de doute
python -m pip list          # paquets installés dans l'environnement actif
```

`which python` est l'outil de diagnostic à réflexe : il répond à « d'où vient ce Python » et lève l'essentiel des confusions.

---

# Vocabulaire

| Terme | Signification |
|---|---|
| **venv** | *Virtual environment*. Un dossier (`.venv/`) contenant les paquets d'un seul projet, isolé des autres. |
| **pyenv** | Outil qui installe plusieurs versions de Python côte à côte et choisit laquelle est active. |
| **pip** | Le gestionnaire de paquets de Python. `python -m pip` garantit qu'on utilise celui du Python actif. |
| **shim** | Petit script intermédiaire placé par pyenv dans le `PATH`. Taper `python` appelle en réalité `~/.pyenv/shims/python`, qui redirige vers la bonne version. |
| **paquet `-dev`** | Sur Ubuntu, les fichiers nécessaires pour *compiler* du code contre une bibliothèque. pyenv compilant Python depuis les sources, ils doivent être présents **avant** l'installation. |
| **fact** (ansible) | Information collectée automatiquement sur la machine cible (version de Python, OS, mémoire…). |
| **idempotent** | Une opération qu'on peut relancer sans effet supplémentaire. C'est le principe d'ansible : la seconde exécution ne change rien. |
| **`site-packages`** | Le dossier où pip dépose les paquets installés. |

---

# Journal d'exécution

Le détail de ce qui a réellement été fait sur cette machine, à des fins de traçabilité et de reproductibilité.

## État initial

| Élément | Avant |
|---|---|
| Python système | 3.12.3 (`/usr/bin/python3`) |
| pyenv | absent |
| Config `~/.bashrc` | aucune ligne pyenv |
| Dépendances de build | 15 des 19 paquets manquants |

## Étapes réalisées

### 1. Dépendances système

pyenv ne télécharge pas un Python tout prêt : il le **compile depuis les sources**. Les bibliothèques ci-dessous doivent donc être présentes *avant*, sans quoi certains modules de Python seront absents à l'arrivée.

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

`~/.bashrc` étant lu à l'ouverture de chaque terminal, ces lignes rendent pyenv disponible en permanence. C'est ce qui manque quand un terminal neuf répond `pyenv: command not found`.

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

Ce contrôle mérite d'être fait systématiquement : sans `libssl-dev`, par exemple, la compilation réussit quand même mais le module `ssl` manque — et l'erreur n'apparaît que bien plus tard, au premier `pip install`, sous une forme peu explicite.

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

Les trois commandes pointent bien vers `~/projects/ansible-lab/.venv/bin/` — critère de la procédure rempli. C'est la preuve que l'isolation fonctionne : ansible s'exécute avec le Python du projet, pas celui du système.

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

> ⚠️ Le venv est lié en dur à `~/.pyenv/versions/3.12.9`. Un `pyenv uninstall 3.12.9` le casserait sans avertissement — il faudrait alors le recréer (`python -m venv .venv` puis `pip install -r requirements.txt`).

## Second projet : une autre version de Python

Ajouté le 2026-08-14, hors procédure initiale, pour valider la cohabitation de deux versions de Python sur la même machine.

```bash
pyenv install 3.13.15

mkdir -p ~/projects/python-lab && cd ~/projects/python-lab
pyenv local 3.13.15          # écrit .python-version dans le dossier
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install ansible
```

Aucun paquet apt supplémentaire n'a été nécessaire : les dépendances de build de l'étape 1 ont resservi telles quelles. Les modules critiques (`ssl`, `sqlite3`, `lzma`, `bz2`, `ctypes`, `readline`, `zlib`, `tkinter`) sont tous présents sur le 3.13.15.

À noter : `pip` était déjà en 26.2.1 dans le venv neuf — l'`ensurepip` de Python 3.13.15 embarque une version récente, l'`--upgrade` n'avait donc rien à faire.

Le venv a d'abord été créé nu, puis ansible y a été ajouté dans un second temps, afin de disposer du même ansible sur deux versions de Python et pouvoir y comparer un comportement.

### Les deux environnements côte à côte

| | `ansible-lab` | `python-lab` |
|---|---|---|
| Python | 3.12.9 | 3.13.15 |
| Sélection de version | global pyenv | `.python-version` (local) |
| ansible | 14.3.1 | 14.3.1 |
| ansible-core | 2.21.3 | 2.21.3 |
| `site-packages` | `.venv/lib/python3.12/` | `.venv/lib/python3.13/` |
| Vérifié en shell neuf | ✅ | ✅ |

Les deux projets embarquent exactement les mêmes versions d'ansible et de ses dépendances (`jinja2 3.1.6`, `cryptography 50.0.0`, `PyYAML 6.0.3`), seul l'interpréteur diffère.

`pyenv global` reste à **3.12.9** : `pyenv local` n'écrit qu'un fichier `.python-version` dans le dossier du projet. Entrer dans `~/projects/python-lab` bascule automatiquement sur 3.13.15, en sortir revient à 3.12.9. Vérifié après coup que `ansible-lab` est resté intact.

## Test fonctionnel : un playbook sur les deux versions

[`smoke-test.yml`](smoke-test.yml) est un playbook minimal qui exerce les briques usuelles d'ansible : collecte de facts, `set_fact`, boucle, module `copy`, `slurp`, filtre `b64decode` et `assert`. Il tourne sur `localhost` en `connection: local`, sans inventaire.

```bash
cd ~/projects/ansible-lab && source .venv/bin/activate   # ou python-lab
ansible-playbook ~/PROJETS/install-python-pyenv-venv/smoke-test.yml
```

### Résultats

| | Python 3.12.9 | Python 3.13.15 |
|---|---|---|
| Tâches OK | 7 | 7 |
| `changed` | 1 | 1 |
| `failed` | 0 | 0 |
| Interpréteur détecté | `ansible-lab/.venv/bin/python` | `python-lab/.venv/bin/python` |
| Fichier généré | `/tmp/smoke-3.12.9.txt` | `/tmp/smoke-3.13.15.txt` |
| `assert` sur le contenu relu | ✅ | ✅ |

Sorties identiques sur les deux versions. Chaque venv utilise bien son propre interpréteur — ansible ne retombe pas sur le Python système. Une seconde passe donne `changed=0`, l'idempotence est donc correcte.

### Note sur les facts dépréciés

À la première exécution, ansible-core 2.21 émettait un `DEPRECATION WARNING` sur `ansible_python_version` :

> `INJECT_FACTS_AS_VARS` default to `True` is deprecated, top-level facts will not be auto injected after the change. This feature will be removed from ansible-core version 2.24.

Le playbook a été réécrit en `ansible_facts['python_version']`, la forme pérenne, ce qui supprime l'avertissement. À garder en tête pour d'anciens playbooks qui utiliseraient encore les facts comme variables de premier niveau.

Les variables magiques `ansible_version` et `ansible_playbook_python` ne sont **pas** concernées : ce ne sont pas des facts, elles gardent leur forme actuelle.
