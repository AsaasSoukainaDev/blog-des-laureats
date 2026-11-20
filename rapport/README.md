# Rapport du projet Blog des lauréats

Ce dossier héberge le rapport technique et académique du projet Blog des lauréats.

## 1. Organisation du rapport

Le rapport est organisé en plusieurs chapitres indépendants dans le sous-dossier `chapitres/`, afin de faciliter la rédaction, la modification et la maintenance du contenu.

- `01-remerciements.md` : Remerciements.
- `02-introduction.md` : Présentation générale et introduction du projet.
- `03-contexte-projet.md` : Contexte du projet.
- `04-objectifs.md` : Objectifs de formation et objectifs du projet.
- `05-cahier-des-charges.md` : Besoins fonctionnels et non fonctionnels.
- `06-methodologie.md` : Méthodologie de travail (Scrum, 2TUP et Design Thinking).
- `07-planification.md` : Organisation et planification du projet.
- `08-branche-fonctionnelle.md` : Analyse fonctionnelle, carte d'empathie, définition du problème, idéation et diagramme de cas d'utilisation.
- `09-branche-technique.md` : Stack technologique et architecture logicielle.
- `10-conception.md` : Conception du système, diagramme de classes et maquettes.
- `11-realisation.md` : Réalisation et présentation des interfaces.
- `12-conclusion.md` : Bilan des réalisations et perspectives.

## 2. Compilation automatique

Le fichier consolidé `rapport-blog.md` est généré automatiquement à partir des chapitres à l'aide d'un script Node.js.

- Script : `concat-chapitres.js`
- Commande directe : `node concat-chapitres.js`
- Ou via npm : `npm run build`

## 3. Guide de rédaction

- Rédiger ou modifier les chapitres dans le sous-dossier `chapitres/`.
- Respecter l'ordre des chapitres grâce à leur numérotation.
- Chaque fichier de chapitre doit commencer par son titre de niveau 1 (`# Titre du chapitre`).
- Lancer le script de compilation après chaque modification pour mettre à jour `rapport-blog.md`.
- Respecter les règles générales : zéro tableau Markdown, liens relatifs et français professionnel.