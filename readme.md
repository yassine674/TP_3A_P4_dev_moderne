# TP Vue.js — Gestionnaire de Quêtes RPG

**Étudiant :** Yassine  
**Module :** Développement Web Avancé  
**Framework :** Vue 3 — Composition API

---

## Présentation du projet

J'ai choisi de réaliser un **gestionnaire de quêtes inspiré des jeux de rôle (RPG)**.

L'application permet de gérer des quêtes organisées en 4 colonnes selon leur avancement :

**Disponible → En cours → Terminée → Échouée**

Chaque quête contient un titre, une description, un niveau de difficulté, une récompense et une date limite. On peut ajouter, modifier, supprimer et déplacer les quêtes entre les colonnes.

---

## Lancer le projet

```bash
npm install
npm run dev
```

L'application s'ouvre à `http://localhost:5173`.

---

## Structure des fichiers

```
src/
├── main.js               # Monte l'application Vue sur #app
├── style.css             # Reset CSS global
├── App.vue               # Composant racine : header + Board
└── components/
    ├── Board.vue         # Hub central : gère tout l'état et la logique
    ├── Column.vue        # Affiche une colonne et ses cartes
    ├── Card.vue          # Affiche une quête individuelle
    ├── CardForm.vue      # Formulaire d'ajout et de modification
    └── Modal.vue         # Fenêtre modale générique (slot)
```

---

## Fonctionnalités Vue utilisées

### Composants et props

Chaque composant reçoit ses données via des **props** passées par le parent.

Exemple : `Board.vue` passe à `Column.vue` la liste des quêtes filtrées et deux props `isFirst` / `isLast` pour contrôler l'affichage des boutons de déplacement.

### Emits

Les événements remontent du bas vers le haut :

- `Card.vue` émet `edit-card`, `delete-card`, `move-card`
- `Column.vue` reçoit ces événements et les re-transmet à `Board.vue`
- `Board.vue` exécute la logique correspondante

### v-for

Les cartes sont générées dynamiquement avec `v-for` et `:key="card.id"` (jamais sur l'index).

### v-if / v-else

- Le bouton **Supprimer** est masqué si la quête est terminée
- Le badge **Récompense** est visible seulement si le statut est "Terminée"
- Les boutons **Reculer** et **Avancer** sont masqués sur la première et dernière colonne

### v-model

Tous les champs du formulaire (`CardForm.vue`) sont liés avec `v-model` à un objet réactif `form`.

### Computed

Trois propriétés calculées dans `Card.vue` :
- `classeSelonDifficulte` : renvoie la classe CSS de la couleur de la carte
- `classeBadge` : renvoie la classe CSS du badge de difficulté
- `estEnRetard` : renvoie `true` si la date limite est dépassée

Deux dans `Board.vue` :
- `nbTerminees` : compte les quêtes terminées
- `pourcentage` : calcule le pourcentage de complétion

### Watch

Dans `Board.vue`, un `watch` avec `{ deep: true }` surveille le tableau des quêtes et sauvegarde automatiquement dans le `localStorage` à chaque modification.

Dans `CardForm.vue`, un `watch` sur la prop `cardData` remplit le formulaire avec les données de la quête quand on ouvre la modale en mode édition (ou le vide en mode ajout).

### Slot

`Modal.vue` est un composant générique qui expose un `<slot />`. Cela permet d'injecter n'importe quel contenu à l'intérieur. Ici, on y place `CardForm.vue`.

### localStorage

Les quêtes sont sauvegardées dans le navigateur avec `JSON.stringify` et rechargées au démarrage avec `JSON.parse`. Les données persistent même après fermeture du navigateur.

---

## Flux de données

```
App.vue
└── Board.vue  (état global : quêtes, filtres, modale)
    ├── Column.vue  <-- props : column, cards, isFirst, isLast
    │   └── Card.vue  <-- props : card, isFirst, isLast
    │       emits -->  edit-card | delete-card | move-card
    │   emits -->  add-card | edit-card | delete-card | move-card
    └── Modal.vue  <-- props : isOpen, title
        └── [slot] CardForm.vue  <-- props : cardData, defaultStatus
                   emits -->  submit-card | cancel
```

---

## Fonctionnalités bonus

- Recherche en temps réel sur le titre et la description
- Filtre par niveau de difficulté
- Affichage du pourcentage de quêtes terminées
- Design responsive (mobile, tablette, desktop)

---

## Choix techniques

| Choix | Raison |
|---|---|
| Vue 3 Composition API | Standard actuel, `<script setup>` plus lisible |
| CSS classique (pas de framework) | Simplicité, contrôle total du style |
| localStorage | Persistance sans backend nécessaire |
| `Date.now()` pour les IDs | Génère un identifiant unique simple |
| `Board.vue` comme hub | Centralise l'état pour un flux de données clair |
