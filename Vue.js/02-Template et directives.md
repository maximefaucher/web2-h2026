# Fiche 02 — Template et directives

> **Application de référence** : afficher la liste des pays et réagir aux actions de l'utilisateur dans l'interface.

## 1. Le template Vue

Le bloc `<template>` contient le HTML de votre composant. Vue y ajoute une syntaxe spéciale pour afficher des données et réagir aux événements.

### Interpolation — afficher une variable

```vue
<template>
  <p>{{ nomPays }}</p>
</template>

<script setup>
const nomPays = 'Canada'
</script>
```

Les doubles accolades `{{ }}` affichent la valeur d'une variable. On appelle ça l'**interpolation**.

## 2. Les directives essentielles

Les directives sont des attributs HTML spéciaux de Vue qui commencent par `v-`.

### `v-bind` — lier un attribut HTML à une variable

```vue
<template>
  <!-- Forme complète -->
  <img v-bind:src="drapeauUrl" v-bind:alt="nomPays" />

  <!-- Raccourci avec : -->
  <img :src="drapeauUrl" :alt="nomPays" />
</template>

<script setup>
const drapeauUrl = 'https://flagcdn.com/ca.svg'
const nomPays = 'Canada'
</script>
```

> On utilise presque toujours le raccourci **`:`** plutôt que `v-bind:`.

### `v-on` — écouter un événement

```vue
<template>
  <!-- Forme complète -->
  <button v-on:click="afficherDetails">Voir détails</button>

  <!-- Raccourci avec @ -->
  <button @click="afficherDetails">Voir détails</button>
</template>

<script setup>
function afficherDetails() {
  console.log('Bouton cliqué!')
}
</script>
```

> On utilise presque toujours le raccourci **`@`** plutôt que `v-on:`.

### `v-if` / `v-else` — affichage conditionnel

```vue
<template>
  <p v-if="isLoading">Chargement en cours…</p>
  <p v-else-if="erreur">Une erreur est survenue.</p>
  <ul v-else>
    <li>Liste des pays ici</li>
  </ul>
</template>

<script setup>
const isLoading = true
const erreur = false
</script>
```

> `v-if` **retire** l'élément du DOM. Pour simplement le **cacher** sans le retirer, utilisez `v-show`.

### `v-for` — boucler sur une liste

```vue
<template>
  <ul>
    <li v-for="pays in listePays" :key="pays.cca3">
      {{ pays.name.common }}
    </li>
  </ul>
</template>

<script setup>
const listePays = [
  { cca3: 'CAN', name: { common: 'Canada' } },
  { cca3: 'FRA', name: { common: 'France' } },
  { cca3: 'JPN', name: { common: 'Japan' } }
]
</script>
```

> **`:key`** est obligatoire avec `v-for`. Il doit être une valeur **unique** pour chaque élément. Il aide Vue à identifier chaque élément pour les mises à jour efficaces.

### `v-model` — liaison bidirectionnelle (formulaires)

`v-model` synchronise la valeur d'un champ de formulaire avec une variable. Quand l'utilisateur tape, la variable se met à jour automatiquement.

```vue
<template>
  <input v-model="recherche" placeholder="Chercher un pays…" />
  <p>Vous cherchez : {{ recherche }}</p>
</template>

<script setup>
import { ref } from 'vue'

const recherche = ref('')
</script>
```

> `v-model` est une combinaison de `:value` et `@input`. On l'utilise sur `<input>`, `<select>` et `<textarea>`.

## 3. Exemple complet — page d'accueil de l'application

```vue
<!-- src/views/AccueilView.vue -->
<template>
  <div>
    <h2>Bienvenue</h2>
    <p>Cette application présente <strong>{{ nombrePays }}</strong> pays.</p>

    <img :src="drapeauExemple" alt="Drapeau exemple" width="80" />

    <button @click="afficherMessage">Cliquez ici</button>
    <p v-if="messageVisible">🎉 Bonjour depuis Vue!</p>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const nombrePays = 195
const drapeauExemple = 'https://flagcdn.com/ca.svg'

const messageVisible = ref(false)

function afficherMessage() {
  messageVisible.value = true
}
</script>
```

> **Remarque** : `ref(false)` crée une variable réactive. On en parle en détail dans la **Fiche 03**.

## À retenir

| Directive | Utilité |
| --- | --- |
| `{{ }}` | Afficher une variable dans le HTML |
| `:attr` (`v-bind`) | Lier un attribut HTML à une variable |
| `@event` (`v-on`) | Écouter un événement (clic, saisie…) |
| `v-if` / `v-else` | Affichage conditionnel |
| `v-for` | Boucler sur une liste |
| `v-model` | Lier un champ de formulaire à une variable |

---

*Prochaine fiche → **Fiche 03 : Réactivité et état***
