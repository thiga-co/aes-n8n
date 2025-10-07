# Exercice 2 - Agent de création d'US

> **Objectif** : Echange au sujet de ton **initiative** avec l'agent Product Owner et connecte le à JIRA.
> **Durée** : 30 min 

---

## 1) Pré-requis & vue d’ensemble

### Pré-requis

- Finir l'exercice 1 et utiliser le même workspace

### Architecture du flow

```
OpenAI Chat Model ──► (AI Language Model) ──► PO Agent ◄── (AI Memory) ── Simple Memory
													│
													└── (AI Tool) ──► Jira Tool (Create Story)

```

---

## 2) Node — **AI Agent **

### 🎯 Rôle

Agent conversationnel connecté au chat de n8n. Il permet de mettre en musique un LLM, des outils et une gestion de la mémoire.

### 🔎 Où l'ajouter

- Déconnecte le Chat du workflow précédent (🗑️) et ajoute un noeud (+).
- **Add node → AI → AI Agent**.

### ⚙️ Configuration

- Renomme le nœud en \`PO Agent\`;
- * **Source for Prompt (User Message)** : `Connected Chat Trigger Node` pour récupérer le message du chat. 
- **Option -> Enable Streaming** : Activer pour voir les réponse de l'agent s'afficher progressivement.
- **Option -> System Message** : Recopie le sytem prompt ci-dessous en modifiant la desciption du Produit. Tu peux le customiser comme tu veux ;).

```markdown
# System Prompt – Agent Product Owner Confirmé
Tu es un Product Owner confirmé qui accompagne l’utilisateur pour clarifier et creuser son besoin afin de le traduire en User Stories JIRA.
Tu dois adopter une posture de facilitateur et expert en Product Management focus sur la valeur.

# Voici une description de ton Produit :
__Desciption du Produit__

# Rôle et comportement
- Tu aides l’utilisateur à préciser son besoin sans jargon inutile.
- Tu poses uniquement une question par message.
- Concentre-toi sur la nouvelle initiative à rajouter dans le produit existant.

# Règles d’interaction
- Maximum 5 tours de conversation (questions-réponses).
- À chaque étape, tu cherches à obtenir une information utile pour rédiger des User Stories (Summary + Description). Si, avant la fin des 5 échanges, tu as assez d’informations pour créer des US clairs et cohérents, tu passes directement à la génération des User Stories. Tu ne demandes pas la permission explicite de créer les US : tu le fais automatiquement quand c’est prêt.

# Sortie attendue :
À la fin de la conversation (ou avant si toutes les infos sont claires), tu appelles automatiquement le tool JIRA pour créer entre 1 et 5 User Stories maximum.

Chaque US doit contenir :
 - Summary clair et concis.
 - Description rédigée au format En tant que [utilisateur], je veux [fonctionnalité] afin de [valeur ajoutée].
```
---

## 3) Ajouter le LLM à l'agent

Dupliquer le  **OpenAI Chat Model** (renommé en : Ecrire les US) de l'exo 1 et le connecter à l'agent de la même façon.

---

## 4) Ajouter la **Memory**

### 🎯 Rôle
Par defaut, un agent n'a pas de mémoire de la conversation. Il faut donc sauvegarder l'historique des échanges entre l'utilisateur et l'agent pour avoir le contexte.

### 🔎 Où l'ajouter
- Clique sur `+` Memory au niveau de l'agent.
- Selectionne : **Simple Memory**

### ⚙️ Configuration
- **Session ID** : `Connected Chat Trigger node`, n8n génère un id de session de la conversation pour retrouver les messages.
- **Context Window Length** : `30`. On enregistre les 30 derniers échanges.

---

## 5) Tool — **Create JIRA issues**

### 🎯 Rôle
L'agent est capable d'appeler en autonomie les outils (Tools) qui lui sont configuré. On lui permet de créer les User Stories dans JIRA quand il evaluera que c'est pertinent.

### 🔎 Où le trouver / ajouter
- Clique sur `+` Tool au niveau de l'agent
- Recherche : **Jira Software Tool**

### ⚙️ Configuration
- **Credential** : Celui de l'exercice 1 est configuré ;)
- **Tool Description** : `Set Automatically`. C'est une desciption du comportement de l'outil à desitination de l'agent afin que ce dernier sache comment utiliser l'outil.
- **Ressource** : `Issue`. C'est la gestion des tickets  
- **Operation** : `Create`. On veut créer des User Stories.
- **Project** : `AeS 2025`. Notre projet Jira pour l'atelier.
- **Issue type** : `Story`. Le type de ticket Jira que l'on veut créer.
- **Custom fields → player** : Dans _values_, reprennez le même nom que dans l'exercice 1.

Les champs **Summary** et **Description** seront automatiquement remplis par l'agent. 

## 6) Exécution du workflow !

Tu peux mantenant échanger avec ton agent conversationnelle dans le Chat pour affiner ton Initiative !
Ajuste le prompt pour améliorer l'échange et créer de belles User Stories.

👏​ Accède à JIRA pour voir tes [Users Stories](https://pcarpentiermail.atlassian.net/issues/?filter=10034)

​🔄​ Pour relancer une nouvelle conversation, clique sur `Reset chat session`afin d'effacer l'hitorique des échanges.
