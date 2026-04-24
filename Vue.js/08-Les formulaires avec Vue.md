# Fiche 08 — Les formulaires avec Vue

> **Application de référence** : le formulaire d'abonnement de l'application pays (`AbonnementView.vue`).

## 1. La liaison avec `v-model`

En HTML classique, il faut écouter les événements du champ ET mettre à jour une variable manuellement. Vue simplifie tout ça avec `v-model`, qui crée une **liaison bidirectionnelle** : quand l'utilisateur tape, la variable se met à jour; quand la variable change, le champ se met à jour.

```vue
<template>
  <input v-model="prenom" type="text" placeholder="Votre prénom" />
  <p>Bonjour, {{ prenom }}!</p>
</template>

<script setup>
import { ref } from 'vue'
const prenom = ref('')
</script>
```

`v-model` fonctionne sur tous les types de champs, mais la variable liée change de type selon le champ :

| Champ HTML | Type de variable | Exemple |
| --- | --- | --- |
| `<input type="text">` | `String` | `ref('')` |
| `<input type="email">` | `String` | `ref('')` |
| `<input type="number">` | `Number` | `ref(0)` |
| `<input type="checkbox">` | `Boolean` | `ref(false)` |
| `<input type="checkbox">` (plusieurs) | `Array` | `ref([])` |
| `<input type="radio">` | `String` | `ref('')` |
| `<select>` | `String` | `ref('')` |
| `<textarea>` | `String` | `ref('')` |

## 2. Regrouper les champs avec `reactive()`

Quand un formulaire a plusieurs champs, on les regroupe dans un seul objet `reactive()` plutôt que de déclarer un `ref()` par champ. C'est plus propre et facile à réinitialiser.

```vue
<script setup>
import { reactive } from 'vue'

const formulaire = reactive({
  prenom: '',
  courriel: '',
  consentement: false
})
</script>

<template>
  <input v-model="formulaire.prenom"      type="text"     placeholder="Prénom" />
  <input v-model="formulaire.courriel"    type="email"    placeholder="Courriel" />
  <input v-model="formulaire.consentement" type="checkbox" />
</template>
```

> **Rappel** : avec `reactive()`, pas de `.value` — on accède directement aux propriétés (`formulaire.prenom`, pas `formulaire.prenom.value`).

## 3. Intercepter la soumission avec `@submit.prevent`

En HTML natif, soumettre un formulaire recharge la page — ce qu'on ne veut jamais dans une SPA Vue. Le modificateur **`.prevent`** bloque ce comportement automatiquement.

```vue
<template>
  <form @submit.prevent="soumettre">
    <input v-model="formulaire.prenom" type="text" />
    <button type="submit">Envoyer</button>
  </form>
</template>

<script setup>
import { reactive } from 'vue'

const formulaire = reactive({ prenom: '' })

function soumettre() {
  // La page ne se recharge PAS grâce à .prevent
  console.log('Formulaire soumis :', formulaire.prenom)
}
</script>
```

> **`.prevent`** est un *modificateur d'événement*. Il est équivalent à appeler `event.preventDefault()` manuellement, mais en une seule ligne directement dans le template.

## 4. Valider les données

La validation se fait dans la fonction de soumission, **avant** de traiter les données. Le principe est simple :

1. Déclarer un objet `erreurs` avec les mêmes champs que le formulaire
2. Dans la fonction `valider()`, vérifier chaque champ et remplir les messages d'erreur
3. Retourner `true` si tout est valide, `false` sinon
4. Afficher les messages d'erreur dans le template avec `v-if`

```vue
<template>
  <form @submit.prevent="soumettre">

    <div>
      <label for="prenom">Prénom *</label>
      <input
        id="prenom"
        v-model="formulaire.prenom"
        type="text"
        :class="{ 'champ-invalide': erreurs.prenom }"
      />
      <!-- Message d'erreur — visible seulement si erreurs.prenom est non vide -->
      <p v-if="erreurs.prenom" class="msg-erreur">{{ erreurs.prenom }}</p>
    </div>

    <div>
      <label for="courriel">Courriel *</label>
      <input
        id="courriel"
        v-model="formulaire.courriel"
        type="email"
        :class="{ 'champ-invalide': erreurs.courriel }"
      />
      <p v-if="erreurs.courriel" class="msg-erreur">{{ erreurs.courriel }}</p>
    </div>

    <button type="submit">S'abonner</button>

  </form>
</template>

<script setup>
import { reactive } from 'vue'

const formulaire = reactive({
  prenom: '',
  courriel: ''
})

// Un objet miroir pour les messages d'erreur
const erreurs = reactive({
  prenom: '',
  courriel: ''
})

function valider() {
  // Réinitialiser les erreurs à chaque tentative
  erreurs.prenom = ''
  erreurs.courriel = ''

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

  return valide
}

function soumettre() {
  if (!valider()) return  // on s'arrête si le formulaire est invalide
  console.log('Données valides :', formulaire)
  // ... traitement (appel API, etc.)
}
</script>

<style scoped>
.champ-invalide { border-color: red; }
.msg-erreur { color: red; font-size: 0.8rem; margin-top: 0.25rem; }
</style>
```

