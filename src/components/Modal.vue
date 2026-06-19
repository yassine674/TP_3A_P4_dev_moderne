<template>
  <!-- Overlay visible uniquement si isOpen est true -->
  <div v-if="isOpen" class="modal-fond" @click.self="$emit('close')">
    <div class="modal-boite">

      <!-- En-tête avec le titre et le bouton fermer -->
      <div class="modal-entete">
        <h2>{{ title }}</h2>
        <button class="btn-fermer" @click="$emit('close')">×</button>
      </div>

      <!-- Le contenu est injecté ici via le slot -->
      <div class="modal-corps">
        <slot />
      </div>

    </div>
  </div>
</template>

<script setup>
// Modal générique : reçoit isOpen et title, et expose un slot pour le contenu
defineProps({
  isOpen: {
    type: Boolean,
    required: true,
  },
  title: {
    type: String,
    default: 'Fenêtre',
  },
})

defineEmits(['close'])
</script>

<style scoped>
.modal-fond {
  position: fixed;
  inset: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
  padding: 16px;
}

.modal-boite {
  background-color: white;
  border-radius: 8px;
  width: 100%;
  max-width: 500px;
  max-height: 90vh;
  display: flex;
  flex-direction: column;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
}

.modal-entete {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 20px;
  border-bottom: 1px solid #ddd;
}

.modal-entete h2 {
  font-size: 17px;
  color: #333;
}

.btn-fermer {
  background: none;
  border: none;
  font-size: 22px;
  color: #888;
  line-height: 1;
}

.btn-fermer:hover {
  color: #333;
}

.modal-corps {
  padding: 20px;
  overflow-y: auto;
}
</style>
