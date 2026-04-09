# Exercice - Liste de tâches : Du HTML statique à Vue 3

## HTML de départ

Avant de créer le projet Vue, voici le HTML statique de référence. Il servira de point de départ à l'étape 3.

```html
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Todo List</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-100 min-h-screen flex items-center justify-center">

  <div class="bg-white rounded-xl shadow-sm p-6 w-full max-w-md">
    <h1 class="text-xl font-medium text-gray-800 mb-4">Ma todo list</h1>

    <div class="flex gap-2 mb-5">
      <input
        type="text"
        placeholder="Ajouter une tâche…"
        class="flex-1 border border-gray-200 rounded-lg px-3 py-2 text-sm outline-none focus:ring-2 focus:ring-blue-200"
      />
      <button class="bg-blue-500 hover:bg-blue-600 text-white text-sm px-4 py-2 rounded-lg">
        Ajouter
      </button>
    </div>

    <p class="text-xs text-gray-400 mb-3">2 tâches restantes</p>

    <ul class="flex flex-col gap-2">
      <li class="flex items-center gap-3 px-3 py-2.5 border border-gray-100 rounded-lg text-sm text-gray-800">
        <input type="checkbox" class="w-4 h-4 accent-blue-500" />
        <span>Apprendre Vue 3</span>
      </li>
      <li class="flex items-center gap-3 px-3 py-2.5 border border-gray-100 rounded-lg text-sm text-gray-400 line-through">
        <input type="checkbox" checked class="w-4 h-4 accent-blue-500" />
        <span>Installer Node.js</span>
      </li>
      <li class="flex items-center gap-3 px-3 py-2.5 border border-gray-100 rounded-lg text-sm text-gray-800">
        <input type="checkbox" class="w-4 h-4 accent-blue-500" />
        <span>Faire l'exercice todo list</span>
      </li>
    </ul>
  </div>

</body>
</html>
```

## Étape 0 — Créer le projet

```bash
npm create vue@latest nom-projet
```

Dans le questionnaire interactif, activer seulement :

- ❌ TypeScript
- ✅ ESLint (optionnel)
- tout le reste → ❌

```bash
cd nom-projet
npm install
```

## Étape 1 - Installer Tailwind

Tailwind s'installe **avant** de lancer le serveur de développement, car il doit être intégré à la chaîne de build de Vite.

Le flag `-D` (ou `--save-dev`) indique à npm d'enregistrer le paquet comme dépendance de développement dans `devDependencies`. Tailwind génère du CSS au moment du build : il n'a plus rien à faire une fois l'app déployée, donc il n'a pas sa place dans les dépendances de **production**.

```bash
npm install -D tailwindcss @tailwindcss/vite
```

Dans `vite.config.js`, ajouter l'import en haut du fichier et `tailwindcss()` dans le tableau `plugins` :

```js
import tailwindcss from '@tailwindcss/vite'   // ajouter cette ligne

export default defineConfig({
  plugins: [
    vue(),
    tailwindcss(),   // ajouter cette ligne
  ],
})
```

Dans `src/assets/main.css`, remplacer tout le contenu par :

```css
@import "tailwindcss";
```

On peut maintenant lancer le serveur :

```bash
npm run dev
```

## Étape 2 - Nettoyer le projet

- Supprimer tout le contenu de `src/components/`
- Dans `App.vue`, vider les balises `<script setup>`, `<style scoped>` et `<template>` (les conserver vides).

## Étape 3 - Coller le HTML du `<body>` dans `App.vue`

C'est le point de départ Vue : le HTML statique de référence est collé tel quel (en remplaçant `<body></body>` par `<div></div>`) dans le `<template>` de `App.vue`. Rien ne bouge encore : c'est voulu.

