# Procédure — Ajout d'une page formulaire d'abonnement

> **Référence** : Fiche 08 - Les formulaires avec Vue

**Application de départ** : Télécharger et désarchiver `liste-de-taches_EXERCICE_4.zip`, puis exécuter ces commandes à la racine :

```bash
npm install
npm run dev
```

---

## Étape 1 — Créer `src/views/AbonnementView.vue`

Créer le fichier `src/views/AbonnementView.vue` et y intégrer les éléments suivants.

### 1.1 — Script (`<script setup>`)

Importer `ref` et `reactive` depuis Vue :

```js
import { ref, reactive } from 'vue'
```

Déclarer les trois objets réactifs :

```js
// État de soumission
const soumis = ref(false)

// Données du formulaire (Fiche 08, section 2)
const formulaire = reactive({
  prenom: '',
  courriel: '',
  frequence: '',
  consentement: false
})

// Objet miroir pour les messages d'erreur (Fiche 08, section 4)
const erreurs = reactive({
  prenom: '',
  courriel: '',
  frequence: '',
  consentement: ''
})
```

Déclarer les trois fonctions :

```js
// Valide chaque champ et retourne true si tout est valide (Fiche 08, section 4)
function valider() {
  erreurs.prenom = ''
  erreurs.courriel = ''
  erreurs.frequence = ''
  erreurs.consentement = ''

  let valide = true

  if (!formulaire.prenom.trim()) {
    erreurs.prenom = 'Le prénom est requis.'
    valide = false
  }

  const regexCourriel = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  if (!formulaire.courriel.trim()) {
    erreurs.courriel = 'Le courriel est requis.'
    valide = false
  } else if (!regexCourriel.test(formulaire.courriel)) {
    erreurs.courriel = 'Adresse courriel invalide.'
    valide = false
  }

  if (!formulaire.frequence) {
    erreurs.frequence = 'Choisissez une fréquence.'
    valide = false
  }

  if (!formulaire.consentement) {
    erreurs.consentement = 'Vous devez accepter pour continuer.'
    valide = false
  }

  return valide
}

// Soumet le formulaire si valide (Fiche 08, section 5)
function soumettre() {
  if (!valider()) return
  soumis.value = true
}

// Réinitialise tous les champs et retourne au formulaire (Fiche 08, section 5)
function reinitialiser() {
  formulaire.prenom = ''
  formulaire.courriel = ''
  formulaire.frequence = ''
  formulaire.consentement = false
  soumis.value = false
}
```

### 1.2 — Template (`<template>`)

Utiliser `v-if="soumis"` / `v-else` pour alterner entre le message de succès et le formulaire (Fiche 08, section 5).

Le formulaire utilise `@submit.prevent="soumettre"` pour bloquer le rechargement de page (Fiche 08, section 3).

Chaque champ est lié avec `v-model` et son message d'erreur est affiché conditionnellement avec `v-if` (Fiche 08, sections 1 et 4).

```html
<template>
  <main class="max-w-lg mx-auto px-4 py-10">
    <h1 class="text-2xl font-bold text-gray-800 mb-6">Abonnement</h1>

    <!-- Message de succès (Fiche 08, section 5) -->
    <div v-if="soumis" class="bg-green-50 border border-green-200 rounded-lg p-6 text-center">
      <p class="text-green-700 text-lg font-medium mb-4">
        🎉 Merci {{ formulaire.prenom }}! Vous êtes abonné à {{ formulaire.courriel }}.
      </p>
      <button
        @click="reinitialiser"
        class="text-sm text-blue-500 hover:underline"
      >
        ← Nouvel abonnement
      </button>
    </div>

    <!-- Formulaire (Fiche 08, section 3) -->
    <form v-else @submit.prevent="soumettre" class="flex flex-col gap-5">

      <!-- Prénom (Fiche 08, sections 1 et 4) -->
      <div>
        <label for="prenom" class="block text-sm font-medium text-gray-700 mb-1">
          Prénom *
        </label>
        <input
          id="prenom"
          v-model="formulaire.prenom"
          type="text"
          placeholder="Marie"
          class="w-full border rounded-lg px-3 py-2 text-sm outline-none focus:ring-2 focus:ring-blue-200"
          :class="{ 'border-red-400': erreurs.prenom, 'border-gray-200': !erreurs.prenom }"
        />
        <p v-if="erreurs.prenom" class="msg-erreur">{{ erreurs.prenom }}</p>
      </div>

      <!-- Courriel (Fiche 08, sections 1 et 4) -->
      <div>
        <label for="courriel" class="block text-sm font-medium text-gray-700 mb-1">
          Courriel *
        </label>
        <input
          id="courriel"
          v-model="formulaire.courriel"
          type="email"
          placeholder="marie@exemple.com"
          class="w-full border rounded-lg px-3 py-2 text-sm outline-none focus:ring-2 focus:ring-blue-200"
          :class="{ 'border-red-400': erreurs.courriel, 'border-gray-200': !erreurs.courriel }"
        />
        <p v-if="erreurs.courriel" class="msg-erreur">{{ erreurs.courriel }}</p>
      </div>

      <!-- Fréquence — boutons radio (Fiche 08, section 1, tableau) -->
      <div>
        <p class="text-sm font-medium text-gray-700 mb-2">Fréquence *</p>
        <div class="flex gap-6">
          <label class="flex items-center gap-2 text-sm text-gray-600 cursor-pointer">
            <input v-model="formulaire.frequence" type="radio" value="hebdomadaire" />
            Hebdomadaire
          </label>
          <label class="flex items-center gap-2 text-sm text-gray-600 cursor-pointer">
            <input v-model="formulaire.frequence" type="radio" value="mensuel" />
            Mensuel
          </label>
        </div>
        <p v-if="erreurs.frequence" class="msg-erreur">{{ erreurs.frequence }}</p>
      </div>

      <!-- Consentement — case à cocher (Fiche 08, section 1, tableau) -->
      <div>
        <label class="flex items-start gap-2 text-sm text-gray-600 cursor-pointer">
          <input
            v-model="formulaire.consentement"
            type="checkbox"
            class="mt-0.5"
          />
          J'accepte de recevoir des courriels. *
        </label>
        <p v-if="erreurs.consentement" class="msg-erreur">{{ erreurs.consentement }}</p>
      </div>

      <!-- Bouton de soumission -->
      <button
        type="submit"
        class="bg-blue-500 hover:bg-blue-600 text-white text-sm px-6 py-2 rounded-lg self-start"
      >
        S'abonner
      </button>

    </form>
  </main>
</template>
```

