# Fiche 01 — Structure et fondements de Vue.js

> **Application de référence** : une petite app qui affiche des informations sur les pays du monde, en utilisant l'API [restcountries.com](https://restcountries.com/v3.1/).

## 1. Qu'est-ce que Vue.js?

Vue.js est un **framework JavaScript progressif** pour construire des interfaces utilisateur. On l'utilise pour créer des applications web interactives organisées en **composants**.

## 2. Les deux styles d'écriture

Vue offre deux façons d'écrire ses composants. Dans ce cours, on utilise la **Composition API avec `<script setup>`**, qui est l'approche moderne et recommandée.

| Style | Description |
| --- | --- |
| **Options API** | Ancienne approche. Code organisé par *options* (`data`, `methods`, `computed`…) |
| **Composition API** | Approche moderne. Logique regroupée par *fonctionnalité* dans `<script setup>` |

## 3. Le fichier `.vue` — Single File Component (SFC)

Chaque page ou composant Vue est écrit dans un fichier `.vue` qui contient trois blocs :

```vue
<template>
  <!-- Le HTML de votre composant -->
  <h1>Bonjour!</h1>
</template>

<script setup>
  // La logique JavaScript
  const message = 'Bonjour!'
</script>

<style scoped>
  /* Le CSS, limité à ce composant grâce à "scoped" */
  h1 {
    color: navy;
  }
</style>
```

> **`scoped`** : l'attribut `scoped` sur `<style>` signifie que le CSS ne s'applique qu'à **ce composant**, sans affecter le reste de l'application.

## 4. Structure d'un projet Vue (avec Vite)

**Vite** (prononcé "veet", mot français signifiant "rapide") est un outil de build moderne créé par Evan You, le même créateur que Vue.js. Vite est aujourd'hui **l'outil recommandé officiellement** par l'équipe Vue pour tous les nouveaux projets. Il remplace Vue CLI (basé sur Webpack) qui est désormais en mode maintenance.

### Créer un projet Vue.js avec Vite

Quand on crée un projet Vue.js avec Vite, il s'agit de taper les commandes suivantes :

```bash
npm create vue@latest
# répondre aux questions interactives du programme : TypeScript ? Router ? Pinia ? Tests ?
cd mon-projet # changer mon-projet pour le nom donné
npm install
npm run dev
```

Nous obtenons ainsi la structure de projet suivante :

```text
mon-projet/
├── public/            ← fichiers statiques (favicon, etc.)
├── src/
│   ├── assets/        ← images, CSS globaux
│   ├── components/    ← composants réutilisables (.vue)
│   ├── views/         ← pages de l'application (.vue)
│   ├── router/
│   │   └── index.js   ← configuration des routes
│   ├── stores/        ← gestion d'état global (Pinia)
│   ├── App.vue        ← composant racine
│   └── main.js        ← point d'entrée de l'application
├── index.html
└── package.json
```

> **`views/` vs `components/`** : par convention, les **pages** (liées à une route) vont dans `views/`, et les **morceaux réutilisables** vont dans `components/`.

## 5. Le cycle de vie d'un composant

Un composant Vue passe par plusieurs étapes depuis sa création jusqu'à sa destruction. Les deux hooks les plus utiles sont :

```vue
<script setup>
import { onMounted, onUnmounted } from 'vue'

onMounted(() => {
  // Exécuté quand le composant est affiché dans la page
  // C'est ici qu'on fait les appels API!
  console.log('Composant prêt.')
})

onUnmounted(() => {
  // Exécuté quand le composant est retiré de la page
  console.log('Composant retiré.')
})
</script>
```

## 6. Dans notre application — `App.vue`

`App.vue` est le composant racine. Il contient l'entête, le pied de page et l'emplacement où les pages s'affichent (via le routeur, vu plus tard).

```vue
<!-- src/App.vue -->
<template>
  <div class="min-h-screen flex flex-col">
    <AppHeader />
    <main class="flex-1">
      <RouterView />
    </main>
    <AppFooter />
  </div>
</template>

<script setup>
import AppHeader from '@/components/AppHeader.vue'
import AppFooter from '@/components/AppFooter.vue'
</script>
```

## À retenir

- Un fichier `.vue` = `<template>` + `<script setup>` + `<style scoped>`
- On utilise la **Composition API** dans ce cours
- Les **pages** vont dans `views/`, les **composants réutilisables** dans `components/`
- `onMounted()` est le bon endroit pour lancer un appel API
- `App.vue` est le composant racine qui englobe toute l'application

---

*Prochaine fiche → **Fiche 02 : Template et directives***
