<template>
  <div>
    <header class="header">
      <h1>Gestionnaire de Quêtes RPG</h1>
    </header>
    <main class="contenu">
      <Board />
    </main>
  </div>
</template>

<script setup>
import Board from './components/Board.vue'
</script>

<style scoped>
.header {
  background-color: #2c3e50;
  color: white;
  padding: 16px 24px;
}

.header h1 {
  font-size: 22px;
}

.contenu {
  max-width: 1200px;
  margin: 0 auto;
  padding: 24px;
}
</style>
