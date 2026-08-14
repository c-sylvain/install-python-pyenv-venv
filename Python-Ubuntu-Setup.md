---
tags:
  - type/procedure
  - status/draft
  - stack/python
  - stack/linux
created: 2026-05-16
---

# Setup Python Ubuntu

## Objectif

Installer un environnement Python complet sur Ubuntu vierge : pyenv, venv, et un premier projet fonctionnel.

## Prérequis

- Ubuntu vierge avec accès sudo
- Connexion internet

## Variables à adapter

| Variable | Valeur exemple | Description |
|---|---|---|
| VERSION_PYTHON | 3.12.9 | Version Python à installer via pyenv |
| NOM_PROJET | ansible-lab | Nom du dossier projet |

## Procédure

### 1. Installer les dépendances système

```
sudo apt update
sudo apt install -y \
  build-essential \
  curl \
  git \
  libssl-dev \
  zlib1g-dev \
  libbz2-dev \
  libreadline-dev \
  libsqlite3-dev \
  wget \
  llvm \
  libncursesw5-dev \
  xz-utils \
  tk-dev \
  libxml2-dev \
  libxmlsec1-dev \
  libffi-dev \
  liblzma-dev \
  python3-openssl \
  python3-pip \
  python3-venv
```

### 2. Installer pyenv

```
curl https://pyenv.run | bash
```

### 3. Configurer le shell

```
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init --path)"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
exec $SHELL
```

### 4. Vérifier pyenv

```
pyenv --version
```

### 5. Installer Python et définir la version globale

```
pyenv install 3.12.9
pyenv global 3.12.9
python --version
```

### 6. Créer un projet avec venv

```
mkdir -p ~/projects/ansible-lab
cd ~/projects/ansible-lab
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install ansible
```

## Vérification

```
which python
which ansible
ansible --version
```

Les trois commandes doivent pointer vers `~/projects/ansible-lab/.venv/bin/`.

## Liens

[[Python-Pyenv]] - [[Python-Venv]] - [[pyenv-cheatsheet]] - [[python-venv-lifecycle]]

## Sources

Conversation IA 2026-04-14
