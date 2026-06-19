<template>
  <div class="colonne">

    <!-- En-tête avec le titre et le nombre de cartes -->
    <div class="colonne-entete" :class="classeEntete">
      <h2>{{ column.title }}</h2>
      <span class="compteur">{{ cards.length }}</span>
    </div>

    <!-- Liste des cartes -->
    <div class="colonne-corps">
      <p v-if="cards.length === 0" class="vide">Aucune quête ici</p>

      <!-- v-for avec :key sur l'id unique de chaque carte -->
      <Card
        v-for="card in cards"
        :key="card.id"
        :card="card"
        :is-first="isFirst"
        :is-last="isLast"
        @edit-card="$emit('edit-card', $event)"
        @delete-card="$emit('delete-card', $event)"
        @move-card="$emit('move-card', $event)"
      />
    </div>

    <!-- Bouton pour ajouter une quête dans cette colonne -->
    <div class="colonne-pied">
      <button class="btn-ajouter-ici" @click="$emit('add-card', column.id)">
        + Ajouter une quête
      </button>
    </div>

  </div>
</template>

<script setup>
import { computed } from 'vue'
import Card from './Card.vue'

const props = defineProps({
  column: {
    type: Object,
    required: true,
  },
  cards: {
    type: Array,
    default: () => [],
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

defineEmits(['add-card', 'edit-card', 'delete-card', 'move-card'])

// Couleur de l'en-tête selon la colonne
const classeEntete = computed(() => {
  if (props.column.id === 'disponible') return 'entete-bleu'
  if (props.column.id === 'en-cours')   return 'entete-orange'
  if (props.column.id === 'terminee')   return 'entete-vert'
  if (props.column.id === 'echouee')    return 'entete-rouge'
  return ''
})
</script>

<style scoped>
.colonne {
  background-color: #ebebeb;
  border-radius: 6px;
  display: flex;
  flex-direction: column;
  min-height: 300px;
}

.colonne-entete {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 14px;
  border-radius: 6px 6px 0 0;
  background-color: #d0d0d0;
}

/* Couleurs des en-têtes de colonnes */
.entete-bleu   { background-color: #2980b9; color: white; }
.entete-orange { background-color: #e67e22; color: white; }
.entete-vert   { background-color: #27ae60; color: white; }
.entete-rouge  { background-color: #c0392b; color: white; }

.colonne-entete h2 {
  font-size: 14px;
  font-weight: bold;
}

.compteur {
  background-color: rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  width: 22px;
  height: 22px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  font-weight: bold;
}

.colonne-corps {
  flex: 1;
  padding: 10px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.vide {
  text-align: center;
  color: #999;
  font-size: 13px;
  padding: 20px 0;
}

.colonne-pied {
  padding: 8px 10px;
  border-top: 1px solid #ccc;
}

.btn-ajouter-ici {
  width: 100%;
  padding: 6px;
  background: none;
  border: 1px dashed #aaa;
  border-radius: 4px;
  color: #666;
  font-size: 13px;
}

.btn-ajouter-ici:hover {
  background-color: #ddd;
  color: #333;
}
</style>
