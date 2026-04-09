# Fiche 04 — Les composants

> **Application de référence** : découper l'application pays en composants réutilisables — carte de pays, entête, pied de page, etc.

## 1. Pourquoi des composants?

Un composant est un **morceau d'interface réutilisable et autonome**. Au lieu d'écrire tout le HTML dans une seule page, on découpe en petites pièces.

**Exemple** : la carte d'un pays peut être utilisée dans la page d'archive ET dans la page de recherche. On l'écrit une seule fois.

```text
src/
|── components/
|   |── HeaderComponent.vue    ← entête commune (toutes les pages)
|   ├── FooterComponent.vue    ← pied de page commun (toutes les pages)
|   └── CartePayComponent.vue  ← réutilisé pour chaque pays (archive)
|── views/
|   └── ArchiveView.vue        ← page des pays
└── App.vue                    ← composant racine de l'application
```

## 2. Créer et utiliser un composant

**Étape 1** — créer le composant dans `src/components/` :

```vue
<!-- src/components/CartePaysComponent.vue -->
<template>
  <div class="carte">
    <img :src="drapeau" :alt="nom" />
    <h3>{{ nom }}</h3>
    <p>Population : {{ population.toLocaleString() }}</p>
  </div>
</template>

<script setup>
// Les props seront définies à l'étape suivante
</script>

<style scoped>
.carte {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 1rem;
  width: 200px;
}
</style>
```

**Étape 2** — l'importer et l'utiliser dans une vue (ou dans un autre composant) :

```vue
<!-- src/views/ArchiveView.vue -->
<template>
  <div class="grille">
    <CartePays v-for="pays in listePays" :key="pays.cca3" />
  </div>
</template>

<script setup>
import CartePays from '@/components/CartePaysComponent.vue'
// ...
</script>
```

> **`@/`** est un raccourci pour `src/`. Il est configuré automatiquement par Vite.

## 3. Les props — passer des données au composant

Les **props** sont les paramètres d'un composant. Le parent lui passe des valeurs, l'enfant les reçoit.

**Dans le composant enfant** — déclarer les props :

```vue
<!-- src/components/CartePaysComponent.vue -->
<template>
  <div class="carte">
    <img :src="drapeau" :alt="nom" />
    <h3>{{ nom }}</h3>
    <p>Population : {{ population.toLocaleString() }}</p>
  </div>
</template>

<script setup>
defineProps({
  nom: String,
  drapeau: String,
  population: Number
})
</script>
```

**Dans le composant parent** — passer les valeurs :

```vue
<!-- src/views/ArchiveView.vue -->
<template>
  <CartePays
    v-for="pays in listePays"
    :key="pays.cca3"
    :nom="pays.name.common"
    :drapeau="pays.flags.svg"
    :population="pays.population"
  />
</template>
```

> **Règle importante** : les données coulent **vers le bas** (parent → enfant via props). Un enfant ne doit jamais modifier directement une prop.

## 4. Les emits — envoyer un événement vers le parent

### Pourquoi les emits?

On vient de voir que les props font descendre les données du parent vers l'enfant. Mais que se passe-t-il quand c'est l'enfant qui doit parler au parent — par exemple, l'utilisateur a cliqué sur une carte de pays?

**Un enfant ne peut pas modifier les props qu'il reçoit**, et il ne devrait pas modifier l'état du parent directement non plus. La solution propre : l'enfant **émet un événement** (comme un signal), et le parent choisit d'y réagir ou non.

C'est exactement le même principe que les événements natifs du navigateur : un `<button>` ne sait pas ce qui se passera quand on le clique — il émet simplement un événement `click`, et c'est le code qui l'écoute qui décide quoi faire.

```text
Parent  ──── props ────▶  Enfant
Parent  ◀─── emits ────   Enfant
```

### `defineEmits` — déclarer les événements de l'enfant

`defineEmits()` remplit deux rôles : il **documente** quels événements le composant peut émettre, et il retourne une **fonction `emit`** qu'on appellera pour déclencher ces événements.

```vue
<script setup>
// On déclare la liste des événements que ce composant peut émettre
const emit = defineEmits(['paysSelectionne'])
//                        ↑
//               nom de l'événement (chaîne de caractères)
</script>
```

On peut déclarer plusieurs événements si nécessaire :

```vue
const emit = defineEmits(['paysSelectionne', 'paysSurvole', 'favoriModifie'])
```