### 1.3 — Style (`<style scoped>`)

```css
<style scoped>
.msg-erreur {
  color: red;
  font-size: 0.8rem;
  margin-top: 0.25rem;
}
</style>
```

---

## Étape 2 — Modifier `src/router/index.js`

### 2.1 — Ajouter l'import de la nouvelle vue

En haut du fichier, après les imports existants :

```js
import AbonnementView from '@/views/AbonnementView.vue'
```

### 2.2 — Ajouter la route

Dans le tableau `routes`, après la route `/pays/:nom` :

```js
{ path: '/abonnement', name: 'abonnement', component: AbonnementView },
```

Le fichier `index.js` devrait ressembler à ceci une fois modifié :

```js
import { createRouter, createWebHistory } from 'vue-router'
import AccueilView    from '@/views/AccueilView.vue'
import TachesView     from '@/views/TachesView.vue'
import PaysView       from '@/views/PaysView.vue'
import PaysDetailView from '@/views/PaysDetailView.vue'
import AbonnementView from '@/views/AbonnementView.vue'   // ← ajouté

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    { path: '/',           name: 'accueil',         component: AccueilView },
    { path: '/taches',     name: 'liste-de-taches', component: TachesView },
    { path: '/pays',       name: 'pays-liste',      component: PaysView },
    { path: '/pays/:nom',  name: 'pays-detail',     component: PaysDetailView },
    { path: '/abonnement', name: 'abonnement',      component: AbonnementView }, // ← ajouté
  ],
})

export default router
```

## Étape 3 — Modifier `src/components/HeaderComponent.vue`

Dans le `<nav>`, après le `<RouterLink>` vers `/pays`, ajouter :

```html
<RouterLink
  to="/abonnement"
  class="text-gray-500 hover:text-blue-500 transition-colors"
  active-class="text-blue-500 font-medium"
>
  Abonnement
</RouterLink>
```

## Vérification

Démarrer l'application avec `npm run dev` et valider les points suivants :

- [ ] Le lien **Abonnement** apparaît dans la barre de navigation
- [ ] La page s'affiche correctement à l'URL `/abonnement`
- [ ] Soumettre le formulaire **vide** affiche tous les messages d'erreur
- [ ] Entrer un courriel invalide affiche le message d'erreur approprié
- [ ] Un formulaire entièrement valide affiche le message de succès avec le prénom et le courriel
- [ ] Le bouton « Nouvel abonnement » réinitialise le formulaire correctement

---

## Structure finale du projet

**Fichiers modifiés :** `src/router/index.js`, `src/components/HeaderComponent.vue`  
**Fichier ajouté :** `src/views/AbonnementView.vue`  
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
│   │   ├── HeaderComponent.vue  ← modifié
│   │   ├── TodoForm.vue
│   │   ├── TodoItem.vue
│   │   └── TodoList.vue
│   ├── router/
│   │   └── index.js             ← modifié
│   ├── services/
│   │   └── paysService.js
│   ├── stores/
│   │   ├── paysStore.js
│   │   └── todoStore.js
│   ├── views/
│   │   ├── AbonnementView.vue   ← nouveau
│   │   ├── AccueilView.vue
│   │   ├── PaysDetailView.vue
│   │   ├── PaysView.vue
│   │   └── TachesView.vue
│   ├── App.vue
│   └── main.js
├── index.html
├── jsconfig.json
├── package.json
└── vite.config.js
```

---

**Vous voilà fin(e) prêt(e) pour compléter votre application Vue.js!**

*Bon travail!*
