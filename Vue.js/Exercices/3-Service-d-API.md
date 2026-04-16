# Exercice 3 - Ajout d'un service d'API + Section archive / détails

## Structure finale visée

```text
src/
├── services/
│   └── paysService.js       ← nouveau  (couche appels API)
├── views/
│   ├── AccueilView.vue      (existant)
│   ├── TachesView.vue       (existant)
│   ├── PaysView.vue         ← nouveau  (liste des pays)
│   └── PaysDetailView.vue   ← nouveau  (fiche d'un pays)
├── router/
│   └── index.js             (à modifier)
└── components/
    └── HeaderComponent.vue  (à modifier)
```

## Étape 1 — Créer la couche service

Plutôt que d'écrire `fetch()` directement dans les composants, on centralise toute la communication avec l'API dans un fichier dédié. Cette séparation des responsabilités rend le code plus lisible et plus facile à maintenir.

### 1.1 Créer le dossier et le fichier

Dans `src/`, crée le dossier `services`, puis le fichier `paysService.js` à l'intérieur.

### 1.2 Écrire le fichier

Copie le contenu suivant dans ce fichier :

```javascript
// src/services/paysService.js
// Couche service : toute la communication avec l'API est ici.
// Les composants Vue n'appellent jamais fetch() directement.

const BASE_URL = 'https://restcountries.com/v3.1'

// Champs nécessaires pour la liste
const CHAMPS_LISTE = 'name,flags'

// Champs nécessaires pour la fiche détaillée
const CHAMPS_DETAILS = 'name,flags,capital,population,area,currencies,maps'

/**
 * Récupère les pays de la région Amériques.
 * @returns {Promise<Array>} Liste triée alphabétiquement
 */
export async function getPaysParRegion() {
  const response = await fetch(
    `${BASE_URL}/region/americas?fields=${CHAMPS_LISTE}`
  )
  if (!response.ok) throw new Error(`Erreur API : ${response.status}`)
  const data = await response.json()
  return data.sort((a, b) => a.name.common.localeCompare(b.name.common))
}

/**
 * Récupère un pays par son nom commun.
 * @param {string} nom - Nom du pays (ex. "Canada")
 * @returns {Promise<Object>} Données complètes du pays
 */
export async function getPaysParNom(nom) {
  const response = await fetch(
    `${BASE_URL}/name/${encodeURIComponent(nom)}?fullText=true&fields=${CHAMPS_DETAILS}`
  )
  if (!response.ok) throw new Error(`Pays introuvable : ${nom}`)
  const data = await response.json()
  return data[0]
}
```

### Points importants - Étape 1

- Les constantes `BASE_URL` et `CHAMPS_*` sont déclarées en haut : une seule ligne à modifier si l'API change.
- Le paramètre `fields=` demande à l'API de ne retourner que les champs nécessaires, ce qui réduit la taille de la réponse.
- Les fonctions lèvent une exception (`throw new Error`) si la réponse n'est pas OK. Le composant n'a qu'à attraper cette erreur.
- `encodeURIComponent()` gère les noms avec espaces (ex. « Costa Rica » → `Costa%20Rica`) dans l'URL.

## Étape 2 — Créer la vue liste (`PaysView.vue`)

Cette vue appelle `getPaysParRegion()` au chargement, puis affiche une grille de cartes pays cliquables.

### 2.1 Créer le fichier

Crée `src/views/PaysView.vue`.

### 2.2 Écrire le composant