### Émettre un événement avec une valeur

`emit()` prend deux arguments : le **nom de l'événement** et une **valeur optionnelle** à transmettre au parent (appelée le *payload*).

```vue
emit('paysSelectionne', props.cca3)
//    ↑ nom              ↑ valeur transmise au parent
```

Le payload peut être n'importe quelle valeur — une chaîne, un nombre, un objet entier, etc.

### Exemple complet — enfant + parent

**Composant enfant** (`CartePaysComponent.vue`) — il émet quand l'utilisateur clique :

```vue
<!-- src/components/CartePaysComponent.vue -->
<template>
  <div class="carte" @click="selectionner">
    <img :src="drapeau" :alt="nom" />
    <h3>{{ nom }}</h3>
  </div>
</template>

<script setup>
const props = defineProps({
  nom: String,
  drapeau: String,
  cca3: String        // le code unique du pays, ex. "CAN"
})

// 1. On déclare l'événement et on récupère la fonction emit
const emit = defineEmits(['paysSelectionne'])

function selectionner() {
  // 2. On émet l'événement en passant le code du pays comme valeur
  emit('paysSelectionne', props.cca3)
  //    ↑ nom de l'événement    ↑ payload = la valeur reçue par le parent
}
</script>
```

**Composant parent** (`ArchiveView.vue`) — il écoute l'événement avec `@nomEvenement` :

```vue
<!-- src/views/ArchiveView.vue -->
<template>
  <CartePays
    v-for="pays in listePays"
    :key="pays.cca3"
    :nom="pays.name.common"
    :drapeau="pays.flags.svg"
    :cca3="pays.cca3"
    @paysSelectionne="allerVersDetails"
    <!-- ↑ quand l'enfant émet 'paysSelectionne', appeler allerVersDetails -->
  />
</template>

<script setup>
// codePays reçoit automatiquement la valeur émise (le payload)
function allerVersDetails(codePays) {
  console.log('Pays sélectionné :', codePays)  // ex. "CAN"
  // Navigation vers la page détails — voir Fiche 05
}
</script>
```

> **Lecture de `@paysSelectionne="allerVersDetails"`** : "quand cet enfant émet l'événement `paysSelectionne`, appelle la fonction `allerVersDetails` en lui passant la valeur émise."

### Comparaison props / emits en un coup d'œil

| | Props | Emits |
| --- | --- | --- |
| **Direction** | Parent → Enfant | Enfant → Parent |
| **Déclaration** | `defineProps({...})` | `defineEmits([...])` |
| **Utilisation côté enfant** | Lire la valeur reçue | Appeler `emit('nom', valeur)` |
| **Utilisation côté parent** | Passer avec `:maProp="valeur"` | Écouter avec `@monEvent="maFonction"` |
| **Analogie** | Paramètres d'une fonction | Valeur de retour / callback |

## 5. Les slots — composants de mise en page

Un **slot** est un emplacement dans un composant où le parent peut injecter du contenu. C'est très utile pour les composants de mise en page comme une carte générique.

```vue
<!-- src/components/CarteGenerique.vue -->
<template>
  <div class="carte">
    <slot />  <!-- le contenu sera injecté ici -->
  </div>
</template>

<style scoped>
.carte {
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 1.5rem;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}
</style>
```

```vue
<!-- Utilisation dans une vue -->
<template>
  <CarteGenerique>
    <h3>Canada</h3>
    <p>Population : 38 000 000</p>
  </CarteGenerique>
</template>
```

## 6. Lien avec le MVC

| Rôle MVC | Dans Vue |
| --- | --- |
| **View** | Le `<template>` du composant |
| **Controller** | Le `<script setup>` (logique, emits, méthodes) |
| **Model** | Les props, les `ref()` locaux, ou le store Pinia |

Un composant bien conçu : son template est simple, sa logique est dans `<script setup>`, et il communique via **props** (entrée) et **emits** (sortie).

## À retenir

- Un composant = un fichier `.vue` dans `components/`
- **Props** : le parent envoie des données à l'enfant (flux descendant)
- **Emits** : l'enfant signale un événement au parent (flux montant)
- **Slots** : le parent injecte du contenu dans l'enfant
- Un composant réutilisable ne doit pas dépendre du contexte de la page — il reçoit tout via des props

---

*Prochaine fiche → **Fiche 05 : Routage — Vue Router***
