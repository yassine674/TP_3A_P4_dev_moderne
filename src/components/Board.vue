<template>
  <div>

    <!-- Barre de recherche et filtres -->
    <div class="filtres">
      <input
        v-model="recherche"
        type="text"
        placeholder="Rechercher une quête..."
        class="input-recherche"
      />
      <select v-model="filtreDifficulte">
        <option value="">Toutes les difficultés</option>
        <option value="facile">Facile</option>
        <option value="moyen">Moyen</option>
        <option value="difficile">Difficile</option>
        <option value="légendaire">Légendaire</option>
      </select>
      <button class="btn-nouveau" @click="ouvrirModalAjout('disponible')">
        + Nouvelle quête
      </button>
    </div>

    <!-- Statistiques -->
    <p class="stats">
      {{ nbTerminees }} / {{ quetes.length }} quêtes terminées ({{ pourcentage }}%)
    </p>

    <!-- Tableau des 4 colonnes -->
    <div class="tableau">
      <Column
        v-for="(col, index) in colonnes"
        :key="col.id"
        :column="col"
        :cards="quetesFiltrees(col.id)"
        :is-first="index === 0"
        :is-last="index === colonnes.length - 1"
        @add-card="ouvrirModalAjout"
        @edit-card="ouvrirModalEdition"
        @delete-card="supprimerQuete"
        @move-card="deplacerQuete"
      />
    </div>

    <!-- Modale avec le formulaire injecté via slot -->
    <Modal
      :is-open="afficherModal"
      :title="queteEnCours ? 'Modifier la quête' : 'Nouvelle quête'"
      @close="fermerModal"
    >
      <CardForm
        :card-data="queteEnCours"
        :default-status="colonneDefaut"
        @submit-card="validerFormulaire"
        @cancel="fermerModal"
      />
    </Modal>

  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import Column from './Column.vue'
import Modal from './Modal.vue'
import CardForm from './CardForm.vue'

// Les 4 colonnes du tableau kanban
const colonnes = [
  { id: 'disponible', title: 'Disponible' },
  { id: 'en-cours',   title: 'En cours'   },
  { id: 'terminee',   title: 'Terminée'   },
  { id: 'echouee',    title: 'Échouée'    },
]

// Données de démonstration (utilisées si localStorage est vide)
const questesDefaut = [
  {
    id: 1,
    title: 'Vaincre le Dragon des Cimes',
    description: 'Un dragon terrorise les villages du nord. Mettez fin à sa tyrannie.',
    difficulty: 'légendaire',
    status: 'disponible',
    reward: '500 XP, Épée de Feu',
    dueDate: '2026-07-01',
  },
  {
    id: 2,
    title: 'Collecter des herbes de lune',
    description: "L'alchimiste a besoin de 10 herbes de lune pour sa potion.",
    difficulty: 'facile',
    status: 'disponible',
    reward: '50 XP, Potion de soin',
    dueDate: '2026-06-10',
  },
  {
    id: 3,
    title: 'Escorter le marchand Aldric',
    description: "Un marchand doit traverser la forêt des Ombres. Assurez sa protection.",
    difficulty: 'moyen',
    status: 'en-cours',
    reward: "150 XP, 50 pièces d'or",
    dueDate: '2026-06-20',
  },
  {
    id: 4,
    title: 'Libérer la taverne du Chêne Noir',
    description: 'Des brigands ont pris en otage la taverne. Délogez-les.',
    difficulty: 'difficile',
    status: 'terminee',
    reward: '250 XP, Bouclier enchanté',
    dueDate: '',
  },
  {
    id: 5,
    title: 'Retrouver le grimoire maudit',
    description: 'Un grimoire volé menace de répandre une malédiction dans Velundra.',
    difficulty: 'difficile',
    status: 'echouee',
    reward: '300 XP, Anneau de protection',
    dueDate: '2026-05-28',
  },
]

