
<script setup>
import { ref } from 'vue'
import { devises, taux } from '../data/taux.js'

// Données du convertisseur
const montant = ref('0')
const deviseSource = ref('USD')
const deviseCible = ref('EUR')
const resultat = ref('')
const erreur = ref(false)

// Touches du clavier numérique
const touches = [
  '7', '8', '9',
  '4', '5', '6',
  '1', '2', '3',
  'C', '0', '.'
]

// Gérer les clics sur le clavier numérique
function appuyer(touche) {
  // Effacer le montant et le résultat
  if (touche === 'C') {
    montant.value = '0'
    resultat.value = ''
    erreur.value = false
    return
  }

  // Ignorer un deuxième point décimal
  if (touche === '.' && montant.value.includes('.')) {
    return
  }

  // Limiter le montant à 10 caractères
  if (montant.value.length >= 10 && montant.value !== '0') {
    return
  }

  // Remplacer le zéro initial par le chiffre saisi
  if (montant.value === '0' && touche !== '.') {
    montant.value = touche
  } else {
    montant.value += touche
  }

  // Effacer les anciens résultats et erreurs
  resultat.value = ''
  erreur.value = false
}

// Convertir le montant entre deux devises
function convertir() {
  const valeur = Number(montant.value)

  // Vérifier que le montant est supérieur à zéro
  if (valeur === 0 || montant.value === '') {
    resultat.value = ''
    erreur.value = true
    return
  }

  erreur.value = false

  // Convertir d'abord le montant en euros
  const montantEnEuro = valeur / taux[deviseSource.value]

  // Convertir ensuite les euros vers la devise cible
  const conversion = montantEnEuro * taux[deviseCible.value]

  // Afficher le résultat avec deux décimales
  resultat.value = conversion.toFixed(2)
}

// Échanger les devises source et cible
function inverser() {
  const ancienneSource = deviseSource.value

  deviseSource.value = deviseCible.value
  deviseCible.value = ancienneSource

  // Recalculer si un résultat est déjà affiché
  if (resultat.value !== '') {
    convertir()
  }
}
</script>

<template>
  <div class="convertisseur">

    <!-- Zone supérieure : affichage et devises -->
    <div class="zone-haute">
      <h1>Currency Converter</h1>

      <!-- Devise source et montant saisi -->
      <div class="ligne-devise">
        <select v-model="deviseSource">
          <option
            v-for="d in devises"
            :key="d"
            :value="d"
          >
            {{ d }}
          </option>
        </select>

        <div class="montant">
          {{ montant }}
        </div>
      </div>

      <!-- Bouton d'inversion -->
      <div class="inversion">
        <button @click="inverser" aria-label="Inverser les devises">
          ⇅
        </button>
      </div>

      <!-- Devise cible et résultat -->
      <div class="ligne-devise">
        <select v-model="deviseCible">
          <option
            v-for="d in devises"
            :key="d"
            :value="d"
          >
            {{ d }}
          </option>
        </select>

        <div class="montant">
          {{ resultat || '0.00' }}
        </div>
      </div>

      <!-- Bouton de conversion -->
      <button class="bouton-convertir" @click="convertir">
        Convertir
      </button>

      <!-- Message d'erreur conditionnel -->
      <p v-if="erreur" class="message-erreur">
        Veuillez saisir un montant
      </p>
    </div>

    <!-- Zone inférieure : clavier numérique -->
    <div class="zone-clavier">
      <div class="clavier">
        <button
          v-for="touche in touches"
          :key="touche"
          :class="{ 'touche-effacer': touche === 'C' }"
          @click="appuyer(touche)"
        >
          {{ touche }}
        </button>
      </div>
    </div>

  </div>
</template>

<style scoped>
/* Carte principale */
.convertisseur {
  width: 100%;
  max-width: 360px;
  margin: 30px auto;
  box-sizing: border-box;
  border-radius: 12px;
  overflow: hidden;
  font-family: Arial, sans-serif;
}

/* Zone turquoise */
.zone-haute {
  background-color: #2BB8E8;
  color: white;
  padding: 20px;
}

/* Titre */
h1 {
  text-align: center;
  font-size: 24px;
  margin: 0 0 30px;
}

/* Lignes des devises */
.ligne-devise {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-bottom: 20px;
}

/* Listes des devises */
select {
  width: 100%;
  padding: 8px;
  font-size: 16px;
  border: none;
  border-radius: 5px;
  background-color: white;
  color: #333;
}

/* Montant et résultat */
.montant {
  width: 100%;
  box-sizing: border-box;
  min-height: 60px;
  font-size: 48px;
  text-align: right;
  border-bottom: 2px solid white;
  overflow: hidden;
  overflow-wrap: anywhere;
}

/* Bouton d'inversion */
.inversion {
  text-align: center;
  margin: 5px 0 15px;
}

.inversion button {
  font-size: 28px;
  color: white;
  background: transparent;
  border: none;
  cursor: pointer;
}

/* Bouton Convertir */
.bouton-convertir {
  width: 100%;
  margin-top: 10px;
  padding: 12px;
  border: none;
  border-radius: 6px;
  background-color: white;
  color: #1599BF;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
}

.bouton-convertir:hover {
  background-color: #eefaff;
}

/* Message d'erreur */
.message-erreur {
  color: #ffffff;
  text-align: center;
  font-size: 14px;
  margin: 12px 0 0;
}

/* Zone claire du clavier */
.zone-clavier {
  background-color: #f7f9fa;
  padding: 12px 20px 20px;
}

/* Grille de 4 lignes et 3 colonnes */
.clavier {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 8px;
}

/* Touches numériques */
.clavier button {
  min-width: 0;
  padding: 8px;
  border: none;
  background: transparent;
  color: #1BA6C9;
  font-size: 32px;
  cursor: pointer;
}

/* Touche C en rouge */
.clavier button.touche-effacer {
  color: #F0727A;
}

/* Effet au survol */
.clavier button:hover {
  background-color: #e7f4f8;
  border-radius: 6px;
}

/* Adaptation aux petits écrans */
@media (max-width: 360px) {
  .convertisseur {
    margin: 0 auto;
  }

  .zone-haute {
    padding: 16px;
  }

  .montant {
    font-size: 42px;
  }
}
</style>
