# Procédure : Intégration de Pinia dans le projet Liste de tâches

## Objectif

Installer Pinia et créer deux stores pour centraliser l'état global de l'application :

- **`useTodoStore`** — gestion des tâches (remplace la logique locale de `TachesView.vue`)
- **`usePaysStore`** — gestion des pays avec mise en cache (remplace la logique locale de `PaysView.vue`)

---

## Étape 1 — Installer Pinia

```bash
npm install pinia
```

## Étape 2 — Enregistrer Pinia dans `main.js`

```js
import { createApp } from 'vue'
import { createPinia } from 'pinia'   // ← ajouter
import App from './App.vue'
import router from './router'

createApp(App)
  .use(createPinia())   // ← avant .use(router)
  .use(router)
  .mount('#app')
```

> **Pourquoi insérer Pinia avant le router ?**  
> Vue Router peut déclencher des *guards* de navigation (`beforeEach`, etc.) dès son initialisation. Si un *guard* accède à un store Pinia avant que `createPinia()` soit enregistré sur l'application, Pinia n'est pas encore actif et une erreur est levée. En déclarant Pinia en premier, on s'assure qu'il est disponible pour tout ce qui s'initialise ensuite, y compris le router.

## Étape 3 — Créer le dossier `src/stores/`

```text
src/
└── stores/       ← créer ce dossier
```

C'est la convention recommandée par Pinia. Chaque store aura son propre fichier dans ce dossier.

---

## Étape 4 — Créer `src/stores/todoStore.js`

Ce store reprend toute la logique actuellement présente dans `TachesView.vue` :
état des tâches, persistance `localStorage`, et les deux actions `addTodo` / `toggleTodo`.

```js
// src/stores/todoStore.js
import { defineStore } from 'pinia'
import { ref, watch } from 'vue'

const defaultTodos = [
  { id: 1, text: 'Apprendre Vue 3', done: false },
  { id: 2, text: 'Installer Node.js', done: true },
  { id: 3, text: 'Faire l\'exercice de la liste de tâches', done: false },
]

export const useTodoStore = defineStore('todo', () => {
  const stored = localStorage.getItem('todos')
  const todos = ref(stored ? JSON.parse(stored) : defaultTodos)

  // Persistance automatique dans localStorage à chaque changement
  watch(todos, (val) => localStorage.setItem('todos', JSON.stringify(val)), { deep: true })

  function addTodo(text) {
    todos.value.push({ id: crypto.randomUUID(), text, done: false })
  }

  function toggleTodo(id) {
    const todo = todos.value.find(t => t.id === id)
    if (todo) todo.done = !todo.done
  }

  return { todos, addTodo, toggleTodo }
})
```

---

## Étape 5 — Mettre à jour `TachesView.vue`

Supprimer toute la logique locale (refs, watch, fonctions) et la remplacer par le store.

```vue
<!-- src/views/TachesView.vue -->
<template>
  <div class="bg-gray-100 min-h-screen flex items-center justify-center">
    <div class="bg-gray-100 min-h-screen flex items-center justify-center">
      <div class="bg-white rounded-xl shadow-sm p-6 w-full max-w-md">
        <h1 class="text-xl font-medium text-gray-800 mb-4">Ma todo list</h1>
        <TodoForm @add="store.addTodo" />
        <TodoList :todos="store.todos" @toggle="store.toggleTodo" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { useTodoStore } from '@/stores/todoStore'
import TodoForm from '@/components/TodoForm.vue'
import TodoList from '@/components/TodoList.vue'

const store = useTodoStore()
</script>
```

---

## Étape 6 — Créer `src/stores/paysStore.js`

Ce store reprend la logique de `PaysView.vue` : appel API, état de chargement et d'erreur.
La ligne `if (pays.value.length > 0) return` est le mécanisme de **cache** :
si les données ont déjà été chargées, on ne refait pas l'appel API lors d'un retour sur la page.

```js
// src/stores/paysStore.js
import { defineStore } from 'pinia'
import { ref } from 'vue'
import { getPaysParRegion } from '@/services/paysService'

export const usePaysStore = defineStore('pays', () => {
  const pays = ref([])
  const chargement = ref(false)
  const erreur = ref(null)

  async function chargerPays() {
    if (pays.value.length > 0) return  // cache : évite un 2e appel API

    chargement.value = true
    erreur.value = null
    try {
      pays.value = await getPaysParRegion()
    } catch (e) {
      erreur.value = e.message
    } finally {
      chargement.value = false
    }
  }

  return { pays, chargement, erreur, chargerPays }
})
```

## Étape 7 — Mettre à jour `PaysView.vue`