```vue
<script setup>
import { ref, onMounted } from 'vue'
import { getPaysParRegion } from '@/services/paysService'

const pays       = ref([])
const chargement = ref(true)
const erreur     = ref(null)

onMounted(async () => {
  try {
    pays.value = await getPaysParRegion()
  } catch (e) {
    erreur.value = e.message
  } finally {
    chargement.value = false
  }
})
</script>

<template>
  <main class="max-w-4xl mx-auto px-4 py-10">
    <h1 class="text-2xl font-semibold text-gray-800 mb-6">Pays des Amériques</h1>

    <!-- État : chargement -->
    <p v-if="chargement" class="text-gray-400">Chargement...</p>

    <!-- État : erreur -->
    <p v-else-if="erreur" class="text-red-500">{{ erreur }}</p>

    <!-- État : données reçues -->
    <div v-else class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 gap-4">
      <RouterLink
        v-for="p in pays"
        :key="p.name.common"
        :to="`/pays/${encodeURIComponent(p.name.common)}`"
        class="bg-white rounded-xl shadow-sm p-4 flex flex-col items-center
               gap-2 hover:shadow-md transition-shadow"
      >
        <img :src="p.flags.svg" :alt="p.name.common" class="w-16 h-10 object-cover rounded" />
        <span class="text-sm text-center text-gray-700">{{ p.name.common }}</span>
      </RouterLink>
    </div>
  </main>
</template>
```

### Points importants - Étape 2

- Les trois `ref` (`pays`, `chargement`, `erreur`) couvrent les trois états possibles de tout appel asynchrone.
- `onMounted` garantit que l'appel API ne se fait qu'une fois, quand le composant est inséré dans le DOM.
- Le bloc `try / catch / finally` capture l'erreur levée dans le service et désactive toujours l'état de chargement, même en cas d'échec.
- Le `v-if / v-else-if / v-else` assure qu'un seul état s'affiche à la fois.
- `encodeURIComponent` sur le nom dans le `:to` est symétrique avec ce que fait le service pour l'appel détail.

## Étape 3 — Créer la vue détail (`PaysDetailView.vue`)

Cette vue lit le paramètre `:nom` de l'URL, appelle `getPaysParNom()` et affiche la fiche complète du pays.

### 3.1 Créer le fichier

Crée `src/views/PaysDetailView.vue`.

### 3.2 Écrire le composant

```vue
<script setup>
import { ref, onMounted } from 'vue'
import { useRoute } from 'vue-router'
import { getPaysParNom } from '@/services/paysService'

const route      = useRoute()
const pays       = ref(null)
const chargement = ref(true)
const erreur     = ref(null)

onMounted(async () => {
  try {
    const nom = decodeURIComponent(route.params.nom)
    pays.value = await getPaysParNom(nom)
  } catch (e) {
    erreur.value = e.message
  } finally {
    chargement.value = false
  }
})

// Formate un objet currencies : { CAD: { name: 'Canadian dollar', symbol: '$' } }
function formatDevise(currencies) {
  if (!currencies) return 'N/A'
  return Object.values(currencies)
    .map(c => `${c.name} (${c.symbol})`)
    .join(', ')
}
</script>

<template>
  <main class="max-w-2xl mx-auto px-4 py-10">

    <p v-if="chargement" class="text-gray-400">Chargement...</p>
    <p v-else-if="erreur" class="text-red-500">{{ erreur }}</p>

    <div v-else-if="pays">
      <!-- En-tête : drapeau + nom -->
      <div class="flex items-center gap-4 mb-8">
        <img :src="pays.flags.svg" :alt="pays.name.common"
             class="w-24 h-16 object-cover rounded shadow" />
        <h1 class="text-3xl font-semibold text-gray-800">
          {{ pays.name.common }}
        </h1>
      </div>

      <!-- Informations -->
      <dl class="grid grid-cols-2 gap-4 text-sm">
        <div>
          <dt class="text-gray-400 font-medium">Capitale</dt>
          <dd class="text-gray-800">{{ pays.capital?.[0] ?? 'N/A' }}</dd>
        </div>
        <div>
          <dt class="text-gray-400 font-medium">Population</dt>
          <dd class="text-gray-800">{{ pays.population.toLocaleString('fr-CA') }}</dd>
        </div>
        <div>
          <dt class="text-gray-400 font-medium">Superficie</dt>
          <dd class="text-gray-800">{{ pays.area.toLocaleString('fr-CA') }} km²</dd>
        </div>
        <div>
          <dt class="text-gray-400 font-medium">Devise</dt>
          <dd class="text-gray-800">{{ formatDevise(pays.currencies) }}</dd>
        </div>
      </dl>

      <!-- Lien Google Maps -->
      <a
        :href="pays.maps.googleMaps"
        target="_blank"
        rel="noopener"
        class="mt-8 inline-block bg-blue-500 hover:bg-blue-600
               text-white text-sm font-medium px-5 py-2 rounded-lg transition-colors"
      >
        Voir sur Google Maps →
      </a>
    </div>
    
    <!-- Retour -->
    <div class="mt-4">
      <RouterLink to="/pays"
        class="text-sm text-gray-400 hover:text-blue-500 transition-colors"
      >
        ← Retour à la liste
      </RouterLink>
    </div>

  </main>
</template>
```

