# Fiche 03 — Réactivité et état

> **Application de référence** : gérer l'état de la recherche, des filtres et des données chargées depuis l'API pays.

## 1. C'est quoi la réactivité?

En Vue, une variable **réactive** est une variable que Vue *surveille*. Quand sa valeur change, Vue **met à jour le HTML automatiquement** — pas besoin de manipuler le DOM manuellement.

Sans réactivité, modifier une variable ne changerait rien à l'écran.

## 2. `ref()` : pour les valeurs simples

`ref()` rend une valeur réactive. On l'utilise pour les types **primitifs** : texte, nombre, booléen.

```vue
<template>
  <p>Recherche : {{ recherche }}</p>
  <input v-model="recherche" placeholder="Nom du pays…" />
</template>

<script setup>
import { ref } from 'vue'

const recherche = ref('')  // chaîne vide au départ
</script>
```

> **Important** : dans `<script setup>`, on accède à la valeur avec `.value` (ex. : `recherche.value`). Dans le `<template>`, Vue le fait automatiquement — on écrit juste `{{ recherche }}`.

```js
// Dans <script setup>
console.log(recherche.value)   // ''
recherche.value = 'Canada'
console.log(recherche.value)   // 'Canada'
```

## 3. `reactive()` : pour les objets

`reactive()` rend un **objet entier** réactif. On n'utilise pas `.value` ici.

```vue
<script setup>
import { reactive } from 'vue'

const filtres = reactive({
  region: '',
  population: 'tous',
  recherche: ''
})

// On accède directement aux propriétés
filtres.region = 'Europe'
</script>
```

> **Règle simple** : utilisez `ref()` pour une valeur unique, `reactive()` pour un groupe de valeurs liées.

## 4. `computed()` : des valeurs dérivées

`computed()` crée une variable dont la valeur est **calculée automatiquement** à partir d'autres variables réactives. Elle se recalcule seulement quand ses dépendances changent.

```vue
<template>
  <input v-model="recherche" placeholder="Chercher…" />
  <p>{{ paysFiltres.length }} résultat(s)</p>
  <ul>
    <li v-for="pays in paysFiltres" :key="pays.cca3">
      {{ pays.name.common }}
    </li>
  </ul>
</template>

<script setup>
import { ref, computed } from 'vue'

const recherche = ref('')

const listePays = ref([
  { cca3: 'CAN', name: { common: 'Canada' } },
  { cca3: 'FRA', name: { common: 'France' } },
  { cca3: 'JPN', name: { common: 'Japan' } },
  { cca3: 'BRA', name: { common: 'Brazil' } }
])

// Se recalcule automatiquement quand "recherche" ou "listePays" change
const paysFiltres = computed(() => {
  if (!recherche.value) return listePays.value
  return listePays.value.filter(pays =>
    pays.name.common.toLowerCase().includes(recherche.value.toLowerCase())
  )
})
</script>
```

## 5. `watch()` : réagir aux changements

`watch()` permet d'**exécuter du code** en réaction au changement d'une variable réactive.

```vue
<script setup>
import { ref, watch } from 'vue'

const region = ref('')

// Déclenché chaque fois que "region" change
watch(region, (nouvelleValeur, ancienneValeur) => {
  console.log(`Région changée : ${ancienneValeur} → ${nouvelleValeur}`)
  // On pourrait relancer un appel API ici
})
</script>
```

> **`computed` vs `watch`** : utilisez `computed` pour **calculer** une valeur dérivée. Utilisez `watch` pour **déclencher une action** (appel API, log, redirection…).

## 6. Exemple complet : liste de pays avec recherche

```vue
<!-- src/views/ArchiveView.vue (version simplifiée) -->
<template>
  <div>
    <h2>Archive des pays</h2>

    <input v-model="recherche" placeholder="Chercher un pays…" />

    <p v-if="isLoading">Chargement…</p>
    <ul v-else>
      <li v-for="pays in paysFiltres" :key="pays.cca3">
        <img :src="pays.flags.svg" :alt="pays.name.common" width="30" />
        {{ pays.name.common }}
        — Population : {{ pays.population.toLocaleString() }}
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const listePays = ref([])
const recherche = ref('')
const isLoading = ref(true)

// Valeur dérivée : liste filtrée selon la recherche
const paysFiltres = computed(() => {
  if (!recherche.value) return listePays.value
  return listePays.value.filter(pays =>
    pays.name.common.toLowerCase().includes(recherche.value.toLowerCase())
  )
})

// Chargement des données au montage du composant
onMounted(async () => {
  const response = await fetch('https://restcountries.com/v3.1/all?fields=name,flags,population,cca3')
  listePays.value = await response.json()
  isLoading.value = false
})
</script>
```

## À retenir

| Outil | Pour quoi |
| --- | --- |
| `ref()` | Valeur réactive simple (texte, nombre, booléen, tableau) |
| `reactive()` | Objet réactif avec plusieurs propriétés |
| `computed()` | Valeur calculée automatiquement depuis l'état |
| `watch()` | Déclencher une action quand une valeur change |

---

*Prochaine fiche → **Fiche 04 : Composants***
