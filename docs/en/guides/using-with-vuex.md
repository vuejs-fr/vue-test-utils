# Utiliser avec Vuex

Dans ce guide, nous allons voir comment tester Vuex dans des composants grâce à Vue Test Utils et comment faire pour tester un store Vuex.

## Tester Vuex dans des composants

### Simuler des actions

Regardons un peu de code !

Ci-dessous, le composant que nous voulons tester. Il fait appel à des actions Vuex.

``` html
<template>
  <div class="text-align-center">
    <input type="text" @input="actionInputIfTrue" />
    <button @click="actionClick()">Click</button>
  </div>
</template>

<script>
import { mapActions } from 'vuex'

export default{
  methods: {
    ...mapActions([
      'actionClick'
    ]),
    actionInputIfTrue: function actionInputIfTrue (event) {
      const inputValue = event.target.value
      if (inputValue === 'input') {
        this.$store.dispatch('actionInput', { inputValue })
      }
    }
  }
}
</script>
```

Pour les objectifs de ce test, on se fiche de ce que les actions font, ou à ce quoi le store ressemble. On doit juste savoir si ces actions sont lancées lorsqu'elles sont supposées l'être, et ce, avec les valeurs attendues.

Pour tester cela, on doit passer un store fictif à Vue lorsque l'on isole notre composant.

