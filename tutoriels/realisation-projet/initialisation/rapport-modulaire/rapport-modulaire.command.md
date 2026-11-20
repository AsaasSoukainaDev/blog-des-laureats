

\# Journal des commandes - Rapport modulaire



\## 1. Creation de l'arborescence



```cmd

mkdir rapport\\chapitres

```



\## 2. Creation des fichiers Markdown



```cmd

notepad rapport\\README.md

notepad rapport\\chapitres\\README.md

notepad rapport\\chapitres\\01-introduction.md

notepad rapport\\chapitres\\02-conclusion.md

```



\## 3. Creation du script et de la configuration Node.js



```cmd

notepad rapport\\concat-chapitres.js

notepad rapport\\package.json

```



\## 4. Compilation du rapport avec Node.js



```cmd

cd rapport

node concat-chapitres.js

```



\## 5. Compilation du rapport avec npm



```cmd

npm run build

cd ..

```



\## 6. Verification des livrables



```cmd

tree rapport /F

dir rapport

git status

```