// Chargement depuis localStorage, ou données par défaut
const donneesStockees = localStorage.getItem('rpg-quetes')
const quetes = ref(donneesStockees ? JSON.parse(donneesStockees) : [...questesDefaut])

// Filtres de recherche
const recherche = ref('')
const filtreDifficulte = ref('')

// État de la modale
const afficherModal = ref(false)
const queteEnCours = ref(null)
const colonneDefaut = ref('disponible')

// Sauvegarde automatique dans localStorage à chaque modification
watch(quetes, (nouvellesQuetes) => {
  localStorage.setItem('rpg-quetes', JSON.stringify(nouvellesQuetes))
}, { deep: true })

// Nombre de quêtes terminées
const nbTerminees = computed(() => {
  return quetes.value.filter(q => q.status === 'terminee').length
})

// Pourcentage de completion
const pourcentage = computed(() => {
  if (quetes.value.length === 0) return 0
  return Math.round((nbTerminees.value / quetes.value.length) * 100)
})

// Retourne les quêtes filtrées pour une colonne donnée
function quetesFiltrees(colonneId) {
  return quetes.value.filter(quete => {
    if (quete.status !== colonneId) return false
    if (filtreDifficulte.value && quete.difficulty !== filtreDifficulte.value) return false
    if (recherche.value) {
      const terme = recherche.value.toLowerCase()
      return quete.title.toLowerCase().includes(terme) ||
             quete.description.toLowerCase().includes(terme)
    }
    return true
  })
}

// Ajouter une nouvelle quête
function ajouterQuete(donnees) {
  quetes.value.push({
    ...donnees,
    id: Date.now(),
  })
}

// Modifier une quête existante
function modifierQuete(donnees) {
  const index = quetes.value.findIndex(q => q.id === donnees.id)
  if (index !== -1) {
    quetes.value[index] = { ...donnees }
  }
}

// Supprimer une quête par son id
function supprimerQuete(queteId) {
  quetes.value = quetes.value.filter(q => q.id !== queteId)
}

// Déplacer une quête vers la colonne suivante ou précédente
function deplacerQuete({ cardId, direction }) {
  const ordre = ['disponible', 'en-cours', 'terminee', 'echouee']
  const quete = quetes.value.find(q => q.id === cardId)
  if (!quete) return

  const index = ordre.indexOf(quete.status)
  if (direction === 'forward' && index < ordre.length - 1) {
    quete.status = ordre[index + 1]
  }
  if (direction === 'backward' && index > 0) {
    quete.status = ordre[index - 1]
  }
}

function ouvrirModalAjout(colonneId) {
  queteEnCours.value = null
  colonneDefaut.value = colonneId || 'disponible'
  afficherModal.value = true
}

function ouvrirModalEdition(quete) {
  queteEnCours.value = { ...quete }
  afficherModal.value = true
}

function fermerModal() {
  afficherModal.value = false
  queteEnCours.value = null
}

// Appelé quand le formulaire est soumis
function validerFormulaire(donnees) {
  if (queteEnCours.value) {
    modifierQuete(donnees)
  } else {
    ajouterQuete(donnees)
  }
  fermerModal()
}
</script>

<style scoped>
.filtres {
  display: flex;
  gap: 10px;
  margin-bottom: 16px;
  flex-wrap: wrap;
  align-items: center;
}

.input-recherche {
  flex: 1;
  min-width: 180px;
  padding: 8px 12px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 14px;
}

.filtres select {
  padding: 8px 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 14px;
  background-color: white;
}

.btn-nouveau {
  padding: 8px 16px;
  background-color: #3498db;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 14px;
  font-weight: bold;
}

.btn-nouveau:hover {
  background-color: #2980b9;
}

.stats {
  font-size: 13px;
  color: #666;
  margin-bottom: 20px;
}

.tableau {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
}

@media (max-width: 900px) {
  .tableau {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 550px) {
  .tableau {
    grid-template-columns: 1fr;
  }
}
</style>