Au lieu de passer le store au constructeur de base de Vue, on peut le passer à - [`localVue`](../api/options.md#localvue). Un `localVue` est un constructeur à portée limitée de Vue sur lequel on peut effectuer des changements sans avoir à affecter le constructeur global.

Voyons à quoi cela ressemble :

``` js
import { shallow, createLocalVue } from '@vue/test-utils'
import Vuex from 'vuex'
import Actions from '../../../src/components/Actions'

const localVue = createLocalVue()

localVue.use(Vuex)

describe('Actions.vue', () => {
  let actions
  let store

  beforeEach(() => {
    actions = {
      actionClick: jest.fn(),
      actionInput: jest.fn()
    }
    store = new Vuex.Store({
      state: {},
      actions
    })
  })

  it("appelle l'action `actionInput` du store quand la valeur du champ est `input` et qu'un évènement `input` est lancé", () => {
    const wrapper = shallow(Actions, { store, localVue })
    const input = wrapper.find('input')
    input.element.value = 'input'
    input.trigger('input')
    expect(actions.actionInput).toHaveBeenCalled()
  })

  it("n'appelle pas l'action `actionInput` du store quand la valeur du champ n'est pas `input` et qu'un évènement `input` est lancé", () => {
    const wrapper = shallow(Actions, { store, localVue })
    const input = wrapper.find('input')
    input.element.value = 'not input'
    input.trigger('input')
    expect(actions.actionInput).not.toHaveBeenCalled()
  })

  it("appelle l'action `actionClick` quand le bouton est cliqué", () => {
    const wrapper = shallow(Actions, { store, localVue })
    wrapper.find('button').trigger('click')
    expect(actions.actionClick).toHaveBeenCalled()
  })
})
```

Que se passe-t-il ici ? Premièrement, on indique à Vue d'utiliser Vuex avec la méthode `localVue.use`. C'est tout simplement une surcouche de `Vue.use`.

On va ensuite créer un store fictif en appelant `new Vuex.Store` avec nos propres valeurs. À noter que l'on indique uniquement nos actions, car on ne s'intéresse qu'à elles.

Les actions sont des [fonctions de simulations de Jest](https://facebook.github.io/jest/docs/en/mock-functions.html). Ces fonctions nous donnent accès à des méthodes afin de réaliser des assertions si l'action a été appelée ou non.

On peut ensuite s'assurer dans nos tests que les actions ont été appelées au bon moment.

La manière dont on définit le store peut vous paraitre un peu étrange.

On utilise `beforeEach` pour s'assurer que nous avons un store propre avant chaque test. `beforeEach` est un hook de Mocha qui est appelé avant chaque test. Dans nos tests, on réassigne des valeurs aux variables du store. Si on ne le fait pas, les fonctions de simulations auraient besoin d'être automatiquement réinitialisées. Cela nous laisse la possibilité de changer l'état dans nos tests, sans avoir à affecter les prochains.

La chose la plus importante à noter dans ce test est que **l'on crée une simulation d'un store Vuex, qui est ensuite passé à Vue Test Utils**.

Génial, on peut désormais simuler des actions. Allons avoir comment simuler des accesseurs !

### Simuler des accesseurs

``` html
<template>
  <div>
    <p v-if="inputValue">{{inputValue}}</p>
    <p v-if="clicks">{{clicks}}</p>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'

export default{
  computed: mapGetters([
    'clicks',
    'inputValue'
  ])
}
</script>
```

C'est un composant relativement simple. Il affiche le résultat des accesseurs `clicks` et `inputValue`. Encore une fois, on se fiche de savoir ce que ces accesseurs retournent. On souhaite juste savoir si les résultats sont affichés correctement.

Jetons un œil à un test :

``` js
import { shallow, createLocalVue } from '@vue/test-utils'
import Vuex from 'vuex'
import Actions from '../../../src/components/Getters'

const localVue = createLocalVue()

localVue.use(Vuex)

describe('Getters.vue', () => {
  let getters
  let store

  beforeEach(() => {
    getters = {
      clicks: () => 2,
      inputValue: () => 'input'
    }

    store = new Vuex.Store({
      getters
    })
  })

  it('affiche `state.inputValue` dans la première balise <p>', () => {
    const wrapper = shallow(Actions, { store, localVue })
    const p = wrapper.find('p')
    expect(p.text()).toBe(getters.inputValue())
  })

  it('affiche `stat.clicks` dans la seconde balise <p>', () => {
    const wrapper = shallow(Actions, { store, localVue })
    const p = wrapper.findAll('p').at(1)
    expect(p.text()).toBe(getters.clicks().toString())
  })
})
```

Ce test est similaire à notre test sur les actions. On créer un store fictif avant chaque test, on le passe ensuite comme une option lorsque l'on appelle `shallow`. Pour finir, on asserte que la valeur retournée par nos accesseurs fictifs est bien affichée.

C'est génial, mais comment faisons-nous pour vérifier que nos accesseurs retournent correctement les parties de l'état ?

### Simulation avec des modules

Les [modules](https://vuex.vuejs.org/en/modules.html) sont utiles pour séparer un store en plusieurs morceaux. Ils exportent des accesseurs que l'on peut utiliser dans nos tests.

Jetons un œil à ce composant :

``` html
<template>
  <div>
    <button @click="moduleActionClick()">Cliquer</button>
    <p>{{moduleClicks}}</p>
  </div>
</template>

<script>
import { mapActions, mapGetters } from 'vuex'

export default{
  methods: {
    ...mapActions([
      'moduleActionClick'
    ])
  },

  computed: mapGetters([
    'moduleClicks'
  ])
}
</script>
```

Simple composant qui possède une action et un accesseur.

Et le test :

``` js
import { shallow, createLocalVue } from '@vue/test-utils'
import Vuex from 'vuex'
import Modules from '../../../src/components/Modules'
import module from '../../../src/store/module'

const localVue = createLocalVue()

localVue.use(Vuex)

describe('Modules.vue', () => {
  let actions
  let state
  let store

  beforeEach(() => {
    state = {
      module: {
        clicks: 2
      }
    }

    actions = {
      moduleActionClick: jest.fn()
    }

    store = new Vuex.Store({
      state,
      actions,
      getters: module.getters
    })
  })

  it("appelle l'action du store moduleActionClick quand le bouton est cliqué", () => {
    const wrapper = shallow(Modules, { store, localVue })
    const button = wrapper.find('button')
    button.trigger('click')
    expect(actions.moduleActionClick).toHaveBeenCalled()
  })

  it('affiche `state.inputValue` dans la première balise <p>', () => {
    const wrapper = shallow(Modules, { store, localVue })
    const p = wrapper.find('p')
    expect(p.text()).toBe(state.module.clicks.toString())
  })
})
```

## Testing a Vuex Store (EN)

There are two approaches to testing a Vuex store. The first approach is to unit test the getters, mutations, and actions separately. The second approach is to create a store and test against that. We'll look at both approaches.

To see how to test a Vuex store, we're going to create a simple counter store. The store will have an `increment` mutation and a `counter` getter.

```js
// mutations.js
export default {
  increment (state) {
    state.count++
  }
}
```

```js
// getters.js
export default {
  evenOrOdd: state => state.count % 2 === 0 ? 'even' : 'odd'
}
```

### Testing getters, mutations, and actions separately (EN)

Getters, mutations, and actions are all JavaScript functions, so we can test them without using Vue Test Utils and Vuex.

The benefit to testing getters, mutations, and actions separately is that your unit tests are detailed. When they fail, you know exactly what is wrong with your code. The downside is that you will need to mock Vuex funtions, like `commit` and `dispatch`. This can lead to a situation where your unit tests pass, but your production code fails because your mocks are incorrect.

We'll create two test files, `mutations.spec.js` and `getters.spec.js`:

First, let's test the `increment` mutations:

```js
// mutations.spec.js

import mutations from './mutations'

test('increment increments state.count by 1', () => {
  const state = {
    count: 0
  }
  mutations.increment(state)
  expect(state.count).toBe(1)
})
```

Now let's test the `evenOrOdd` getter. We can test it by creating a mock `state`, calling the getter with the `state` and checking it returns the correct value.

```js
// getters.spec.js

import getters from './getters'

test('evenOrOdd returns even if state.count is even', () => {
  const state = {
    count: 2
  }
  expect(getters.evenOrOdd(state)).toBe('even')
})

test('evenOrOdd returns odd if state.count is even', () => {
  const state = {
    count: 1
  }
  expect(getters.evenOrOdd(state)).toBe('odd')
})

```

### Testing a running store (EN)

Another approach to testing a Vuex store is to create a running store using the store config.

The benefit of testing creating a running store instance is we don't have to mock any Vuex functions.

The downside is that when a test breaks, it can be difficult to find where the problem is.

Let's write a test. When we create a store, we'll use `localVue` to avoid polluting the Vue base constructor. The test creates a store using the `store-config.js` export:

```js
// store-config.spec.js

import mutations from './mutations'
import getters from './getters'

export default {
  state: {
    count: 0
  },
  mutations,
  getters
}
```

```js
import { createLocalVue } from '@vue/test-utils'
import Vuex from 'vuex'
import storeConfig from './store-config'
import { cloneDeep } from 'lodash'

test('increments count value when increment is commited', () => {
  const localVue = createLocalVue()
  localVue.use(Vuex)
  const store = new Vuex.Store(cloneDeep(storeConfig))
  expect(store.state.count).toBe(0)
  store.commit('increment')
  expect(store.state.count).toBe(1)
})

test('updates evenOrOdd getter when increment is commited', () => {
  const localVue = createLocalVue()
  localVue.use(Vuex)
  const store = new Vuex.Store(cloneDeep(storeConfig))
  expect(store.getters.evenOrOdd).toBe('even')
  store.commit('increment')
  expect(store.getters.evenOrOdd).toBe('odd')
})
```

Notice that we use `cloneDeep` to clone the store config before creating a store with it. This is because Vuex mutates the options object used to create the store. To make sure we have a clean store in each test, we need to clone the `storeConfig` object.

## Ressources

- [Projet exemple pour tester les composants](https://github.com/eddyerburgh/vue-test-utils-vuex-example)
- [Projet exemple pour tester les stores](https://github.com/eddyerburgh/testing-vuex-store-example)
- [`localVue`](../api/options.md#localvue)
- [`createLocalVue`](../api/createLocalVue.md)
