# Exercice 1 - Flow de création d'US

> **Objectif** : Propose une nouvelle *initiative* pour ton **Produit** dans un chat n8n etgénérer automatiquement des **User Stories dans JIRA** 
> **Durée** : 30–45 min 

---

## 1) Pré-requis & vue d’ensemble

### Pré-requis
- **n8n Cloud** ou n8n local (Docker) accessible.

### Architecture du flow

```
Chat Trigger → Set(ProductDescription) → LLM Chain(Write User Stories)
   ↘                    ↗           			     ↘
   (input)        (context)    				  Output Parser + LLM model
                               					          ↓
                             	   		      Split Out (user_stories[])
                                      	    			  ↓
                                     		 Jira – Create Issue (Story)
```

---

## 2) Node — **Chat Trigger**

### 🎯 Rôle

Point d’entrée : Tu décris l’initiative dans un chat embarqué.

### 🔎 Où le trouver / ajouter

- Clique sur `+` au milieu de ton canva workflow
- Tape **Chat Trigger** dans la barre de recherche.

### ⚙️ Configuration

- Renomme le nœud en \`chat\`;

### 🧪 Test rapide

- Dans le cadre de saisie de texte (en bas à gauche), vérifie le statut  **Node Executed Successfully** du trigger.
- Pour information, la donnée sera disponible via `$('chat').item.json.chatInput`.

### 🧱 Code du nœud

[01_chat.json](./nodes/01_chat.json)

---

## 3) Node — **Set (Product Description)**

## 🎯 Rôle
Fournir au LLM un **contexte produit** clair et stable, distinct de l’initiative.

### Pré-requis 
Rédigez une description strucuturé **de l'objectif de ton produit, des utilisateurs et des fonctionnalités principales**

### 🔎 Où le trouver
- Clique sur `+` à la fin de ton workflow
- Selectionne : **Data transformation → Edit Field (Set)**.
  
### ⚙️ Configuration
- Onglet **Paramèters** → cliquer sur **Add a field** :
  - **Name** : `ProductDescription`
  - **Type** : `string`
  - **Value** : La desciption du produit rédigée pendant les pré-requis
- Cliquer sur **Execute Step**
- Renomme le nœud : \`**Product Description**\`

### 🧱 Code du nœud

???

---

## 4) Node — **Basic LLM Chain**

### 🎯 Rôle
Paramètrer **l'Agent** permettant de rédiger les User Stories.

### 🔎 Où le trouver
- Clique sur `+` à la fin de ton workflow
- Selectionne : **AI → Basic LLM Chain**

### ⚙️ Configuration
- Renommez le noeud : `Write User Stories`.
- Si les noeuds ne sont pas connectés entre eux automatiquement, alors dans le champ (**Source for Prompt (User Message**) choisissez `Define below`. Cela permet d'envoyer l'initiative au LLM
- **Prompt (User Message)** :
```markdown
Initiative à découper :
{{ $('Chat').item.json.chatInput }}
```
- **Require Specific Output Format** : `activté`. On souhaite que le LLM nous donne les US dans un format strucuturé (json)
- Cliquer sur **Add  prompt**
     - **Type Name or ID** : `system`
     - **Message** : _system_prompt_ défini ci-dessous au préalable
- Ne pas cliquer sur **Execute Step**. Tant que les sections 5) et 6) ne sont pas réalisées, le workflow sera toujours en statut **failed**

### 🖋️ Configurer l'agent Product Ower

#### À quoi sert le _system prompt_
- C’est **le brief de base** donné au modèle : il fixe **le rôle**, **les règles** et **les limites** (ex. “tu es un Product Owner, parle en français, réponds en JSON”).
- Il agit comme des **garde‑fous** : quand plusieurs instructions se contredisent, le _system prompt_ est **prioritaire** sur le reste des messages. De plus il reste toujours en mémoire.
- Il **stabilise** les réponses (ton, format, niveau de détail) et **réduit l’imprévu**.
- Il sert à **spécifier le format de sortie** (ex. champs JSON obligatoires) et les **interdictions** (“pas de Markdown, pas de spé techniques”).
- On y met ce qui doit rester **constant** (rôle, cadre, critères de qualité) ; on met dans les messages suivants ce qui est **variable** (le brief utilisateur).

#### Template de system prompt
Utilisez un prompt en 5 parties claires :
1. **Role** — qui est l'agent ? (ex. *Product Owner expérimenté*)
2. **Mission** — transformer une initiative en User Stories (avec contraintes INVEST ?)
3. **Instruction** — gabarit de Story (Title + Description en « As a / I want / So that »). Limite à **5 User Stories max**.
4. **Product Description** — injectez `{{ $json.ProductDescription }}` pour cadrer le périmètre
5. **Tone & Style** — concision, valeur business, actionnable


#### Exemple de System Prompt Product Owner
```markdown
# Rôle
Tu es un **Product Owner Scrum expérimenté**. Tu parles **français**. Tu aides à clarifier une initiative et à la traduire en **User Stories** utiles pour une équipe Agile.

# Mission
À partir d’une initiative fournie par l’utilisateur, produis **au plus 5** User Stories **conformes INVEST** et adaptées au produit existant. Si l’information est insuffisante, pose des questions ciblées et concises avant de générer les stories.