```vue
<template>
  <div class="bg-gray-100 min-h-screen flex items-center justify-center">
    <div class="bg-white rounded-xl shadow-sm p-6 w-full max-w-md">
      <h1 class="text-xl font-medium text-gray-800 mb-4">Ma todo list</h1>

      <div class="flex gap-2 mb-5">
        <input
          type="text"
          placeholder="Ajouter une tâche…"
          class="flex-1 border border-gray-200 rounded-lg px-3 py-2 text-sm outline-none focus:ring-2 focus:ring-blue-200"
        />
        <button class="bg-blue-500 hover:bg-blue-600 text-white text-sm px-4 py-2 rounded-lg">
          Ajouter
        </button>
      </div>

      <p class="text-xs text-gray-400 mb-3">2 tâches restantes</p>

      <ul class="flex flex-col gap-2">
        <li class="flex items-center gap-3 px-3 py-2.5 border border-gray-100 rounded-lg text-sm text-gray-800">
          <input type="checkbox" class="w-4 h-4 accent-blue-500" />
          <span>Apprendre Vue 3</span>
        </li>
        <li class="flex items-center gap-3 px-3 py-2.5 border border-gray-100 rounded-lg text-sm text-gray-400 line-through">
          <input type="checkbox" checked class="w-4 h-4 accent-blue-500" />
          <span>Installer Node.js</span>
        </li>
        <li class="flex items-center gap-3 px-3 py-2.5 border border-gray-100 rounded-lg text-sm text-gray-800">
          <input type="checkbox" class="w-4 h-4 accent-blue-500" />
          <span>Faire l'exercice todo list</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
</script>
```

✅ **Résultat attendu** : l'app s'affiche identiquement au HTML statique. Tailwind fonctionne, le projet est propre.

## Étape 4 - Rendre les données réactives

On remplace les `<li>` codés en dur par une liste réactive. C'est ici qu'on revoit `ref` et les directives `v-for`, `v-model` et une nouvelle : `:class`.

La méthode `crypto.randomUUID()` génère un identifiant unique universel (UUID) — une chaîne de 36 caractères garantie unique. C'est une meilleure pratique que `Date.now()`, qui peut produire des doublons si deux tâches sont créées très rapidement.

```vue
<script setup>
import { ref } from 'vue'

const todos = ref([
  { id: crypto.randomUUID(), text: 'Apprendre Vue 3', done: false },
  { id: crypto.randomUUID(), text: 'Installer Node.js', done: true },
  { id: crypto.randomUUID(), text: 'Faire l\'exercice todo list', done: false },
])
</script>
```

Dans le template, remplacer les `<li>` statiques :

```html
<ul class="flex flex-col gap-2">
  <li
    v-for="todo in todos"
    :key="todo.id"
    class="flex items-center gap-3 px-3 py-2.5 border border-gray-100 rounded-lg text-sm"
    :class="todo.done ? 'text-gray-400 line-through' : 'text-gray-800'"
  >
    <input type="checkbox" v-model="todo.done" class="w-4 h-4 accent-blue-500" />
    <span>{{ todo.text }}</span>
  </li>
</ul>
```

La clé `:key="todo.id"` permet à Vue d'identifier chaque élément de la liste de manière unique. Sans `:key`, Vue pourrait réutiliser ou réordonner incorrectement des éléments lors des mises à jour, surtout quand on ajoute, supprime ou réorganise des tâches.

## Étape 5 - `computed` : le compteur de tâches restantes

On remplace le texte statique `"2 tâches restantes"` par une valeur calculée automatiquement. C'est l'occasion de revoir `computed`.

```vue
<script setup>
import { ref, computed } from 'vue'

// ...

const remaining = computed(() => todos.value.filter(t => !t.done).length)
</script>
```

Dans le template, remplacer le `<p>` statique :

```html
<p class="text-xs text-gray-400 mb-3">{{ remaining }} tâche(s) restante(s)</p>
```

## Étape 6 - Rendre le formulaire fonctionnel

On ajoute la logique d'ajout d'une tâche pour aborder les événements et la liaison bidirectionnelle sur un champ texte.

```vue
<script setup>
import { ref, computed } from 'vue'

const todos = ref([
  { id: crypto.randomUUID(), text: 'Apprendre Vue 3', done: false },
  { id: crypto.randomUUID(), text: 'Installer Node.js', done: true },
  { id: crypto.randomUUID(), text: 'Faire l\'exercice todo list', done: false },
])

const newTodo = ref('')

const remaining = computed(() => todos.value.filter(t => !t.done).length)

function addTodo() {
  if (!newTodo.value.trim()) return
  todos.value.push({
    id: crypto.randomUUID(),
    text: newTodo.value.trim(),
    done: false,
  })
  newTodo.value = ''
}
</script>
```

Dans le template :

