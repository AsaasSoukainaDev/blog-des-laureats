---
marp: true
theme: default
paginate: true
size: 16:9
header: "Solicode — IA et développement logiciel"
footer: "Prompt Engineering & Context Engineering"
---

<!-- _class: lead -->

# Prompt Engineering
## Comprendre les LLM, structurer les prompts et maîtriser le contexte

- **Apprenant :** Asaas soukayna
- **Encadrant :** ESSARRAJ Fouad
- **Établissement :** Solicode Tanger
- **Branche :** `sprint1`
- **Date :** 20 novembre 2026

---

# Objectifs de la restitution

À la fin de cette présentation, je dois pouvoir :

- expliquer simplement comment un LLM génère du texte et du code ;
- identifier les risques d'hallucination ;
- structurer une consigne avec la méthode **RIO** ;
- choisir entre Zero-Shot et Few-Shot ;
- utiliser des patterns et des contraintes de façon pertinente ;
- préparer un contexte utile pour un agent de développement.

---

# 1. Comment fonctionne un LLM ?

- Le texte est découpé en **tokens** (mots ou fragments de mots).
- Le modèle analyse les tokens du contexte avec son architecture, généralement de type Transformer.
- À chaque étape, un modèle autoregressif estime une distribution de probabilité sur le **prochain token**.
- Le token sélectionné est ajouté à la séquence ; le processus recommence.

<small>Un token probable n'est pas nécessairement une information vraie. Le modèle génère une suite plausible à partir de ce qu'il a appris et du contexte fourni.</small>

---

# 2. Pourquoi les LLM hallucinent-ils ?

Une hallucination est une sortie convaincante, mais fausse ou non fondée.

**Exemples en développement :**

- une méthode ou une API inexistante ;
- un package qui n'existe pas ;
- une version ou un paramètre incorrect ;
- un chemin ou un schéma de base de données inventé ;
- du code plausible qui ne compile pas ou échoue aux tests.

**Pourquoi ?** Contexte incomplet, données imparfaites, versions inconnues et complétion probabiliste.

---

# 3. Un risque mesuré dans le code

Une étude USENIX Security 2025 a analysé **576 000 échantillons** générés par 16 LLM.

Elle rapporte, dans son protocole :

- au moins **5,2 %** de packages hallucinés en moyenne pour les modèles commerciaux ;
- **21,7 %** pour les modèles open source.

Ces chiffres concernent cette étude et son protocole : ils ne constituent pas un taux universel.

**Conséquence :** vérifier les dépendances, la documentation, les versions, la compilation et les tests.

<small>Source : Spracklen et al., USENIX Security 2025.</small>

---

# 4. Prompt Engineering : le contrat de la tâche

Le Prompt Engineering consiste à concevoir et améliorer les instructions fournies au modèle.

Un bon prompt précise :

- **qui** doit répondre (rôle) ;
- **quoi** réaliser et dans quel contexte ;
- **ce qui est autorisé ou interdit** ;
- **comment** présenter le résultat ;
- **comment vérifier** que le travail est correct.

Un prompt réduit les ambiguïtés, mais ne remplace pas les tests ni la revue de code.

---

# 5. La méthode RIO

<article>

**R — Rôle (Persona)**  
Définir l'expertise et la posture : « développeur web senior spécialisé en PHP ».

**I — Instruction (Contexte et tâche)**  
Décrire l'objectif, les fichiers concernés, les versions, les étapes et les contraintes négatives.

**O — Output (Format attendu)**  
Préciser la langue, les sections, le type de sortie, les limites de longueur et les vérifications.

</article>

**RIO transforme une demande vague en consigne structurée et vérifiable.**

---

# 6. Exemple rapide de prompt RIO

**R — Rôle**  
Tu es un développeur web expérimenté en HTML, CSS et JavaScript.

**I — Instruction**  
Ajoute une validation d'e-mail au formulaire fourni. Conserve les champs existants. Ne modifie pas le CSS. N'ajoute aucune dépendance. Si le fichier manque, demande-le.

**O — Output**  
Présente un résumé, le diff proposé et les tests manuels à effectuer. Réponds en français.

---

# 7. Patterns fondamentaux

- **Persona :** préciser le rôle et l'expertise.
- **Template / Format :** fournir un squelette de sortie.
- **Constraint :** limiter les fichiers, dépendances et actions.
- **Step-by-Step :** décomposer le travail en étapes vérifiables.
- **Few-Shot :** montrer des exemples entrée/sortie.
- **Reflective :** demander un audit final à partir de critères.

