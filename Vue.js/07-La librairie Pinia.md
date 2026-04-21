# Fiche 07 — Gestion d'état global avec Pinia

> **Application de référence** : partager les données des pays entre plusieurs pages sans les recharger à chaque navigation.

## 1. Pourquoi un état global?

Chaque composant Vue a son propre état local (`ref`, `reactive`, etc.). Mais que se passe-t-il quand **plusieurs pages ont besoin des mêmes données**?

**Problème sans état global :**

```text
ArchiveView  →  charge 250 pays depuis l'API
  ↓ (l'utilisateur navigue vers les détails d'unpays ou une autre page, puis revient)
ArchiveView  →  charge à nouveau 250 pays depuis l'API  ← inutile!
```

**Solution avec Pinia :**

```text
paysStore  →  charge 250 pays UNE SEULE FOIS
ArchiveView  →  lit les pays depuis le store
DetailsView  →  lit aussi depuis le store
ArchiveView  →  relit les pays depuis le store
...
```

Pinia est le gestionnaire d'état officiel de Vue. Un **store** est un endroit centralisé pour stocker des données partagées entre composants.

## 2. Installation

> [**Site officiel de Pinia**](https://pinia.vuejs.org/)

Si vous avez coché "Pinia" lors de `npm create vue@latest`, c'est déjà configuré. Sinon :

```bash
npm install pinia
```

Dans `main.js`, il faut importer `createPinia` et chaîner son appel dans la méthode `use()` (comme pour `router`) :

```js
// src/main.js
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'
import router from './router'

createApp(App)
  .use(createPinia())
  .use(router)
  .mount('#app')
```

## 3. Créer un store

```js
// src/stores/paysStore.js
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import { getTousLesPays } from '@/services/paysService.js'

export const usePaysStore = defineStore('pays', () => {

  // --- ÉTAT ---
  const listePays = ref([])
  const isLoading = ref(false)
  const erreur = ref(null)
  const recherche = ref('')

  // --- GETTERS (valeurs dérivées) ---
  const paysFiltres = computed(() => {
    if (!recherche.value) return listePays.value
    return listePays.value.filter(pays =>
      pays.name.common.toLowerCase().includes(recherche.value.toLowerCase())
    )
  })

  const nombrePays = computed(() => listePays.value.length)

  // --- ACTIONS ---
  async function chargerPays() {
    // Ne pas recharger si les données sont déjà là
    if (listePays.value.length > 0) return

    isLoading.value = true
    erreur.value = null

    try {
      listePays.value = await getTousLesPays()
    } catch (e) {
      erreur.value = e.message
    } finally {
      isLoading.value = false
    }
  }

  return {
    // État
    listePays,
    isLoading,
    erreur,
    recherche,
    // Getters
    paysFiltres,
    nombrePays,
    // Actions
    chargerPays
  }
})
```

> **Structure** d'un store Pinia :
    > - **État** : les `ref()` / `reactive()` — les données
    > - **Getters** : les `computed()` — valeurs dérivées de l'état
    > - **Actions** : les fonctions — modifier l'état ou appeler une API

## 4. Utiliser le store dans les composants

```vue
<!-- src/views/ArchiveView.vue -->
<template>
  <div>
    <h2>Archive ({{ paysStore.nombrePays }} pays)</h2>

    <input v-model="paysStore.recherche" placeholder="Chercher…" />

    <p v-if="paysStore.isLoading">⏳ Chargement…</p>
    <p v-else-if="paysStore.erreur">{{ paysStore.erreur }}</p>

    <div class="grille" v-else>
      <CartePays
        v-for="pays in paysStore.paysFiltres"
        :key="pays.cca3"
        :nom="pays.name.common"
        :drapeau="pays.flags.svg"
        :population="pays.population"
        :cca3="pays.cca3"
        @paysSelectionne="allerVersDetails"
      />
    </div>
  </div>
</template>

<script setup>
import { onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { usePaysStore } from '@/stores/paysStore.js'
import CartePays from '@/components/CartePaysComponent.vue'

const paysStore = usePaysStore()
const router = useRouter()

onMounted(() => {
  paysStore.chargerPays()  // Ne recharge pas si déjà chargé
})

function allerVersDetails(code) {
  router.push({ name: 'details', params: { code } })
}
</script>
```

## 5. Accéder au store depuis plusieurs pages

La puissance de Pinia : **le même store, depuis n'importe quel composant**.

```vue
<!-- src/views/AccueilView.vue -->
<script setup>
import { onMounted } from 'vue'
import { usePaysStore } from '@/stores/paysStore.js'

const paysStore = usePaysStore()

onMounted(() => {
  paysStore.chargerPays()  // Données déjà en cache si Archive visitée avant
})
</script>

<template>
  <div>
    <h2>Bienvenue</h2>
    <p>Notre base contient {{ paysStore.nombrePays }} pays.</p>
  </div>
</template>
```

Voir également l'utilisation du store Pinia dans [la page des détails d'un pays](https://github.com/maximefaucher/monde-app/blob/main/src/views/DetailsView.vue).

**REMARQUE** : voir les lignes 117 à 126 pour la définition d'un **sous-composant local** et les lignes 67 à 72 pour son utilisation.

## 6. Lien avec le MVC

Avec Pinia, la séparation des responsabilités est complète :

| Couche MVC | Dans notre application |
| --- | --- |
| **Model** | `paysStore.js` (état) + `paysService.js` (appels API) |
| **View** | Le `<template>` de chaque composant |
| **Controller** | Le `<script setup>` de chaque composant (coordonne le store et la vue) |

Le store contient les **données et leur logique de gestion**. Les composants ne font que **lire** le store et **appeler ses actions**.

## 7. Vue d'ensemble de l'architecture finale

```text
src/
├── services/
│   └── paysService.js      ← appels fetch vers l'API (Model)
├── stores/
│   └── paysStore.js        ← état global Pinia (Model)
├── components/
│   ├── CartePaysComponent.vue
│   ├── HeaderComponent.vue
│   └── FooterComponent.vue
├── views/
│   ├── AccueilView.vue     ← lit le store
│   ├── ArchiveView.vue     ← lit et filtre le store
│   ├── DetailsView.vue     ← appel API individuel
│   └── FormulaireView.vue  ← état local (pas besoin du store)
├── router/
│   └── index.js
└── App.vue
```

## À retenir

- Pinia centralise les données **partagées entre plusieurs pages**
- Un store contient : **état** (`ref`), **getters** (`computed`), **actions** (fonctions)
- On utilise un store avec `const store = useMonStore()` dans n'importe quel composant
- Évitez le store pour l'état **local** à un seul composant — utilisez `ref()` directement
- Pinia + Service = séparation propre entre la récupération des données et leur gestion

---

*Vous avez maintenant les 7 fiches pour développer votre application avec Vue.js!*