### 7.1 Dans le `<script>`

Remplacer les refs locaux et le `onMounted` par le store. La variable `recherche` reste locale
car elle représente un état d'interface propre à cette vue, pas un état global partagé.

Pour la `computed` (`paysFiltres`), il faut remplacer l'ancien tableau `pays` par `paysStore.pays`, son équivalent.

### 7.2 Dans le `<template>`

Il faut remplacer les occurrences de `chargement` et `erreur` par les valeurs équivalentes en provenance du store : `paysStore.chargement` et `paysStore.erreur`.

```vue
<!-- src/views/PaysView.vue -->
<script setup>
import { ref, computed, onMounted } from 'vue'
import { usePaysStore } from '@/stores/paysStore'

const paysStore = usePaysStore()
const recherche = ref('')   // état local à la vue, pas besoin d'un store

onMounted(() => paysStore.chargerPays())

const paysFiltres = computed(() =>
  paysStore.pays.filter(p =>
    p.name.common.toLowerCase().includes(recherche.value.toLowerCase())
  )
)
</script>

<template>
  <main class="max-w-4xl mx-auto px-4 py-10">
    <h1 class="text-2xl font-semibold text-gray-800 mb-6">Pays des Amériques</h1>

    <p v-if="paysStore.chargement" class="text-gray-400">Chargement...</p>
    <p v-else-if="paysStore.erreur" class="text-red-500">{{ paysStore.erreur }}</p>

    <div v-else>
      <div class="flex items-center gap-2 mb-4">
        <input
          v-model="recherche"
          type="text"
          placeholder="Rechercher un pays..."
          class="border border-gray-300 rounded-lg px-4 py-2 text-sm w-64 focus:outline-none focus:ring-2 focus:ring-blue-300"
        />
        <button
          v-if="recherche"
          @click="recherche = ''"
          class="text-sm text-gray-400 hover:text-red-400 transition-colors"
        >
          ✕ Effacer
        </button>
      </div>

      <p class="text-sm text-gray-400 mb-4">{{ paysFiltres.length }} pays affichés</p>

      <div class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 gap-4">
        <RouterLink
          v-for="p in paysFiltres"
          :key="p.name.common"
          :to="`/pays/${encodeURIComponent(p.name.common)}`"
          class="bg-white rounded-xl shadow-sm p-4 flex flex-col items-center
                 gap-2 hover:shadow-md transition-shadow"
        >
          <img :src="p.flags.svg" :alt="p.name.common" class="w-16 h-10 object-cover rounded" />
          <span class="text-sm text-center text-gray-700">{{ p.name.common }}</span>
        </RouterLink>
      </div>

      <p v-if="paysFiltres.length === 0" class="text-gray-400 mt-6">
        Aucun pays ne correspond à « {{ recherche }} ».
      </p>
    </div>
  </main>
</template>
```

## Étape 8 — `PaysDetailView.vue` — aucune modification requise

`PaysDetailView.vue` appelle directement `getPaysParNom()` depuis `paysService.js` pour charger
les détails d'**un seul pays** à la demande. Ce cas d'usage ne bénéficie pas d'un store
(les données de détail varient selon le pays affiché et ne sont pas partagées entre vues),
donc ce fichier reste inchangé.

---

## Structure finale du projet

**Fichiers modifiés :** `main.js`, `TachesView.vue`, `PaysView.vue`  
**Fichiers ajoutés :** `src/stores/todoStore.js`, `src/stores/paysStore.js`  
**Fichiers inchangés :** tous les autres

```text
liste-de-taches/
├── public/
│   └── favicon.ico
├── src/
│   ├── assets/
│   │   ├── base.css
│   │   ├── logo.svg
│   │   └── main.css
│   ├── components/
│   │   ├── FooterComponent.vue
│   │   ├── HeaderComponent.vue
│   │   ├── TodoForm.vue
│   │   ├── TodoItem.vue
│   │   └── TodoList.vue
│   ├── router/
│   │   └── index.js
│   ├── services/
│   │   └── paysService.js
│   ├── stores/                  ← nouveau
│   │   ├── paysStore.js         ← nouveau
│   │   └── todoStore.js         ← nouveau
│   ├── views/
│   │   ├── AccueilView.vue
│   │   ├── PaysDetailView.vue
│   │   ├── PaysView.vue         ← modifié
│   │   └── TachesView.vue       ← modifié
│   ├── App.vue
│   └── main.js                  ← modifié
├── index.html
├── jsconfig.json
├── package.json
└── vite.config.js
```

---

**Vous voilà fin(e) prêt(e) pour compléter votre application Vue.js!**

*Bon travail!*