### Points importants - Étape 3

- `useRoute()` donne accès aux paramètres de la route active. Ici, `route.params.nom` correspond au `:nom` déclaré dans le routeur.
- `decodeURIComponent` reconvertit `Costa%20Rica` en `"Costa Rica"` avant de passer le nom au service.
- L'objet `currencies` de l'API est un dictionnaire (ex. `{ CAD: { name: ..., symbol: ... } }`) : `Object.values()` permet d'en extraire les valeurs simplement.
- La balise `<dl>` (description list) avec `<dt>` / `<dd>` est sémantiquement adaptée à l'affichage de paires clé / valeur.
- Le champ `maps.googleMaps` est fourni directement par l'API REST Countries : aucune construction d'URL manuelle n'est nécessaire.

## Étape 4 — Déclarer les routes dans `router/index.js`

Vue Router doit connaître les deux nouvelles vues pour pouvoir les afficher. Ouvre `src/router/index.js` et apporte les modifications suivantes.

### 4.1 Ajouter les imports

Ajoute les deux lignes d'import après les imports existants :

```javascript
import PaysView       from '../views/PaysView.vue'
import PaysDetailView from '../views/PaysDetailView.vue'
```

### 4.2 Ajouter les routes

Dans le tableau `routes`, ajoute les deux nouvelles entrées :

```javascript
{ path: '/pays',      name: 'pays-liste',  component: PaysView },
{ path: '/pays/:nom', name: 'pays-detail', component: PaysDetailView },
```

### Fichier complet après modifications

```javascript
import { createRouter, createWebHistory } from 'vue-router'
import AccueilView    from '../views/AccueilView.vue'
import TachesView     from '../views/TachesView.vue'
import PaysView       from '../views/PaysView.vue'
import PaysDetailView from '../views/PaysDetailView.vue'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    { path: '/',          name: 'accueil',         component: AccueilView },
    { path: '/taches',    name: 'liste-de-taches', component: TachesView },
    { path: '/pays',      name: 'pays-liste',      component: PaysView },
    { path: '/pays/:nom', name: 'pays-detail',     component: PaysDetailView, props: true },
    // `props: true` permet de passer les params de la route comme props au composant
  ],
})

export default router
```

### Points importants - Étape 4

- L'ordre des routes `/pays` et `/pays/:nom` est important : la route statique doit précéder la route dynamique pour éviter toute ambiguïté.
- Le segment `:nom` dans le `path` est le paramètre dynamique; il sera accessible dans le composant via `route.params.nom`.
- Le `name` de chaque route est optionnel, mais recommandé : il permet de naviguer par nom plutôt que par chemin.

## Étape 5 — Ajouter le lien de navigation dans `HeaderComponent.vue`

Il reste à ajouter le lien « Pays » dans la barre de navigation. Ouvre `src/components/HeaderComponent.vue` et repère le `<nav>` existant.

### 5.1 Ajouter le RouterLink

Copie le pattern des liens existants et ajoute un troisième lien :

