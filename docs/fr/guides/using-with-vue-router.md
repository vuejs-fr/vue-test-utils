# Utilisation avec Vue Router

## Installer Vue Router pour nos tests

Vous ne devez jamais installer Vue Router sur le constructeur de base de Vue pour vos tests. Installer Vue Router de cette manière ajoute `$route` et `$router` en tant que propriétés en lecture seule sur le prototype Vue.

Pour éviter cela, on peut utiliser une `localVue`, et installer Vue Router dessus.

```js
import { shallow, createLocalVue } from '@vue/test-utils'
import VueRouter from 'vue-router'

const localVue = createLocalVue()
localVue.use(VueRouter)
const router = new VueRouter()

shallow(Component, {
  localVue,
  router
})
```

> **Note :** installer Vue Router sur une `localVue` ajoute également `$route` et `$router` comme des propriétés en lectures seules à `localVue`. Cela signifie que vous ne pouvez pas utiliser l'option `mocks` pour réécrire `$route` et `$router` quand un composant est monté en utilisant une`localVue` avec Vue Router d'installé.

## Tester des composants qui utilisent `router-link` ou `router-view`

Quand vous installez Vue Router, les composants `router-link` et `router-view` sont enregistrés. Cela veut dire que l'on peut les utiliser n'importe où dans notre application sans avoir besoin de les importer.

Quand nous lançons des test, on doit donc rendre ces composants Vue Router disponibles au composant que nous testons. Il y a deux méthodes pour cela.

### Utiliser des stubs

```js
import { shallow } from '@vue/test-utils'

shallow(Component, {
  stubs: ['router-link', 'router-view']
})
```

### Installer Vue Router et `localVue`

```js
import { shallow, createLocalVue } from '@vue/test-utils'
import VueRouter from 'vue-router'

const localVue = createLocalVue()
localVue.use(VueRouter)

shallow(Component, {
  localVue
})
```

## Simuler `$route` et `$router`

Quelques fois, vous souhaitez tester qu'un composant réagisse correctement avec les paramètres des objets `$route` et `$router`. Pour cela, vous pouvez passer des imitations à l'instance de Vue.

```js
import { shallow } from '@vue/test-utils'

const $route = {
  path: '/un/super/chemin'
}

const wrapper = shallow(Component, {
  mocks: {
    $route
  }
})

wrapper.vm.$route.path // `'/un/super/chemin'`
```

## Trucs et astuces

Installer Vue Router ajoute `$route` et `$router` en tant que propriétés en lecture seule au prototype de Vue.

Cela veut dire que n'importe quel futur test qui va essayer de modifier `$route` ou `$router` va échouer.

Pour éviter cela, n'installez jamais Vue Router globalement quand vous lancez vos tests ; utilisez une `localVue` comme expliqué plus haut.
