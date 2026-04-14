# Exercice 2 - Ajout de Vue Router, Header, Footer et Views

## Structure finale visée

```text
src/
├── assets/
├── components/
│   ├── HeaderComponent.vue   ← nouveau
│   ├── FooterComponent.vue   ← nouveau
│   ├── TodoForm.vue
│   ├── TodoItem.vue
│   └── TodoList.vue
├── router/
│   └── index.js              ← nouveau
├── views/
│   ├── AccueilView.vue       ← nouveau
│   └── TachesView.vue        ← nouveau (contenu extrait de App.vue)
├── App.vue                   ← modifié (layout seulement)
└── main.js                   ← modifié (use router)
```

## Étape 1 — Installer Vue Router

```bash
npm install vue-router
```

## Étape 2 — Créer `HeaderComponent.vue`

Créer le composant pour l'entête **`src/components/HeaderComponent.vue`**

```vue
<template>
  <header class="bg-white border-b border-gray-200 sticky top-0 z-10">
    <div class="max-w-4xl mx-auto px-4 h-14 flex items-center justify-between">
      <RouterLink to="/" class="text-blue-500 font-semibold text-base tracking-tight">
        ✅ Todo App
      </RouterLink>
      <nav class="flex gap-6 text-sm">
        <RouterLink
          to="/"
          class="text-gray-500 hover:text-blue-500 transition-colors"
          active-class="text-blue-500 font-medium"
        >
          Accueil
        </RouterLink>
        <RouterLink
          to="/taches"
          class="text-gray-500 hover:text-blue-500 transition-colors"
          active-class="text-blue-500 font-medium"
        >
          Tâches
        </RouterLink>
      </nav>
    </div>
  </header>
</template>
```

## Étape 3 — Créer `FooterComponent.vue`

Créer le composant pour le pied de page **`src/components/FooterComponent.vue`**

```vue
<template>
  <footer class="border-t border-gray-200 mt-auto">
    <div class="max-w-4xl mx-auto px-4 h-12 flex items-center justify-center">
      <p class="text-xs text-gray-400">
        © {{ new Date().getFullYear() }} Todo App — Fait avec Vue 3 & Tailwind CSS
      </p>
    </div>
  </footer>
</template>
```

## Étape 4 — Créer les views

Créer le dossier `src/views/`.

Créer la page d'accueil **`src/views/AccueilView.vue`**

```vue
<template>
  <main class="max-w-4xl mx-auto px-4 py-16 flex flex-col items-center text-center gap-6">
    <div class="text-5xl">✅</div>
    <h1 class="text-3xl font-semibold text-gray-800">
      Bienvenue sur Todo App
    </h1>
    <p class="text-gray-500 max-w-sm leading-relaxed">
      Une application simple pour gérer tes tâches au quotidien.
      Ajoute, coche, et retrouve tes tâches en un clin d'œil.
    </p>
    <RouterLink
      to="/taches"
      class="bg-blue-500 hover:bg-blue-600 text-white text-sm font-medium px-6 py-2.5 rounded-lg transition-colors"
    >
      Voir mes tâches →
    </RouterLink>
  </main>
</template>
```

Créer la page des tâches **`src/views/TachesView.vue`** : extraire tout de `App.vue` :

```vue
<template>
  <main class="flex items-center justify-center py-10 px-4">
    <div class="bg-white rounded-xl shadow-sm p-6 w-full max-w-md">
      <h1 class="text-xl font-medium text-gray-800 mb-4">Ma todo list</h1>
      <TodoForm @add="addTodo" />
      <TodoList :todos="todos" @toggle="toggleTodo" />
    </div>
  </main>
</template>

<script setup>
import { ref, watch } from 'vue';
import TodoForm from '../components/TodoForm.vue';
import TodoList from '../components/TodoList.vue';

const defaultTodos = [
  { id: 1, text: 'Apprendre Vue 3', done: false },
  { id: 2, text: 'Installer Node.js', done: true },
  { id: 3, text: 'Faire l\'exercice de la liste de tâches', done: false },
]

const stored = localStorage.getItem('todos')
const todos = ref(stored ? JSON.parse(stored) : defaultTodos)

watch(todos, (newTodos) => localStorage.setItem('todos', JSON.stringify(newTodos)), { deep: true })

function addTodo(text) {
  todos.value.push({ id: crypto.randomUUID(), text, done: false })
}

function toggleTodo(id) {
  const todo = todos.value.find(t => t.id === id)
  if (todo) todo.done = !todo.done
}
</script>
```

## Étape 5 — Créer le router

Créer le fichier pour le routeur et les routes **`src/router/index.js`**

```vue
import { createRouter, createWebHistory } from 'vue-router'
import AccueilView from '../views/AccueilView.vue'
import TachesView from '../views/TachesView.vue'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    { path: '/', component: AccueilView },
    { path: '/taches', component: TachesView },
  ],
})

export default router
```

## Étape 6 — Mettre à jour `main.js`

```vue
import './assets/main.css'
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

createApp(App).use(router).mount('#app')
```

## Étape 7 — Refaire `App.vue`

Remplacer tout le contenu par le layout avec `HeaderComponent`, `RouterView` et `FooterComponent` :

```vue
<template>
  <div class="bg-gray-100 min-h-screen flex flex-col">
    <HeaderComponent />
    <RouterView />
    <FooterComponent />
  </div>
</template>

<script setup>
import HeaderComponent from './components/HeaderComponent.vue'
import FooterComponent from './components/FooterComponent.vue'
</script>
```

## Étape 8 — Pousser sur GitHub

```bash
git add .
git commit -m "feat: ajout Vue Router, HeaderComponent, FooterComponent, views Accueil et Tâches"
git push
```