```html
<div class="flex gap-2 mb-5">
  <input
    v-model="newTodo"
    @keyup.enter="addTodo"
    type="text"
    placeholder="Ajouter une tâche…"
    class="flex-1 border border-gray-200 rounded-lg px-3 py-2 text-sm outline-none focus:ring-2 focus:ring-blue-200"
  />
  <button
    @click="addTodo"
    class="bg-blue-500 hover:bg-blue-600 text-white text-sm px-4 py-2 rounded-lg"
  >
    Ajouter
  </button>
</div>
```

## Étape 7 - Persistance des données avec localStorage

Avant d'introduire `Pinia`, la bibliothèque de gestion d'état officielle et recommandée pour Vue.js, on utilise ici `localStorage` directement pour préserver l'état entre les rechargements de page. C'est une solution simple et suffisante pour l'instant : elle servira aussi de motivation pour Pinia plus tard.

On initialise `todos` depuis `localStorage` si des données existent, et on utilise `watch` pour sauvegarder automatiquement à chaque modification.

```vue
<script setup>
import { ref, computed, watch } from 'vue'

const defaultTodos = [
  { id: crypto.randomUUID(), text: 'Apprendre Vue 3', done: false },
  { id: crypto.randomUUID(), text: 'Installer Node.js', done: true },
  { id: crypto.randomUUID(), text: 'Faire l\'exercice todo list', done: false },
]

const stored = localStorage.getItem('todos')
const todos = ref(stored ? JSON.parse(stored) : defaultTodos)

watch(
  todos,
  (newTodos) => localStorage.setItem('todos', JSON.stringify(newTodos)),
  { deep: true }
)

const newTodo = ref('')
const remaining = computed(() => todos.value.filter(t => !t.done).length)

function addTodo() {
  if (!newTodo.value.trim()) return
  todos.value.push({
    id: crypto.randomUUID(),
    text: newTodo.value.trim(),
    done: false,
  })
  newTodo.value = ''
}
</script>
```

> **Note** : `watch` avec l'option `{ deep: true }` surveille les modifications à l'intérieur des objets du tableau (comme le changement de `done`), pas seulement les ajouts ou suppressions.

## Étape 8 - Extraire le composant `TodoItem`

C'est le cœur pédagogique de l'exercice : montrer pourquoi et comment on extrait un composant. Chaque `<li>` devient un composant autonome qui reçoit une tâche via une **prop** et communique avec le parent via un **événement**.

Créer `src/components/TodoItem.vue` :

```vue
<template>
  <li
    class="flex items-center gap-3 px-3 py-2.5 border border-gray-100 rounded-lg text-sm"
    :class="todo.done ? 'text-gray-400 line-through' : 'text-gray-800'"
  >
    <input
      type="checkbox"
      :checked="todo.done"
      @change="$emit('toggle', todo.id)"
      class="w-4 h-4 accent-blue-500"
    />
    <span>{{ todo.text }}</span>
  </li>
</template>

<script setup>
defineProps({ todo: Object })
defineEmits(['toggle'])
</script>
```

Dans `App.vue`, importer le composant et ajouter la fonction `toggleTodo` :

```vue
<script setup>
import { ref, computed, watch } from 'vue'
import TodoItem from './components/TodoItem.vue'

// ... (tout le code de l'étape précédente)

function toggleTodo(id) {
  const todo = todos.value.find(t => t.id === id)
  if (todo) todo.done = !todo.done
}
</script>
```

Remplacer les `<li>` dans le template :

```html
<ul class="flex flex-col gap-2">
  <TodoItem
    v-for="todo in todos"
    :key="todo.id"
    :todo="todo"
    @toggle="toggleTodo"
  />
</ul>
```

✅ **Notions abordées** : `defineProps`, `defineEmits`, communication parent → enfant (props), enfant → parent (événements).

## Étape 9 - Extraire les composants `TodoForm` et `TodoList`

On pousse la décomposition plus loin en extrayant les deux autres sections de l'interface.

### `TodoForm.vue`

Ce composant gère le champ texte et le bouton. Il émet un événement `add` avec le texte de la nouvelle tâche — il ne touche pas directement à la liste.

Créer `src/components/TodoForm.vue` :