## 5. Afficher un message de succès

Après la soumission, on affiche un message de confirmation en remplaçant le formulaire avec `v-if` / `v-else`.

```vue
<template>

  <!-- Message de succès -->
  <div v-if="soumis">
    <p>✅ Merci {{ formulaire.prenom }}, votre abonnement est confirmé!</p>
    <button @click="reinitialiser">← Nouvel abonnement</button>
  </div>

  <!-- Formulaire -->
  <form v-else @submit.prevent="soumettre">
    <!-- ... champs ... -->
    <button type="submit">S'abonner</button>
  </form>

</template>

<script setup>
import { ref, reactive } from 'vue'

const soumis = ref(false)
const formulaire = reactive({ prenom: '', courriel: '' })
const erreurs = reactive({ prenom: '', courriel: '' })

function soumettre() {
  if (!valider()) return
  soumis.value = true
}

function reinitialiser() {
  formulaire.prenom = ''
  formulaire.courriel = ''
  soumis.value = false
}

function valider() { /* ... voir section 4 ... */ }
</script>
```

## 6. Exemple complet — formulaire d'abonnement

Voici le formulaire de l'application pays simplifié, avec tous les éléments ci-dessus assemblés :

```vue
<!-- src/views/AbonnementView.vue -->
<template>
  <div>
    <h1>Abonnement</h1>

    <!-- Succès -->
    <div v-if="soumis">
      <p>🎉 Merci {{ formulaire.prenom }}! Vous êtes abonné à {{ formulaire.courriel }}.</p>
      <button @click="reinitialiser">← Nouvel abonnement</button>
    </div>

    <!-- Formulaire -->
    <form v-else @submit.prevent="soumettre">

      <!-- Prénom -->
      <div>
        <label for="prenom">Prénom *</label>
        <input id="prenom" v-model="formulaire.prenom" type="text" placeholder="Marie" />
        <p v-if="erreurs.prenom" class="msg-erreur">{{ erreurs.prenom }}</p>
      </div>

      <!-- Courriel -->
      <div>
        <label for="courriel">Courriel *</label>
        <input id="courriel" v-model="formulaire.courriel" type="email" placeholder="marie@exemple.com" />
        <p v-if="erreurs.courriel" class="msg-erreur">{{ erreurs.courriel }}</p>
      </div>

      <!-- Fréquence (radio) -->
      <div>
        <p>Fréquence *</p>
        <label>
          <input v-model="formulaire.frequence" type="radio" value="hebdomadaire" /> Hebdomadaire
        </label>
        <label>
          <input v-model="formulaire.frequence" type="radio" value="mensuel" /> Mensuel
        </label>
        <p v-if="erreurs.frequence" class="msg-erreur">{{ erreurs.frequence }}</p>
      </div>

      <!-- Consentement (checkbox) -->
      <div>
        <label>
          <input v-model="formulaire.consentement" type="checkbox" />
          J'accepte de recevoir des courriels. *
        </label>
        <p v-if="erreurs.consentement" class="msg-erreur">{{ erreurs.consentement }}</p>
      </div>

      <button type="submit">S'abonner</button>

    </form>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'

const soumis = ref(false)

const formulaire = reactive({
  prenom: '',
  courriel: '',
  frequence: '',
  consentement: false
})

const erreurs = reactive({
  prenom: '',
  courriel: '',
  frequence: '',
  consentement: ''
})

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

function soumettre() {
  if (!valider()) return
  soumis.value = true
}

function reinitialiser() {
  formulaire.prenom = ''
  formulaire.courriel = ''
  formulaire.frequence = ''
  formulaire.consentement = false
  soumis.value = false
}
</script>

<style scoped>
.msg-erreur { color: red; font-size: 0.8rem; margin-top: 0.25rem; }
</style>
```

---

## À retenir

- `v-model` lie un champ de formulaire à une variable réactive — la mise à jour est automatique dans les deux sens
- `reactive()` regroupe tous les champs du formulaire dans un seul objet
- `@submit.prevent` intercepte la soumission sans recharger la page
- La validation se fait dans une fonction `valider()` qui remplit un objet `erreurs` et retourne `true` ou `false`
- On affiche le succès avec `v-if="soumis"` / `v-else` pour basculer entre le formulaire et le message de confirmation

---

*Ces 8 fiches couvrent l'ensemble des notions nécessaires pour réaliser le projet. Bon travail!*
