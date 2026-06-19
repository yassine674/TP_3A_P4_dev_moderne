<template>
  <form @submit.prevent="handleSubmit">

    <!-- Titre -->
    <div class="champ">
      <label>Titre <span class="obligatoire">*</span></label>
      <input
        v-model="form.title"
        type="text"
        placeholder="Ex : Vaincre le Dragon des Cimes"
        :class="erreurTitre ? 'input-erreur' : ''"
      />
      <p v-if="erreurTitre" class="message-erreur">{{ erreurTitre }}</p>
    </div>

    <!-- Description -->
    <div class="champ">
      <label>Description <span class="obligatoire">*</span></label>
      <textarea
        v-model="form.description"
        placeholder="Décrivez la quête..."
        rows="3"
        :class="erreurDescription ? 'input-erreur' : ''"
      ></textarea>
      <p v-if="erreurDescription" class="message-erreur">{{ erreurDescription }}</p>
    </div>

    <!-- Difficulté -->
    <div class="champ">
      <label>Difficulté</label>
      <div class="radio-groupe">
        <label v-for="niveau in niveaux" :key="niveau.valeur" class="radio-option">
          <input type="radio" v-model="form.difficulty" :value="niveau.valeur" />
          {{ niveau.label }}
        </label>
      </div>
    </div>

    <!-- Statut -->
    <div class="champ">
      <label>Statut</label>
      <select v-model="form.status">
        <option value="disponible">Disponible</option>
        <option value="en-cours">En cours</option>
        <option value="terminee">Terminée</option>
        <option value="echouee">Échouée</option>
      </select>
    </div>

    <!-- Récompense -->
    <div class="champ">
      <label>Récompense</label>
      <input
        v-model="form.reward"
        type="text"
        placeholder="Ex : 100 XP, Épée enchantée..."
      />
    </div>

    <!-- Date limite -->
    <div class="champ">
      <label>Date limite</label>
      <input v-model="form.dueDate" type="date" />
    </div>

    <!-- Boutons -->
    <div class="boutons-formulaire">
      <button type="submit" class="btn-valider">
        {{ modeEdition ? 'Modifier' : 'Créer la quête' }}
      </button>
      <button type="button" class="btn-annuler" @click="$emit('cancel')">
        Annuler
      </button>
    </div>

  </form>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  // null si ajout, objet quête si modification
  cardData: {
    type: Object,
    default: null,
  },
  defaultStatus: {
    type: String,
    default: 'disponible',
  },
})

const emit = defineEmits(['submit-card', 'cancel'])

const niveaux = [
  { valeur: 'facile',     label: 'Facile'     },
  { valeur: 'moyen',      label: 'Moyen'      },
  { valeur: 'difficile',  label: 'Difficile'  },
  { valeur: 'légendaire', label: 'Légendaire' },
]

// Données du formulaire
const form = ref({
  title: '',
  description: '',
  difficulty: 'moyen',
  status: props.defaultStatus,
  reward: '',
  dueDate: '',
})

const erreurTitre = ref('')
const erreurDescription = ref('')

// true si on modifie une quête existante
const modeEdition = computed(() => props.cardData !== null)

// Remplir le formulaire quand on ouvre la modale en mode édition
watch(() => props.cardData, (quete) => {
  erreurTitre.value = ''
  erreurDescription.value = ''
  if (quete) {
    form.value = { ...quete }
  } else {
    form.value = {
      title: '',
      description: '',
      difficulty: 'moyen',
      status: props.defaultStatus,
      reward: '',
      dueDate: '',
    }
  }
}, { immediate: true })

function handleSubmit() {
  // Vérification des champs obligatoires
  erreurTitre.value = ''
  erreurDescription.value = ''

  if (!form.value.title.trim()) {
    erreurTitre.value = 'Le titre est obligatoire.'
    return
  }
  if (!form.value.description.trim()) {
    erreurDescription.value = 'La description est obligatoire.'
    return
  }

  emit('submit-card', {
    id: props.cardData ? props.cardData.id : null,
    title: form.value.title.trim(),
    description: form.value.description.trim(),
    difficulty: form.value.difficulty,
    status: form.value.status,
    reward: form.value.reward,
    dueDate: form.value.dueDate,
  })
}
</script>

<style scoped>
.champ {
  margin-bottom: 16px;
}

.champ label {
  display: block;
  font-size: 14px;
  font-weight: bold;
  margin-bottom: 6px;
  color: #444;
}

.obligatoire {
  color: #e74c3c;
}

.champ input[type="text"],
.champ input[type="date"],
.champ textarea,
.champ select {
  width: 100%;
  padding: 8px 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 14px;
}

.champ input:focus,
.champ textarea:focus,
.champ select:focus {
  outline: none;
  border-color: #3498db;
}

.input-erreur {
  border-color: #e74c3c !important;
}

.message-erreur {
  color: #e74c3c;
  font-size: 12px;
  margin-top: 4px;
}

.radio-groupe {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
}

.radio-option {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 14px;
  cursor: pointer;
}

.champ textarea {
  resize: vertical;
}

.boutons-formulaire {
  display: flex;
  gap: 10px;
  margin-top: 8px;
}

.btn-valider {
  flex: 1;
  padding: 10px;
  background-color: #3498db;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 14px;
  font-weight: bold;
}

.btn-valider:hover {
  background-color: #2980b9;
}

.btn-annuler {
  padding: 10px 16px;
  background-color: #ecf0f1;
  color: #555;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 14px;
}

.btn-annuler:hover {
  background-color: #ddd;
}
</style>
