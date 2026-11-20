# Prompt Engineering et Context Engineering

**Fiche technique de synthèse — Développement logiciel**  
**Branche cible :** `sprint1`  
**Chemin du livrable :** `tutoriels/ia/agent-skills/prompt-engineering/prompt-engineering.md`

> **Objet de la fiche :** comprendre le fonctionnement général d'un LLM, structurer une consigne avec la méthode RIO, maîtriser des patterns de prompt et préparer un contexte fiable pour un agent de développement.

## 1. Comprendre le fonctionnement d'un LLM

Un **Large Language Model (LLM)** est un modèle entraîné sur de grands volumes de données textuelles (et, selon le modèle, d'autres modalités) afin de traiter et de générer du langage. Les modèles génératifs de type autoregressif produisent une séquence progressivement : à chaque étape, le modèle estime une distribution de probabilité sur les prochains tokens en tenant compte des tokens déjà présents dans le contexte. Il sélectionne ensuite un token selon la stratégie de génération, l'ajoute à la séquence et répète l'opération.

Un **token** est une unité de texte utilisée par le modèle. Il peut correspondre à un mot, une partie de mot, un signe de ponctuation ou un autre fragment. Le modèle ne raisonne donc pas directement sur les mots comme un humain : il traite une représentation tokenisée de l'entrée.

Dans les architectures modernes de type Transformer, le mécanisme d'attention aide le modèle à pondérer les relations entre les éléments du contexte. Le modèle apprend des régularités statistiques et structurelles à partir de ses données d'entraînement. Cette capacité lui permet de produire du texte et du code cohérents, mais **la probabilité d'une suite de tokens ne constitue pas une preuve de vérité**.

### 1.1 Pourquoi un LLM peut-il halluciner ?

Une **hallucination** est une sortie formulée de manière convaincante, mais fausse, inventée, non vérifiée ou insuffisamment fondée sur les informations disponibles. Elle peut concerner une explication, une référence, une API, une fonction ou un package.

Les causes possibles sont multiples :

- **Complétion probabiliste :** le modèle cherche une continuation plausible selon les régularités apprises, pas nécessairement une information vraie et vérifiée.
- **Contexte incomplet ou ambigu :** le modèle peut combler les informations manquantes par des suppositions.
- **Données d'entraînement imparfaites ou décalées :** une bibliothèque, une version ou une pratique peut être absente, rare ou obsolète.
- **Manque d'ancrage dans le projet réel :** sans accès aux fichiers, à la documentation ou à l'environnement, l'agent peut inventer des chemins, des méthodes ou des dépendances.
- **Génération en plusieurs étapes :** une erreur introduite tôt peut influencer les tokens suivants et se propager dans le code.

Dans le développement logiciel, ces erreurs peuvent prendre plusieurs formes : méthode inexistante, paramètre incorrect, package imaginaire, version incompatible, import manquant, hypothèse erronée sur le schéma de base de données ou code qui semble correct mais ne passe ni la compilation ni les tests.

Une étude présentée à USENIX Security 2025 a analysé 576 000 échantillons de code produits par 16 LLM. Elle rapporte des taux moyens d'hallucination de packages d'au moins 5,2 % pour les modèles commerciaux et 21,7 % pour les modèles open source dans le cadre précis de son protocole. Ces chiffres illustrent un risque mesuré dans cette étude ; ils ne doivent pas être interprétés comme un taux universel applicable à tous les modèles, langages ou prompts [5]. Une revue systématique de 2025 recense notamment le bruit des données, l'exposition aux erreurs pendant la génération et l'ancrage sémantique insuffisant parmi les facteurs associés aux hallucinations de code [6].

### 1.2 Réduire le risque dans un projet

Le prompt peut réduire les ambiguïtés, mais il ne garantit pas l'absence d'hallucination. Pour fiabiliser une contribution, il faut combiner plusieurs contrôles :

- fournir les fichiers réellement concernés et les versions des dépendances ;
- demander à l'agent de signaler explicitement les informations manquantes ;
- lui demander de vérifier les API dans la documentation correspondant à la version du projet ;
- limiter la modification aux fichiers autorisés ;
- lancer le linter, le compilateur, les tests et, lorsque c'est pertinent, les tests d'intégration ;
- examiner les changements (`diff`) avant de les accepter.

**Principe à retenir :** un code généré est une proposition à vérifier, pas une preuve de fonctionnement.

## 2. Prompt Engineering : concevoir une consigne exploitable

Le **Prompt Engineering** consiste à concevoir, organiser, tester et améliorer les instructions données à un LLM pour obtenir une sortie adaptée à une tâche. En développement logiciel, il sert à préciser le besoin, les contraintes techniques, le périmètre des modifications et les critères de validation.

Une consigne efficace réduit les interprétations inutiles. Elle ne remplace toutefois ni la spécification, ni les tests, ni la revue de code. Les guides de prompting recommandent notamment de définir l'identité ou le rôle, les instructions, les exemples et le contexte utile [1].

## 3. La méthode RIO

Dans cette fiche, **RIO** est utilisé comme cadre pratique de structuration d'une consigne :

### R — Rôle (Persona)

Attribuer au modèle une posture professionnelle et un domaine d'expertise précis.

Exemple : « Tu es un développeur web senior spécialisé en PHP et en conception d'API REST. »

Le rôle aide à calibrer le vocabulaire, le niveau de détail et l'angle d'analyse. Il ne donne cependant pas au modèle un accès réel à des connaissances ou à des fichiers qu'il ne possède pas.

### I — Instruction (Contexte et tâche)

Décrire ce qu'il faut faire avec des verbes d'action et un contexte vérifiable : objectif métier, fichiers concernés, environnement, versions, dépendances et limites.

Les contraintes négatives précisent ce qui est interdit : ne pas modifier les fichiers de configuration, ne pas ajouter de dépendance, ne pas supprimer de fonctionnalités, ne pas effectuer de commit. Ces consignes sont des garde-fous exprimés en langage naturel ; elles ne constituent pas à elles seules des permissions techniques inviolables.

### O — Output (Format attendu)

Définir le format de la réponse : langue, sections, longueur, code complet ou diff, chemins de fichiers, critères de validation et éléments à signaler. Un exemple de sortie attendue peut lever les ambiguïtés de format.

### 3.1 Exemple de consigne RIO pour une tâche web

```text
R — Rôle
Tu es un développeur web senior spécialisé en HTML, CSS et JavaScript accessibles.

I — Instruction
À partir du formulaire fourni, ajoute une validation côté client pour vérifier
que le champ e-mail n'est pas vide et respecte un format d'adresse courant.
Conserve les champs et le comportement existants.
Ne modifie que le fichier du formulaire.
N'ajoute aucune bibliothèque ni dépendance.
Ne prétends pas que la validation côté client remplace la validation serveur.
Si le fichier nécessaire manque, demande-le au lieu d'inventer son contenu.

O — Output
Présente d'abord un résumé de trois lignes maximum, puis le diff proposé.
Termine par une liste de vérifications manuelles à réaliser dans le navigateur.
Réponds en français.
```

## 4. Patterns fondamentaux de Prompt Engineering

Un **pattern** est une manière réutilisable de formuler une consigne pour résoudre un problème récurrent.

### 4.1 Persona Pattern

Définir un rôle et une expertise : « Agis comme un développeur backend expérimenté chargé d'une revue de sécurité. » Le rôle aide à orienter l'analyse, mais ne garantit pas l'exactitude technique.

### 4.2 Template / Format Pattern

Fournir un squelette de réponse ou de fichier. Exemple :

```text
## Résumé
## Fichiers concernés
## Modifications proposées
## Tests à exécuter
## Risques et limites
```

Ce pattern est utile pour obtenir des réponses homogènes, faciliter les revues et intégrer les résultats dans une documentation.

### 4.3 Constraint Pattern

Définir les limites et interdictions : périmètre de fichiers, versions, dépendances, taille de réponse, format, actions autorisées ou interdites. Les contraintes doivent être précises et non contradictoires.

### 4.4 Chain-of-Thought et Step-by-Step

Le principe consiste à découper une tâche complexe en étapes explicites : comprendre le besoin, inspecter les fichiers, proposer un plan, modifier, tester et résumer. Des travaux sur le Chain-of-Thought ont montré des gains sur certaines tâches de raisonnement avec des modèles suffisamment grands et des exemples adaptés [4]. Cela ne signifie pas que demander une longue explication améliore toute tâche de programmation.

En pratique, il est préférable de demander un **plan vérifiable, les étapes exécutées, les hypothèses et les résultats des tests**, plutôt que d'exiger la divulgation d'un raisonnement interne détaillé. Les étapes visibles facilitent le contrôle sans constituer une garantie de vérité.

### 4.5 Few-Shot Pattern

Ajouter quelques exemples entrée/sortie au prompt pour illustrer le comportement ou le format souhaité. Les exemples doivent être corrects, variés et proches du cas réel. Le modèle peut alors reproduire le schéma démontré [2][3].

### 4.6 Reflective Pattern

Demander à l'agent de relire son travail selon une grille : exigences respectées, fichiers modifiés, erreurs possibles, tests manquants et risques. Pour une tâche avec modification du dépôt, demander d'abord un plan et attendre l'accord avant d'appliquer les changements, si le processus l'exige.

L'auto-évaluation reste une vérification effectuée par le même modèle : elle peut manquer les mêmes erreurs que la première génération. Elle doit donc être complétée par des outils indépendants (tests, analyse statique, revue humaine).

## 5. Zero-Shot et Few-Shot : comparaison en développement web

### 5.1 Zero-Shot prompting

Le **Zero-Shot** demande une tâche sans fournir d'exemple de démonstration. Le modèle s'appuie sur ses connaissances préalables et sur les instructions.

Exemple :

```text
Écris une fonction JavaScript qui valide un champ e-mail.
Retourne uniquement le code, avec un nom de fonction explicite.
```

**Avantages :** prompt court, préparation rapide, adapté aux tâches courantes et bien spécifiées.  
**Limites :** le modèle doit deviner certaines conventions de nommage, le style du projet ou la structure exacte attendue si ces éléments ne sont pas précisés.

### 5.2 Few-Shot prompting

Le **Few-Shot** ajoute un ou plusieurs exemples d'entrée et de sortie pour montrer les conventions à reproduire.

Exemple :

```text
Écris une fonction JavaScript qui valide un champ e-mail.
Respecte ce style :

Exemple d'entrée : "contact@example.com"
Exemple de sortie : { valid: true, message: "Adresse valide" }

Exemple d'entrée : ""
Exemple de sortie : { valid: false, message: "Adresse obligatoire" }

Produis maintenant la fonction pour les autres cas, en conservant exactement
la structure de résultat { valid, message }.
```

**Avantages :** illustre les conventions, le contrat de sortie et le niveau de détail ; peut réduire les ambiguïtés.  
**Limites :** exemples erronés ou trop spécifiques peuvent induire des erreurs ; ils consomment des tokens et ne couvrent pas tous les cas.

### 5.3 Synthèse comparative

- Choisir **Zero-Shot** lorsque la tâche est simple, standard et déjà décrite avec précision.
- Choisir **Few-Shot** lorsque le format, le style du projet ou une règle métier doit être reproduit exactement.
- Dans les deux cas, fournir le contexte réel et vérifier le code généré. Les exemples guident le modèle, mais ne garantissent pas la correction.

## 6. Contraintes négatives et reproductibilité

Une contrainte négative indique ce que le modèle ne doit pas faire : « ne modifie pas les fichiers de configuration », « n'ajoute pas de dépendance », « ne crée pas de commit », « ne retourne pas de tableau Markdown ».

Ces contraintes peuvent améliorer la conformité observable en réduisant l'espace des réponses acceptables. Elles sont particulièrement utiles pour protéger le périmètre d'un changement et imposer un format. Cependant, elles restent des instructions probabilistes : le modèle peut les oublier, les interpréter différemment ou rencontrer une instruction contradictoire.

La **reproductibilité** ne dépend donc pas seulement du prompt. Elle dépend aussi du modèle et de sa version, des paramètres de génération, du contexte fourni, des outils disponibles, de l'état du dépôt et de l'environnement d'exécution. Pour renforcer la reproductibilité :

- versionner les prompts et les exemples ;
- conserver le même modèle et les mêmes paramètres pendant une comparaison ;
- utiliser un contexte et un jeu de tests identiques ;
- définir des critères de conformité mesurables ;
- répéter les essais et consigner les sorties ;
- déplacer les contraintes critiques vers des mécanismes techniques lorsque c'est possible : permissions, validation de schéma, tests automatisés, hooks et CI.

### 6.1 Protocole d'expérimentation proposé

**Objectif :** mesurer l'effet des exemples et des contraintes négatives sur la conformité d'une réponse d'agent.

**Tâche commune :** demander à l'agent d'ajouter une validation simple à un formulaire web fourni, sans changer son apparence.

**Conditions à comparer :**

- A — Zero-Shot : instruction claire, sans exemple.
- B — Few-Shot : même instruction avec deux exemples d'entrée/sortie.
- C — Contraintes négatives : instruction B complétée par des interdictions explicites (aucune dépendance, aucun changement CSS, aucun autre fichier modifié).

**Protocole :**

1. Utiliser le même modèle, la même version, les mêmes fichiers et les mêmes paramètres pour chaque condition.
2. Lancer chaque condition plusieurs fois (par exemple cinq essais), en repartant du même état initial.
3. Enregistrer les prompts et les réponses, sans corriger les sorties avant l'évaluation.
4. Évaluer chaque réponse selon les mêmes critères.

**Indicateurs à relever :**

- conformité au format attendu ;
- respect du périmètre des fichiers ;
- présence d'une dépendance non demandée ;
- validation des cas valides et invalides ;
- réussite des tests ;
- nombre de corrections humaines nécessaires.

**Grille de résultats à compléter après les essais :**

| Condition | Essais conformes | Fichiers hors périmètre | Tests réussis | Observations |
|---|---:|---:|---:|---|
| A — Zero-Shot | À mesurer | À mesurer | À mesurer | À compléter |
| B — Few-Shot | À mesurer | À mesurer | À mesurer | À compléter |
| C — Few-Shot + contraintes | À mesurer | À mesurer | À mesurer | À compléter |

> **Important :** aucun résultat chiffré n'est présenté ici comme une expérimentation déjà réalisée. Cette grille doit être remplie avec les observations obtenues lors des essais réels. L'hypothèse à vérifier est que les exemples peuvent améliorer la conformité au format et que les contraintes explicites peuvent réduire les changements hors périmètre, sans garantir une sortie identique à chaque exécution.

## 7. Context Engineering et fenêtre de contexte

Le **Context Engineering** consiste à sélectionner, organiser, maintenir et actualiser les informations réellement utiles au modèle pendant l'exécution. Il dépasse la rédaction de la consigne : il concerne l'ensemble des éléments accessibles à l'agent à un instant donné, comme les instructions, l'historique, les fichiers, les résultats d'outils et les documents externes [7].

### 7.1 Comprendre la fenêtre de contexte

La fenêtre de contexte est la quantité limitée de tokens que le modèle peut traiter à un moment donné. Elle peut contenir les instructions, la demande, les extraits de fichiers, les messages précédents et les réponses d'outils. Une fenêtre plus grande ne signifie pas que chaque information sera utilisée avec la même précision : un contexte trop long, redondant ou bruité peut réduire la pertinence des éléments importants [7].

Un modèle seul ne conserve pas automatiquement une mémoire personnelle complète entre deux sessions distinctes. Une application peut toutefois fournir une continuité grâce à un historique, des fichiers de mémoire, une base de données ou une récupération documentaire. Il faut donc distinguer la mémoire du modèle et la mémoire construite par le système qui l'entoure.

### 7.2 Application au développement logiciel

Pour travailler sur un dépôt, un contexte utile peut inclure :

- l'objectif de la tâche et les critères d'acceptation ;
- le README et les consignes du projet (`AGENTS.md`, `CONTRIBUTING.md` ou équivalent) ;
- l'arborescence pertinente et uniquement les fichiers nécessaires ;
- les versions du langage, du framework et des dépendances ;
- les conventions de code, les routes, modèles ou schémas concernés ;
- les tests existants et les derniers messages d'erreur utiles ;
- les décisions déjà prises et les actions interdites.

### 7.3 Bonnes pratiques de Context Engineering

- **Sélectionner :** donner les fichiers pertinents, pas tout le dépôt sans distinction.
- **Hiérarchiser :** séparer les règles prioritaires, le contexte métier, les données de référence et la demande actuelle.
- **Actualiser :** remplacer les anciennes versions de fichiers ou les informations obsolètes.
- **Récupérer à la demande :** rechercher les fichiers supplémentaires lorsqu'un besoin précis apparaît.
- **Compacter :** résumer l'historique en préservant les décisions, les contraintes et les tâches restantes.
- **Tracer :** conserver les versions de prompts, les références et les résultats de tests importants.
- **Protéger :** ne pas injecter de secrets, clés API, mots de passe ou données personnelles inutiles.

Le but n'est pas de fournir le maximum d'informations, mais le **meilleur contexte utile** pour la tâche en cours.

## 8. Synthèse personnelle

Je retiens que la qualité d'une réponse générée par un LLM dépend à la fois de la consigne et du contexte fourni. La méthode RIO m'aide à structurer une demande en trois parties simples : définir le rôle, expliquer précisément la tâche et imposer le format de sortie. Les patterns permettent ensuite d'adapter la consigne : exemples pour montrer le résultat attendu, contraintes pour délimiter le périmètre, étapes pour organiser le travail et réflexion critique pour contrôler le résultat.

Je retiens également qu'un LLM peut produire du code plausible mais incorrect. Je dois donc traiter chaque proposition comme une hypothèse à vérifier avec la documentation du projet, les outils de développement, les tests et une revue humaine. Enfin, le Context Engineering m'apprend à fournir à l'agent les bons fichiers et les bonnes règles, plutôt que de saturer sa fenêtre de contexte avec des informations inutiles.

**À personnaliser après les essais :** compléter la grille de la section 6.1 et ajouter une courte observation personnelle sur les différences réellement constatées entre Zero-Shot, Few-Shot et Few-Shot avec contraintes négatives.

## 9. Références documentaires

1. OpenAI, [Prompt engineering — OpenAI API documentation](https://developers.openai.com/api/docs/guides/prompt-engineering), consulté le 20 novembre 2026. Guide sur l'organisation des instructions, le contexte, les exemples et le Few-Shot.
2. Tom B. Brown et al., [Language Models are Few-Shot Learners](https://arxiv.org/abs/2005.14165), 2020. Étude fondatrice sur les capacités Few-Shot de GPT-3 sans mise à jour des poids pendant les tâches présentées.
3. OpenAI, [Prompt engineering](https://developers.openai.com/api/docs/guides/prompt-engineering#few-shot-learning), documentation sur les exemples entrée/sortie et le Few-Shot.
4. Jason Wei et al., [Chain-of-Thought Prompting Elicits Reasoning in Large Language Models](https://proceedings.neurips.cc/paper/2022/hash/9d5609613524ecf4f15af0f7b31abca4-Abstract-Conference.html), NeurIPS 2022.
5. Joseph Spracklen et al., [We Have a Package for You! A Comprehensive Analysis of Package Hallucinations by Code Generating LLMs](https://www.usenix.org/conference/usenixsecurity25/presentation/spracklen), USENIX Security 2025.
6. Cuiyun Gao et al., [A Systematic Literature Review of Code Hallucinations in LLMs](https://arxiv.org/abs/2511.00776), 2025.
7. Anthropic, [Effective context engineering for AI agents](https://www.anthropic.com/research/effective-context-engineering-for-ai-agents), 29 septembre 2025.
