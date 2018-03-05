<<<<<<< HEAD
# Pour traduire la documentation de vue-test-utils

### Workflow de travail

Cette branche de travail `working` est volontairement mise en avant et doit uniquement être mise à jour dans le sens :
=======
# Vue Test Utils [![Build Status](https://circleci.com/gh/vuejs/vue-test-utils/tree/dev.png?style=shield)](https://circleci.com/gh/vuejs/vue-test-utils)

## Currently in beta
To use Vue Test Utils beta:
```
// npm
npm install --save-dev @vue/test-utils

// yarn
yarn add --dev @vue/test-utils
```

## Intro

Vue Test Utils is the official test library for [Vue.js](http://vuejs.org). It provides methods for unit testing Vue components.

## Documentation

Refer to the [documentation](https://vue-test-utils.vuejs.org/)
>>>>>>> upstream/dev

`vuejs/vue-test-utils:dev` --> `vuejs-fr/vue-test-utils:working`.

Nous traduisons les fichiers directement dans le dossier `en` sans les renommer. Cela permet lors de la mise à jour de la documentation via l'utilisation des commandes :

```
git fetch upstream
git merge upstream/dev
```

d'obtenir des conflits **sur les pages déjà traduites** et ainsi maintenir la documentation à jour en fonction des modifications à travers **les documents déjà traduits**.

<<<<<<< HEAD
### Traduction
=======
Please make sure to read the [Contributing Guide](https://github.com/vuejs/vue/blob/dev/.github/CONTRIBUTING.md) before making a pull request.
>>>>>>> upstream/dev

Pour savoir ce qui est [en cours de traduction](https://github.com/vuejs-fr/vue-test-utils/issues/1) ou [comment traduire un fichier](https://github.com/vuejs-fr/vue-test-utils/issues/2), référez vous aux issues correspondantes.

<<<<<<< HEAD
### Reverssement
=======
Changes for each release are documented in the [release notes](https://github.com/vuejs/vue-test-utils/releases).
>>>>>>> upstream/dev

Quand un fichier traduit est validé par pull request, on le met à jour dans le dossier `fr` de `vuejs-fr/vue-test-utils:dev` puis on propose une pull request au site principal :

`vuejs-fr/vue-test-utils:dev` --> `vuejs/vue-test-utils:dev`

ainsi le dossier officiel hébergeant la documentation possède bien le dossier `fr` en français et le dossier `en` en anglais.

Note : il peut être intéressant de faire une pull request par ficher validé et donc de créer une branche dérivée de `vuejs-fr/vue-test-utils:dev` pour faire la pull request (`vuejs-fr/vue-test-utils:dev` --> `vuejs-fr/vue-test-utils:only_one_changed_file_from_master` --> `vuejs/vue-test-utils:dev`)
