# Fiche 06 — Les appels API

> **Application de référence** : charger les données de l'API restcountries.com dans les pages Archive et Détails.

## 1. Le principe

Vue ne fournit pas d'outil intégré pour les appels API — on utilise soit le `fetch` natif de JavaScript (disponible dans tous les navigateurs), soit la librairie **Axios**.

Dans l'application de référence, et de manière plus générale avec Vue.js, on utilise **`fetch` avec `async/await`**, mais l'utilisation des promesses avec `.then()` et `.catch()` peut aussi être utilisée.

## 2. `async/await` — la syntaxe à connaître

`async/await` permet d'écrire du code asynchrone sans enchaîner des `.then()`.

```js
// Sans async/await
fetch('https://api.exemple.com/data')
  .then(response => response.json())
  .then(data => console.log(data))

// Avec async/await
async function chargerDonnees() {
  const response = await fetch('https://api.exemple.com/data')
  const data = await response.json()
  console.log(data)
}
```

> **`await`** met l'exécution en pause jusqu'à ce que la promesse soit résolue. Il ne fonctionne qu'à l'intérieur d'une fonction `async`.

## 3. Charger une liste — page Archive

Le bon endroit pour un appel API est dans `onMounted()`, qui s'exécute quand le composant est affiché.

> **Références** :  
[Les hooks du cycle de vie](https://fr.vuejs.org/guide/essentials/lifecycle.html)  
[Composition API: Les hooks du cycle de vie](https://fr.vuejs.org/api/composition-api-lifecycle.html)

```vue
<!-- src/views/ArchiveView.vue -->
<template>
  <div>
    <h2>Pays du monde</h2>

    <!-- États de chargement et d'erreur -->
    <p v-if="isLoading">⏳ Chargement…</p>
    <p v-else-if="erreur" style="color: red;">{{ erreur }}</p>

    <!-- Données chargées -->
    <ul v-else>
      <li v-for="pays in listePays" :key="pays.cca3">
        <img :src="pays.flags.svg" :alt="pays.name.common" width="30" />
        {{ pays.name.common }}
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const listePays = ref([])
const isLoading = ref(true)
const erreur = ref(null)

onMounted(async () => {
  try {
    const response = await fetch(
      'https://restcountries.com/v3.1/all?fields=name,flags,cca3,population,region'
    )

    if (!response.ok) {
      throw new Error(`Erreur HTTP : ${response.status}`)
    }

    const data = await response.json()

    // Trier par nom alphabétique
    listePays.value = data.sort((a, b) =>
      a.name.common.localeCompare(b.name.common)
    )
  } catch (e) {
    erreur.value = 'Impossible de charger les données. Réessayez plus tard.'
    console.error(e)
  } finally {
    isLoading.value = false
  }
})
</script>
```

> **Toujours gérer trois états** : `isLoading` (en cours), `erreur` (échec), et les données (succès). C'est une bonne pratique indispensable.

## 4. Charger un seul élément — page Détails

```vue
<!-- src/views/DetailsView.vue -->
<template>
  <div>
    <button @click="retour">← Retour</button>

    <p v-if="isLoading">⏳ Chargement…</p>
    <p v-else-if="erreur">{{ erreur }}</p>

    <article v-else-if="pays">
      <h2>{{ pays.name.common }}</h2>
      <h3>{{ pays.name.official }}</h3>
      <img :src="pays.flags.svg" :alt="pays.name.common" width="200" />

      <table>
        <tr><th>Capitale</th><td>{{ pays.capital?.[0] ?? 'N/D' }}</td></tr>
        <tr><th>Région</th><td>{{ pays.region }}</td></tr>
        <tr><th>Population</th><td>{{ pays.population.toLocaleString() }}</td></tr>
        <tr><th>Superficie</th><td>{{ pays.area.toLocaleString() }} km²</td></tr>
        <tr><th>Langues</th><td>{{ langues }}</td></tr>
      </table>
    </article>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'

const route = useRoute()
const router = useRouter()

const pays = ref(null)
const isLoading = ref(true)
const erreur = ref(null)

// Formater les langues depuis l'objet de l'API
const langues = computed(() => {
  if (!pays.value?.languages) return 'N/D'
  return Object.values(pays.value.languages).join(', ')
})

onMounted(async () => {
  try {
    const code = route.params.code
    const response = await fetch(
      `https://restcountries.com/v3.1/alpha/${code}?fields=name,flags,capital,region,population,area,languages`
    )

    if (!response.ok) throw new Error(`Pays introuvable (code : ${code})`)

    const data = await response.json()
    pays.value = data
  } catch (e) {
    erreur.value = e.message
  } finally {
    isLoading.value = false
  }
})

function retour() {
  router.push({ name: 'archive' })
}
</script>
```

## 5. Les endpoints de l'API restcountries

| Objectif | URL |
| --- | --- |
| Tous les pays | `https://restcountries.com/v3.1/all` |
| Tous les pays (champs limités) | `https://restcountries.com/v3.1/all?fields=name,flags,cca3` |
| Un pays par code | `https://restcountries.com/v3.1/alpha/CAN` |
| Pays par région | `https://restcountries.com/v3.1/region/europe` |
| Pays par nom | `https://restcountries.com/v3.1/name/canada` |

> **Bonne pratique** : utilisez toujours `?fields=...` pour ne demander que les champs dont vous avez besoin — ça accélère le chargement.

## 6. Extraire la logique dans un fichier séparé (bonne pratique MVC)

Pour garder les composants simples, on peut déplacer les appels API dans un fichier **service** (le "Model" dans notre parallèle MVC) :

```js
// src/services/paysService.js
const BASE_URL = 'https://restcountries.com/v3.1'

export async function getTousLesPays() {
  const response = await fetch(`${BASE_URL}/all?fields=name,flags,cca3,population,region`)
  if (!response.ok) throw new Error('Erreur lors du chargement des pays')
  const data = await response.json()
  return data.sort((a, b) => a.name.common.localeCompare(b.name.common))
}

export async function getPaysParCode(code) {
  const response = await fetch(`${BASE_URL}/alpha/${code}`)
  if (!response.ok) throw new Error(`Pays introuvable : ${code}`)
  return await response.json()
}
```

```vue
<!-- Dans ArchiveView.vue — le composant reste simple -->
<script setup>
import { ref, onMounted } from 'vue'
import { getTousLesPays } from '@/services/paysService.js'

const listePays = ref([])
const isLoading = ref(true)
const erreur = ref(null)

onMounted(async () => {
  try {
    listePays.value = await getTousLesPays()
  } catch (e) {
    erreur.value = e.message
  } finally {
    isLoading.value = false
  }
})
</script>
```

## À retenir

- Faites les appels API dans `onMounted()` avec `async/await`
- Gérez toujours **trois états** : `isLoading`, `erreur`, données
- Utilisez `try/catch/finally` pour capturer les erreurs
- Vérifiez `response.ok` après `fetch()`
- Isolez vos appels dans un fichier `services/` pour une meilleure séparation des responsabilités

---

*Prochaine fiche → **Fiche 07 : Gestion d'état global avec Pinia***