```vue
<template>
  <div class="flex gap-2 mb-5">
    <input
      v-model="newTodo"
      @keyup.enter="submit"
      type="text"
      placeholder="Ajouter une tâche…"
      class="flex-1 border border-gray-200 rounded-lg px-3 py-2 text-sm outline-none focus:ring-2 focus:ring-blue-200"
    />
    <button
      @click="submit"
      class="bg-blue-500 hover:bg-blue-600 text-white text-sm px-4 py-2 rounded-lg"
    >
      Ajouter
    </button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const emit = defineEmits(['add'])
const newTodo = ref('')

function submit() {
  if (!newTodo.value.trim()) return
  emit('add', newTodo.value.trim())
  newTodo.value = ''
}
</script>
```

### `TodoList.vue`

Ce composant affiche le compteur et la liste. Il reçoit `todos` en prop et relaie l'événement `toggle` vers le parent.

Créer `src/components/TodoList.vue` :

```vue
<template>
  <div>
    <p class="text-xs text-gray-400 mb-3">{{ remaining }} tâche(s) restante(s)</p>
    <ul class="flex flex-col gap-2">
      <TodoItem
        v-for="todo in todos"
        :key="todo.id"
        :todo="todo"
        @toggle="$emit('toggle', $event)"
      />
    </ul>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import TodoItem from './TodoItem.vue'

const props = defineProps({ todos: Array })
defineEmits(['toggle'])

const remaining = computed(() => props.todos.filter(t => !t.done).length)
</script>
```

### `App.vue` final

`App.vue` devient un simple coordinateur : il détient le state et orchestre les composants.

```vue
<template>
  <div class="bg-gray-100 min-h-screen flex items-center justify-center">
    <div class="bg-white rounded-xl shadow-sm p-6 w-full max-w-md">
      <h1 class="text-xl font-medium text-gray-800 mb-4">Ma todo list</h1>
      <TodoForm @add="addTodo" />
      <TodoList :todos="todos" @toggle="toggleTodo" />
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'
import TodoForm from './components/TodoForm.vue'
import TodoList from './components/TodoList.vue'

const defaultTodos = [
  { id: crypto.randomUUID(), text: 'Apprendre Vue 3', done: false },
  { id: crypto.randomUUID(), text: 'Installer Node.js', done: true },
  { id: crypto.randomUUID(), text: 'Faire l\'exercice todo list', done: false },
]

const stored = localStorage.getItem('todos')
const todos = ref(stored ? JSON.parse(stored) : defaultTodos)

watch(
  todos,
  (newTodos) => localStorage.setItem('todos', JSON.stringify(newTodos)),
  { deep: true }
)

function addTodo(text) {
  todos.value.push({ id: crypto.randomUUID(), text, done: false })
}

function toggleTodo(id) {
  const todo = todos.value.find(t => t.id === id)
  if (todo) todo.done = !todo.done
}
</script>
```

✅ **Notions abordées** : décomposition de l'interface en composants responsables d'une seule chose, propagation d'événements entre plusieurs niveaux.

## Étape 10 - Créer le dépôt GitHub

### Initialiser Git

À la racine du projet :

```bash
git init
git add .
git commit -m "feat: todo list Vue 3 complète"
```

### Créer le dépôt sur GitHub

1. Sur [github.com](https://github.com), cliquer sur **New repository**
2. Nommer le dépôt `todo-app`, le mettre en **Private**
3. Ne pas initialiser avec un README (le projet en a déjà un)

### Pousser le projet

```bash
git remote add origin https://github.com/votre-username/todo-app.git
git branch -M main
git push -u origin main
```

### Ajouter le professeur comme collaborateur (lecture seule)

1. Dans le dépôt GitHub, aller dans **Settings → Collaborators**
2. Cliquer sur **Add people**
3. Entrer le nom d'utilisateur GitHub du professeur
4. Sélectionner le rôle **Read**

Le professeur recevra une invitation par courriel à accepter.

---

## Vue d'ensemble de la progression

| Étape | Notion Vue introduite |
| --- | --- |
| 3 | Structure d'un SFC (`.vue`) |
| 4 | `ref`, `v-for`, `v-model`, `:class` |
| 5 | `computed` |
| 6 | `@click`, `@keyup.enter`, mutation de l'état |
| 7 | `watch`, `localStorage` |
| 8 | `defineProps`, `defineEmits`, composants |
| 9 | Décomposition avancée, propagation d'événements |
| 10 | Git, GitHub, collaboration |