# Instructions
- **Gabarit de User Story**
  - **title** : résumé clair, orienté valeur utilisateur.
  - **description** : En tant que [rôle], je veux [capacité], afin de [bénéfice].
  - **Format de sortie (JSON uniquement)
  - Pas de texte hors JSON. Pas de Markdown. Pas de spécifications techniques. Pas de tâches/sous-tâches.

# Description du produit
{{ $json.ProductDescription }}

# Ton & style
Clair, concis, **orienté valeur métier**, immédiatement **actionnable** par une équipe. Évite le jargon. Garantis la cohérence et la testabilité de chaque story.

```

---

## 5) Ajouter le LLM à l'agent
### 🎯 Rôle

Définir le Large Language Model utilisé par l'agent.

### 🔎 Où l'ajouter
- Clique sur `+` Model au niveau de l'agent.
- Selectionne : **OpenAI Chat Model**

### ⚙️ Configuration

1. **Create new Credential** : Se connecter au compte OpenAI pour utiliser l'API de chatGPT.
2. **API Key** : Utiliser la clef d'API suivante
```text
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
```

4. **Save & Test** : Sauvegarder, la connexion avec Open AI est validée.
5. **Model** : gpt-4o . Offre un bon compris de vitesse et de qualité. Pour choisir votre modèle openAI [modèle openAI](https://platform.openai.com/docs/models/compare)
6. **Sampling Temperature** : `0.2` C'est le niveau de predictibilité / créativité du modèle compris entre 0 et 2.
7. Renomme le nœud : \`**Ecrire les US**\`

### ✅ Bonnes pratiques
- Tu peux essayer plus tard de changer le model pour comparer les résultats.

---

## 6) Ajouter le **Structured Output Parser**

### 🎯 Rôle
Imposer un **schéma** que le LLM doit respecter (liste de User Stories avec `title` et `description`) pour récupérer une strucuture défini.

### 🔎 Où l'ajouter
- Clique sur `+` Output Parser au niveau de l'agent.
- Selectionne : **Structured Output Parser**

### ⚙️ Configuration
- **Schema type** : `Define using JSON Schema`
- **Input schema** : collez le JSON Schema fourni (ci-dessous).

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "User Story Document",
  "type": "object",
  "properties": {    
    "user_stories": {
      "type": "array",
      "description": "List of User Stories derived from the Epic.",
      "items": {
        "type": "object",
        "properties": {
          "title": {
            "type": "string",
            "description": "A concise and user-oriented name for the User Story."
          },
          "description": {
            "type": "string",
            "description": "Standard User Story format: 'As a [user role], I want [feature or action], so that [expected benefit]'."
          }
        },
        "required": ["title", "description"]
      }
    }
  },
  "required": ["user_stories"]
}
```

### ✅ Bonnes pratiques
- Tu peux aussi générer automatiquement le schema à partir d'un exemple en json.

---

### 🧱 Code du nœud complet

???

---

## 7) Node — **Split Out**

### 🎯 Rôle
Parcourir **chaque élément** du tableau `output.user_stories` produit par la LLM Chain. La suite du workflow sera appelée autant de fois qu'il y a d'éléments.

### 🔎 Où le trouver
- Clique sur `+` à la fin de ton workflow
- Selectionne : **Split Out**

### ⚙️ Configuration
- **Field to split out** : `output.user_stories` : Chaque exécution suivante recevra un item USer Story `{ title, description }`.

### 🧱 Code du nœud
???

---

## 8) Node — **Jira – Create Issue**

### 🎯 Rôle
Créer une **issue Story** dans Jira pour chaque item en sortie du `Split Out`.

### 🔎 Où le trouver / ajouter
- Clique sur `+` à la fin de ton workflow
- Recherche : **Jira Software → Create an issue**

### ⚙️ Configuration
- **Credential** : Créer la connexion avec l'espace JIRA
    - Email : `pierre.carpentier@thiga.co`
    - API Token :
```text
ATATT3xFfGF0zK5Hlhc4pX35b6HJNGg7PsZ1VDCC37GCP9T1wMScddwgyWSp-QHPYMQWMiF0L3_Hy1Yq6uX54DX_DtA_aT56yHPRRwrrTfKgr7oHk4f3iLq6_rLXbwD6O42sxTVMxetQJsFPqtuurpm-OrQqCi_ZPdsksq3dCMNxsyTj17LTt9U=823EF8CC
```
    - Domain : `https://pcarpentiermail.atlassian.net/` 
- **Ressource** : Issue
- **Project** : `AeS 2025`
- **Issue type** : `Story`
- **Summary** : `{{ $json.title }}`
- **Add fields → Description** : `{{ $json.description }}`
- **Custom fields → player** : Dans _values_, définir un nom unique pour identifier vos US ;)

### 🧱 Code du nœud
???

---


## 9) Exécution du work flow !

### 🧪 Test Du workflow
Tu peux envoyer dans le Chat ton initiative pour tester ton workflow n8n et créer tes User Stories !

👏​ Accède à JIRA pour voir tes [Users Stories](https://pcarpentiermail.atlassian.net/issues/?filter=10034)

➿​ Tu peux maintenant ajuster ton prompt pour améliorer tes User Stories ! 