L'auto-évaluation ne remplace pas les tests indépendants ni la revue humaine.

---

# 8. Zero-Shot ou Few-Shot ?

**Zero-Shot**

- Aucun exemple fourni.
- Plus rapide et plus court.
- Pertinent pour une tâche standard bien décrite.

**Few-Shot**

- Quelques exemples entrée/sortie sont inclus.
- Le format et les conventions sont plus explicites.
- Utile pour reproduire un style ou une règle métier.

**Limite commune :** un exemple incorrect peut orienter le modèle vers une mauvaise réponse. Les deux approches nécessitent une validation.

---

# 9. Contraintes négatives et reproductibilité

Exemples :

- « Ne modifie que le fichier du formulaire. »
- « N'ajoute aucune dépendance. »
- « Ne touche pas aux fichiers de configuration. »
- « Ne crée aucun commit. »

Elles clarifient les limites, mais restent des instructions en langage naturel.

Pour sécuriser une règle critique : compléter le prompt par des permissions, des validations, des tests, des hooks ou une CI.

La reproductibilité dépend aussi du modèle, de sa version, des paramètres, du contexte et de l'environnement.

---

# 10. Expérimentation à réaliser

**Même tâche :** ajouter une validation à un formulaire web fourni.

Trois conditions à comparer :

1. **A — Zero-Shot :** instruction sans exemple.
2. **B — Few-Shot :** même instruction avec deux exemples.
3. **C — Few-Shot + contraintes :** exemples et interdictions explicites.

Réaliser plusieurs essais depuis le même état initial et conserver chaque sortie.

**Mesures :** format conforme, fichiers modifiés, dépendances ajoutées, tests réussis et corrections humaines.

<small>Les résultats doivent être renseignés après les essais réels ; aucun chiffre n'est inventé dans cette présentation.</small>

---

# 11. Context Engineering

Le Prompt Engineering travaille surtout la formulation des instructions. Le **Context Engineering** organise l'ensemble des informations disponibles au modèle pendant l'exécution :

- consignes et historique utiles ;
- fichiers du dépôt et documentation ;
- versions et dépendances ;
- outils disponibles et résultats de tests ;
- décisions déjà prises et tâches restantes.

La fenêtre de contexte est limitée. Un contexte trop volumineux ou redondant peut noyer les informations essentielles.

---

# 12. Appliquer le Context Engineering au code

**À fournir :**

- l'objectif et les critères d'acceptation ;
- les consignes du dépôt (`README`, `AGENTS.md`, etc.) ;
- les seuls fichiers pertinents ;
- les versions du langage et des dépendances ;
- les tests existants et les erreurs utiles.

**À éviter :**

- injecter tout le dépôt sans sélection ;
- conserver des consignes obsolètes ;
- inclure des secrets ou des mots de passe ;
- confondre historique de conversation et mémoire persistante.

Un système peut créer une mémoire entre sessions en sauvegardant puis en réinjectant des notes ou des fichiers ; ce n'est pas une mémoire implicite garantie du modèle seul.

---

# 13. Conclusion — Ce que je retiens

- RIO m'aide à formuler une demande claire : **Rôle, Instruction, Output**.
- Les patterns rendent les consignes réutilisables et plus précises.
- Few-Shot aide à montrer un exemple du résultat attendu.
- Les contraintes négatives réduisent les ambiguïtés, mais ne sont pas une protection technique absolue.
- Le Context Engineering fournit à l'agent les informations réellement utiles.
- Le code généré doit toujours être contrôlé par la documentation, les tests et une revue.

**Un bon résultat dépend du prompt, du contexte et de la validation.**

---

# Références

- OpenAI — [Prompt engineering](https://developers.openai.com/api/docs/guides/prompt-engineering)
- Brown et al. — [Language Models are Few-Shot Learners (2020)](https://arxiv.org/abs/2005.14165)
- Wei et al. — [Chain-of-Thought Prompting (NeurIPS 2022)](https://proceedings.neurips.cc/paper/2022/hash/9d5609613524ecf4f15af0f7b31abca4-Abstract-Conference.html)
- Spracklen et al. — [Package Hallucinations (USENIX Security 2025)](https://www.usenix.org/conference/usenixsecurity25/presentation/spracklen)
- Gao et al. — [Code Hallucinations: Systematic Review (2025)](https://arxiv.org/abs/2511.00776)
- Anthropic — [Effective Context Engineering for AI Agents (2025)](https://www.anthropic.com/research/effective-context-engineering-for-ai-agents)