```vue
<RouterLink
  to="/pays"
  class="text-gray-500 hover:text-blue-500 transition-colors"
  active-class="text-blue-500 font-medium"
>
  Pays
</RouterLink>
```

### Bloc `<nav>` complet après modification

```vue
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
  <RouterLink
    to="/pays"
    class="text-gray-500 hover:text-blue-500 transition-colors"
    active-class="text-blue-500 font-medium"
  >
    Pays
  </RouterLink>
</nav>
```

> 💡 La prop `active-class` est gérée automatiquement par Vue Router : le lien « Pays » sera mis en surbrillance dès que la route active commence par `/pays`, ce qui couvre aussi la vue détail.

---

## Vérification

Lance l'application et effectue les vérifications suivantes :

1. La barre de navigation affiche maintenant trois liens : **Accueil**, **Tâches**, **Pays**.
2. La page `/pays` affiche une grille de drapeaux et de noms de pays des Amériques.
3. Un clic sur un pays navigue vers `/pays/Canada` (ou le nom du pays cliqué).
4. La fiche affiche : drapeau, nom, capitale, population, superficie, devise et le bouton Google Maps.
5. Le bouton **← Retour à la liste** ramène bien vers `/pays`.
6. Le lien « Pays » dans la nav devient bleu quand on est sur `/pays` ou `/pays/:nom`.

---

## Étape 6 — **DÉFI** : Ajouter un compteur, un champ de recherche et un bouton *reset*

Pour cette étape, nous vous mettons au défi afin d'ajouter dans la page des pays (`PaysView.vue`) les éléments suivants :

- Un **champ de recherche** afin de **filtrer par nom les pays** affichés dans la page;
- Un **paragraphe indiquant le nombre de pays affichés** dans la page;
- Un **bouton permettant d'effacer le filtre** par nom.

À vous d'ajouter le HTML requis dans le `<template>` et d'ajouter les éléments nécessaire aux nouvelles fonctionnalités dans le `<script setup>`.

### Étapes à suivre : Ajout d'une recherche en temps réel dans `PaysView.vue`

Tous les changements se font uniquement dans ce fichier.

#### Dans le `<script setup>`

- Importe `computed` depuis Vue, en plus de `ref` et `onMounted` qui y sont déjà.
- Ajoute un nouveau `ref` pour stocker la valeur saisie dans le champ de recherche.
- Crée une propriété `computed` qui retourne une version filtrée du tableau `pays` selon la valeur de ce `ref`. Utilise `toLowerCase()` des deux côtés de la comparaison pour rendre la recherche insensible à la casse. *Rappel : ne filtre jamais `pays` directement, sinon les données sont perdues et le filtre devient irréversible.*

#### Dans le `<template>`

- Ajoute un `<input>` lié à ton `ref` de recherche via `v-model`, placé avant la grille.
- Juste à côté, ajoute un bouton « Effacer » qui remet le `ref` à une chaîne vide. Affiche-le uniquement quand le champ n'est pas vide avec `v-if`.
- Ajoute un paragraphe affichant le nombre de pays affichés en utilisant la propriété `.length` de ta `computed`.
- Dans le `v-for` de la grille, itère sur ta `computed` plutôt que sur `pays` directement.
- *Bonus* : affiche un message « Aucun résultat » quand ta `computed` retourne un tableau vide.

---

## Récapitulatif des fichiers

| Fichier | Action | Rôle |
| --- | --- | --- |
| `src/services/paysService.js` | Créer | Constantes API + fonctions `getPaysParRegion` / `getPaysParNom` |
| `src/views/PaysView.vue` | Créer | Vue liste : appel API, grille de cartes, gestion des états |
| `src/views/PaysDetailView.vue` | Créer | Vue détail : lecture du paramètre `:nom`, fiche complète |
| `src/router/index.js` | Modifier | Déclarer les routes `/pays` et `/pays/:nom` |
| `src/components/HeaderComponent.vue` | Modifier | Ajouter le `RouterLink` vers `/pays` dans la nav |
