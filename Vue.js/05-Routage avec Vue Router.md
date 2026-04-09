# Fiche 05 — Routage avec Vue Router

> **Application de référence** : naviguer entre l'Accueil, l'Archive des pays, la page Détails d'un pays, et le Formulaire — sans rechargement de page.

## 1. C'est quoi le routage?

Dans une application Vue, toute la navigation se fait **sans recharger la page**. Vue Router gère ce qu'on appelle une **SPA** (Single Page Application) : une seule page HTML dont le contenu change selon l'URL.

## 2. Installation et configuration

Si vous avez coché "Vue Router" lors de `npm create vue@latest`, il est déjà en place. Sinon :

```bash
npm install vue-router
```

La configuration des routes se fait dans `src/router/index.js` :

```js
// src/router/index.js
import { createRouter, createWebHistory } from 'vue-router'
import AccueilView from '@/views/AccueilView.vue'
import ArchiveView from '@/views/ArchiveView.vue'
import DetailsView from '@/views/DetailsView.vue'
import FormulaireView from '@/views/FormulaireView.vue'

const routes = [
  {
    path: '/',
    name: 'accueil',
    component: AccueilView
  },
  {
    path: '/pays',
    name: 'archive',
    component: ArchiveView
  },
  {
    path: '/pays/:code',        // :code est un paramètre dynamique
    name: 'details',
    component: DetailsView
  },
  {
    path: '/formulaire',
    name: 'formulaire',
    component: FormulaireView
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```

## 3. `<RouterView>` et `<RouterLink>`

Deux composants sont au cœur de Vue Router :

**`<RouterView>`** — l'emplacement où la page courante s'affiche :

```vue
<!-- src/App.vue -->
<template>
  <header>
    <nav>
      <RouterLink to="/">Accueil</RouterLink>
      <RouterLink to="/pays">Archive</RouterLink>
      <RouterLink to="/formulaire">Formulaire</RouterLink>
    </nav>
  </header>

  <main>
    <RouterView />   <!-- la page active s'affiche ici -->
  </main>

  <footer>
    <p>© 2025 Pays du monde</p>
  </footer>
</template>
```

> **`<RouterLink>`** génère un `<a>` qui ne recharge pas la page. N'utilisez pas `<a href="...">` pour naviguer entre les pages Vue!

## 4. Les paramètres de route

Pour afficher les détails d'un pays, on passe son code dans l'URL : `/pays/CAN`.

**Récupérer le paramètre dans la page Détails :**

```vue
<!-- src/views/DetailsView.vue -->
<template>
  <div v-if="pays">
    <h2>{{ pays.name.common }}</h2>
    <img :src="pays.flags.svg" :alt="pays.name.common" width="150" />
    <p>Capitale : {{ pays.capital?.[0] }}</p>
    <p>Population : {{ pays.population.toLocaleString() }}</p>
    <p>Région : {{ pays.region }}</p>
  </div>
  <p v-else-if="isLoading">Chargement…</p>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()       // accès aux infos de la route courante
const pays = ref(null)
const isLoading = ref(true)

onMounted(async () => {
  const code = route.params.code   // récupère "CAN" depuis /pays/CAN
  const response = await fetch(`https://restcountries.com/v3.1/alpha/${code}`)
  const data = await response.json()
  pays.value = data[0]
  isLoading.value = false
})
</script>
```

## 5. Navigation programmatique

Parfois on veut naviguer depuis le code (ex. : après un clic sur une carte de pays) :

```vue
<script setup>
import { useRouter } from 'vue-router'

const router = useRouter()

function allerVersDetails(codePays) {
  // Navigation vers /pays/CAN par exemple
  router.push({ name: 'details', params: { code: codePays } })
}
</script>
```

## 6. Lien dans `main.js`

Pour que le routeur soit disponible dans toute l'application, on le déclare dans `main.js` :

```js
// src/main.js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

createApp(App)
  .use(router)
  .mount('#app')
```

## 7. Vue d'ensemble de la navigation dans l'application

```text
URL                  →  Composant affiché
─────────────────────────────────────────
/                    →  AccueilView.vue
/pays                →  ArchiveView.vue
/pays/CAN            →  DetailsView.vue  (code = 'CAN')
/pays/FRA            →  DetailsView.vue  (code = 'FRA')
/formulaire          →  FormulaireView.vue
```

La page `DetailsView.vue` est **réutilisée** pour chaque pays — seul le paramètre `:code` change.

## À retenir

| Élément | Rôle |
| --- | --- |
| `src/router/index.js` | Déclarer toutes les routes |
| `<RouterView>` | Afficher la page active |
| `<RouterLink to="...">` | Lien de navigation sans rechargement |
| `useRoute()` | Lire les paramètres de la route courante |
| `useRouter()` | Naviguer par le code (`router.push(...)`) |
| `:code` dans le path | Paramètre dynamique dans l'URL |

---

*Prochaine fiche → **Fiche 06 : Appels API***
