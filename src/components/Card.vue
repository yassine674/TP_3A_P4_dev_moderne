<template>
  <!-- Clic sur la carte pour ouvrir la modale d'édition -->
  <div class="carte" :class="classeSelonDifficulte" @click="$emit('edit-card', card)">

    <!-- Titre et badge difficulté -->
    <div class="carte-entete">
      <h3>{{ card.title }}</h3>
      <span class="badge" :class="classeBadge">{{ card.difficulty }}</span>
    </div>

    <!-- Description -->
    <p class="description">{{ card.description }}</p>

    <!-- Récompense : visible uniquement si la quête est terminée -->
    <p v-if="card.status === 'terminee' && card.reward" class="recompense">
      Récompense : {{ card.reward }}
    </p>

    <!-- Date limite (en rouge si dépassée) -->
    <p v-if="card.dueDate" class="date" :class="{ 'date-retard': estEnRetard }">
      Date limite : {{ formatDate(card.dueDate) }}
      <span v-if="estEnRetard">— En retard !</span>
    </p>

    <!-- Boutons d'action -->
    <div class="actions" @click.stop>

      <div class="boutons-deplacement">
        <!-- Bouton reculer : masqué si première colonne -->
        <button
          v-if="!isFirst"
          class="btn-deplacer"
          @click="$emit('move-card', { cardId: card.id, direction: 'backward' })"
        >
          &larr; Reculer
        </button>

        <!-- Bouton avancer : masqué si dernière colonne -->
        <button
          v-if="!isLast"
          class="btn-deplacer"
          @click="$emit('move-card', { cardId: card.id, direction: 'forward' })"
        >
          Avancer &rarr;
        </button>
      </div>

      <!-- Bouton supprimer : masqué si quête terminée -->
      <button
        v-if="card.status !== 'terminee'"
        class="btn-supprimer"
        @click="$emit('delete-card', card.id)"
      >
        Supprimer
      </button>

    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  card: {
    type: Object,
    required: true,
  },
  isFirst: {
    type: Boolean,
    default: false,
  },
  isLast: {
    type: Boolean,
    default: false,
  },
})

defineEmits(['edit-card', 'delete-card', 'move-card'])

// Classe CSS pour la couleur de la bordure gauche selon la difficulté
const classeSelonDifficulte = computed(() => {
  if (props.card.difficulty === 'facile')     return 'carte-facile'
  if (props.card.difficulty === 'moyen')      return 'carte-moyen'
  if (props.card.difficulty === 'difficile')  return 'carte-difficile'
  if (props.card.difficulty === 'légendaire') return 'carte-legendaire'
  return ''
})

// Classe CSS pour la couleur du badge de difficulté
const classeBadge = computed(() => {
  if (props.card.difficulty === 'facile')     return 'badge-facile'
  if (props.card.difficulty === 'moyen')      return 'badge-moyen'
  if (props.card.difficulty === 'difficile')  return 'badge-difficile'
  if (props.card.difficulty === 'légendaire') return 'badge-legendaire'
  return ''
})

// Vrai si la date limite est dépassée et que la quête n'est pas encore terminée
const estEnRetard = computed(() => {
  if (!props.card.dueDate || props.card.status === 'terminee') return false
  return new Date(props.card.dueDate) < new Date()
})

function formatDate(dateStr) {
  return new Date(dateStr).toLocaleDateString('fr-FR')
}
</script>

<style scoped>
.carte {
  background-color: white;
  border: 1px solid #ddd;
  border-left: 4px solid #ccc;
  border-radius: 6px;
  padding: 12px;
  cursor: pointer;
}

.carte:hover {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

/* Couleur de la bordure gauche selon la difficulté */
.carte-facile     { border-left-color: #27ae60; }
.carte-moyen      { border-left-color: #f39c12; }
.carte-difficile  { border-left-color: #e74c3c; }
.carte-legendaire { border-left-color: #9b59b6; }

.carte-entete {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 8px;
  margin-bottom: 8px;
}

.carte-entete h3 {
  font-size: 14px;
  font-weight: bold;
  color: #222;
}

.badge {
  font-size: 11px;
  padding: 2px 8px;
  border-radius: 10px;
  font-weight: bold;
  white-space: nowrap;
}

.badge-facile     { background-color: #d4edda; color: #155724; }
.badge-moyen      { background-color: #fff3cd; color: #856404; }
.badge-difficile  { background-color: #f8d7da; color: #721c24; }
.badge-legendaire { background-color: #e2d9f3; color: #4a235a; }

.description {
  font-size: 12px;
  color: #666;
  margin-bottom: 8px;
  line-height: 1.5;
}

.recompense {
  font-size: 12px;
  color: #27ae60;
  margin-bottom: 6px;
  font-style: italic;
}

.date {
  font-size: 12px;
  color: #888;
  margin-bottom: 8px;
}

.date-retard {
  color: #e74c3c;
  font-weight: bold;
}

.actions {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 8px;
  border-top: 1px solid #eee;
}

.boutons-deplacement {
  display: flex;
  gap: 4px;
}

.btn-deplacer {
  font-size: 12px;
  padding: 4px 8px;
  background-color: #ecf0f1;
  border: 1px solid #bdc3c7;
  border-radius: 4px;
  color: #555;
}

.btn-deplacer:hover {
  background-color: #bdc3c7;
}

.btn-supprimer {
  font-size: 12px;
  padding: 4px 8px;
  background: none;
  border: none;
  color: #e74c3c;
}

.btn-supprimer:hover {
  text-decoration: underline;
}
</style>
