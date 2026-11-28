# ia-coding-knowledge-base

This repository gathers all my knowledge and best practices in IA coding

## Models

The only models I use are those I can host myself. Here they are:

- Qwen 3.8 27b
- Qwen 3.6 35b MoE
- Gemma 4 26b MoE

## Environnement

I use [opencode](https://opencode.ai) as agents orchestrator and chat software. All the examples and settings files are for this software but they have their equivalent in *almost* any other orchestrator.

The models are configured as agents with specific roles:
- **Coordinator** - The main LLM responsible of understanding the needs and invokes the other agents.
- **Coordinator-plan** - The coordinator in plan mode (read only).
- **Architect** - In charge of the architecture of the project.
- **Developer** - In charge of the development of the project.
- **Debugger** - In charge of the diagnosis of the problems.
- **Reviewer** - In charge of the review of the source code produced.
- **Scribe** - In charge of the documentation of the project.

## Structure du projet

Le projet de développement a une structure qui donne une large place à la documentation. Celle-ci a une importance **critique** dans le succès de la phase de codage. En effet, toute la connaissance liée au projet doit être disponible pour les agents de codage, selon leur spécialité, dans des documents facilement accessibles et correctement structurés.

Voici l'organisation typique de mes projets :
```
project/
|-- .opencode/
|   |-- agents/
|   |   |-- architect.md
|   |   |-- coordinator.md
|   |   |-- coordinator-plan.md
|   |   |-- debugger.md
|   |   |-- developper.md
|   |   |-- reviewer.md
|   |   |-- scribe.md
|-- docs/
|   |-- diagrams/
|   |-- plans/
|   |   |-- refactoring/
|   |   |   |-- refactoring-plan.md
|   |   |   |-- test-run-1.md
|   |   |   |-- test-run-2.md
|   |-- architecture/
|   |   |-- components/
|   |   |   |-- component1.md
|   |   |   |-- component2.md
|   |-- decisions/
|   |   |-- 0001-decision1.md
|   |   |-- 0002-decision2.md
|   |-- design-rules.md
|   |-- overview.md
|   |-- development/
|   |   |-- building.md
|   |   |-- coding-rules.md
|   |   |-- packaging.mg
|   |   |-- testing.mg
|   |-- security/
|   |   |-- security-model.md
|   |-- specifications/
|   |   |-- features.md
|   |   |-- api.md
|   project-status.md
|-- sources/
|   AGENTS.md
```

## Organisation des agents

L'objectif de cette tâche est de maximiser la vitesse d'analyse et d'exécution, tout en minimisant l'impact sur les tokens consommés et la mémoire utilisée sur le serveur d'inférence.

L'organisation des agents permet de déléguer les bonnes tâches aux bons agents et de router les commandes aux bon modèles, en fonction de leurs domaines d'excellence respectifs.

Chaque agent doit être vu comme un membre de l'équipe produit ayant des compétence spécifiques qui sont utiles au projet. 

La notion d'agent est indépendante de la notion de modèle. La relation entre agent et modèle est de l'ordre `agent` 1..1 ---- 1..n `modèle`. Tous les agents peuvent être associés au même modèle le cas échéant.

Voici un exemple de mappage agent - modèle :

| Agent | Besoin | Modèle | Réglages |
|--|--|--|--|
| Architecte | Raisonnement profond, planification, vision globale | Qwen 3 32b | Thinking activé, température : 1.0 |
| Coordinateurs | Routage rapide, gestion d'état, appels d'outils | Qwen 3 8b | Thinking désactivé, température : 0.7 |
| Débogeur | Capacité de raisonnement étape par étape | Qwen 3 32b | Thinking activé, température : 1.0 |
| Développeur | Génération de code rapide, respect strict de la syntaxe | Qwen 3 32b | Thinking désactivé, température : 0.7 |
| Relecteur | Analyse critique, détection de bugs et failles de sécurité | Deepseek-R1-Distill (14b) | Thinking activé, température : 1.0 |
| Scribe | Synthèse, mise à jour de documentation, mémoire long terme | Gemma 3 (12b) | Thinking désactivé, température : 0.7 |

### Routage

Le principe du routage est de permettre au `coordinator` de déléguer à un ou plusieurs agents certaines tâches en considérant qu'elle est dans le domaine d'expertise de l'agent.

Dans Opencode, voici les fichiers que j'utilise pour mes agents.

### Coordinator in plan mode

[.opencode/agents/coordinator-plan.md](examples/.opencode/agents/coordinator-plan.md)


### Coordinator

[.opencode/agents/coordinator.md](examples/.opencode/agents/coordinator.md)

### Architect

[.opencode/agents/architect.md](examples/.opencode/agents/architect.md)

### Debugger

[.opencode/agents/debugger.md](examples/.opencode/agents/debugger.md)

### Developer

[.opencode/agents/developer.md](examples/.opencode/agents/developer.md)

### Reviewer

[.opencode/agents/reviewer.md](examples/.opencode/agents/reviewer.md)

### Scribe

[.opencode/agents/scribe.md](examples/.opencode/agents/scribe.md)

## RAG

Si le projet s'appuie sur une documentation copieuse et externe (spécifications, référentiels normatifs, réglementations, documents d'entreprise, etc) il est préférable de déclarer un RAG permettant aux agents de requêter la documentation aussi souvent que nécessaire sans impacter les performances.

## AGENTS.md

Le fichier `AGENTS.md` est très important pour déclarer le contexte technique du projet, son organisation et ses règles de fonctionnement, mais également pour déclarer les différents éléments du corpus documentaire que les agents pourront aller chercher au besoin.

