# Dotfiles (chezmoi)

Configuration personnelle gérée avec [chezmoi](https://www.chezmoi.io/).
Les paquets Homebrew sont décrits dans le `Brewfile` et installés automatiquement.

## Première installation sur une nouvelle machine

```bash
brew install chezmoi
chezmoi init --apply git@github.com:renaudmathieu/dotfiles.git
```

## Ajouter un paquet (formula ou cask) et le propager

### Sur la machine source

```bash
# 1. Installer le paquet normalement
brew install --cask raycast        # ou: brew install <formula>

# 2. Régénérer le Brewfile depuis l'état réel de la machine
brew bundle dump --file=~/Brewfile --force

# 3. Re-synchroniser le Brewfile vers le source state chezmoi
chezmoi add ~/Brewfile

# 4. Commit & push
chezmoi cd
git add .
git commit -m "Add raycast cask"
git push
exit
```

> **Étape 3 = la plus importante.** `brew bundle dump` modifie `~/Brewfile`
> (le target), mais pas la copie dans le source state chezmoi (ce qui est
> versionné par git). Sans `chezmoi add`, on pousse l'ancien Brewfile.

### Sur les autres machines

```bash
chezmoi update -v
```

Cette commande fait un `git pull` puis applique le source state. Le contenu
du `Brewfile` ayant changé, le hash dans le script `run_onchange_` change
aussi, donc chezmoi ré-exécute `brew bundle install` automatiquement. Le
nouveau paquet est installé sans action manuelle.

## Vérifications

```bash
chezmoi diff                       # avant un commit : rien sur le Brewfile = OK
chezmoi managed | grep Brewfile    # confirme que le Brewfile est tracké
brew list --cask | grep raycast    # confirme l'installation côté machine 2
```

## Le réflexe anti-piège

Toujours lancer `chezmoi diff` **avant** de commiter.
S'il affiche des différences sur le `Brewfile`, c'est que `chezmoi add` a été
oublié. S'il n'affiche rien, le push est sûr.

## Fonctionnement interne

Le fichier `run_onchange_install-packages.sh.tmpl` contient :

```bash
#!/bin/bash
# Brewfile hash: {{ include "Brewfile" | sha256sum }}
brew bundle install --file=~/Brewfile
```

Le commentaire avec le hash n'est pas cosmétique : quand le `Brewfile`
change, son hash change, donc le contenu rendu du script change, et chezmoi
ré-exécute le script (préfixe `run_onchange_`).
