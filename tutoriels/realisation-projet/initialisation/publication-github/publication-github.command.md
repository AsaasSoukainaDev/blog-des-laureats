# Publication GitHub — Journal des commandes

## 1. Vérification de l'authentification

```bash
gh auth status
```

## 2. Initialisation du projet

```cmd
type nul > .gitignore
git init
```

## 3. Création et vérification de la branche

```bash
git checkout -b sprint1
git branch
```

## 4. Création de l'arborescence

```cmd
mkdir tutoriels\realisation-projet\initialisation\publication-github
type nul > tutoriels\realisation-projet\initialisation\publication-github\publication-github.command.md
```

## 5. Installation de GitHub CLI

```cmd
winget list --id GitHub.cli
winget install --id GitHub.cli -e
gh --version
```

## 6. Authentification GitHub

```bash
gh auth login
gh auth status
```