Ce fichier est automatiquement et systématiquement inséré dans le contexte de la conversation par l'orchestrateur.

Voici un exemple de fichier [`AGENTS.md`](examples/Safecor.README.md) que j'utilise pour [Safecor](https://github.com/TristanIsrael/Safecor). Il contient les sections suivantes :

- Architecture - a short description of the architecture and a link to the architecture document.
- Directory layout - a description of the project structure.
- Tech stack - a description of the technical stack.
- Documentation - a description of the documents the agents can use for their work and different rules for the documentation generation.
- Installation packages - a description of the packaging solution used.
- Build - a description of the build process and actions to do before and during the build.
- Coding standards - a description of the coding standards and the related document.
- Security rules - a description of the security rules and a link to the related document.
- Linting and static analysis - a description of the static analysis process.
- Issue tracking - a list of rules for the issue tracking.
- Quality gates - a list of rules to apply before the push to git.

## Scenario

### 1 - Planification

When I have an improvement to make or a bug to fix I start with a discussion in `Plan mode`: I explain my objectives and the modification to the architecture if needed. Ensuite je discute avec l'agent jusqu'à ce que je sois sûr qu'il a bien compris mes objectifs et les contraintes techniques que je lui impose.

Dans un second temps je lui demande de mettre à jour la documentation du projet, en particulier les documents d'architecture et la description des fonctions du système.

*La mise à jour des documents est importante à la fois pour partager le contexte avec les autres contributeurs au projet, qu'ils utilisent une IA ou non, mais également pour être libre de changer de poste de développement à n'importe quel moment, et en cas de problème ou de plantage sur le poste.*

Enfin je lui demande de rédiger un plan de réalisation dans le dossier `docs/plans`. Ce plan me permettra de suivre l'avancée des travaux et permettra de reprendre le travail où il en était en cas de changement d'environnement de travail ou de transfert à une autre personne.

Dans l'idéal, un plan est accompagné de tests de validation, la création d'un sous-répertoire dans `docs/plans` est alors préférable afin de regrouper la fiche de suivi du refactoring et les fiches de suivi des tests. Par exemple pour un plan de refactoring : `docs/plans/storage-refactoring/storage-refactoring-plan.md`.

### 2 - Codage

Une fois que les documents sont à jour, et le plan écrit, je demande au `coordinateur` de lancer les travaux du plan et mettre à jour le plan à chaque étape.

Au fur et à mesure de l'avancée des travaux, les sous-agents vont être mis à contribution par le `coordinateur`. D'abord l'`architecte` puis le `developpeur` puis le `reviewer` qui va contrôler les travaux, et remonter des observations au `coordinateur` qui va éventuellement remobiliser le `developpeur` et le `reviewer`, et ainsi de suite. 

Le `coordinateur` me fait enfin un rapport sur les modifications réalisées et les résultats des tests automatisés, je peux tout accepter ou demander des changements. 

Le `coordinateur` me demande si je souhaite committer le code et le pousser, j'accepte et le `scribe` se met automatiquement en route pour vérifier l'état de la documentation et la mettre à jour le cas échéant.

### 3 - Validation

La phase suivante est la validation des réalisations des agents. Je demande au `coordinateur` de rédiger un plan de tests, ou s'il existe déjà, de le mettre à jour.

Le plan de validation est alors écrit dans un fichier `docs/plans/storage-refactoring/storage-refactoring-validation-plan.md`.

Chaque phase de tests doit être consignée dans une fiche dédiée. En fonction de l'avancement des tests, le `coordinateur` doit proposer un ensemble d'actions et de résultats attendus dans la fiche de test. Les fiches de tests sont écrites dans le même répertoire que le plan, par exemple : `docs/plans/storage-refactoring/test-run-1.md`.

A chaque itération de tests j'écris les résultats des tests (OK | KO), les sorties et mes observations dans la fiche correspondant au run de test. 

A la fin de mon run de test, je passe en mode Plan et j'indique qu'il peut lire la fiche et me faire une proposition de corrections. Je lui demande d'écrire les propositions et les analyses dans la fiche du run, pour le suivi.

Et ainsi de suite, jusqu'à réussite complète du plan de validation.

## Bonnes pratiques

- Structurer et organiser la documentation en documents d'une taille raisonnable (environ 200 lignes) pour permettre aux agents **concernés** de consulter certaines documentations et pas toutes, tout en optimisant la vitesse de traitement et l'utilisation des tokens.

- Si l'IA ne le fait pas elle-même, lui demander d'indiquer la date et l'heure de dernière mise à jour de chaque document. Cela permet à chaque membre de l'équipe de connaître la fraicheur du document.

- Il est important de demander régulièrement à l'assistant de mettre à jour l'ensemble de la documentation. Parfois un agent peut oublier une partie de sa mission et petit à petit la documentation n'est plus à jour avec le code ou les décisions prises. Les décisions doivent être écrites dans le registre (`docs/decisions`) pour être réutilisées lors des futures analyses.

- Toujours committer avec des commentaires, l'IA le fait spontanément, et pousser régulièrement dans des branches. Cela permet aux agents d'analyser les actions passées et d'identifier rapidement le moment où une régression a été introduite.

- Définir le mapping des agents et des modèles dans le fichier `opencode.jsonc` de l'environnement de développement et les agents dans les fichiers `.opencode/agents/*.md` du projet. Cela permet de gérer la définition des agents dans le source, mais le mapping dépend des modèles disponibles dans chaque environnement de développement.
