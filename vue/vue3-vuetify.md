## Guide for project with Vue, Vuetify and Pinia
- [Vue.js 3](https://vuejs.org/guide/introduction.html) as framework
- [Vuetify](https://vuetifyjs.com/en/getting-started/installation/) as UI Component
- [MDI Icons](https://pictogrammers.github.io/@mdi/font/2.0.46/) as Icons
- [Pinia](https://pinia.vuejs.org/introduction.html) as State management
### Create app
```sh
npm init vue@latest
```
```
✔ Project name: … <your-project-name>
✔ Add TypeScript? … No
✔ Add JSX Support? … No
✔ Add Vue Router for Single Page Application development? … Yes
✔ Add Pinia for state management? … Yes
✔ Add Vitest for Unit testing? … No
✔ Add Cypress for both Unit and End-to-End testing? … No
✔ Add ESLint for code quality? … Yes
✔ Add Prettier for code formatting? … Yes
```
### Run your app
```
cd <your-project-name>
npm install
npm run dev
```
### Add Vuetify

```install
npm add vuetify
```
- create a vuetify file
```
/* plugins/vuetify.js */
import 'vuetify/styles'
import { createVuetify } from 'vuetify'
import * as components from 'vuetify/components'
import * as directives from 'vuetify/directives'
import { aliases, mdi } from 'vuetify/iconsets/mdi'

export const vuetify = createVuetify({
    components,
    directives,
    icons: {
        defaultSet: 'mdi',
        aliases,
        sets: {
            mdi,
        }
    },
})

```
- Add mdi icons font
```
npm add @mdi/font
```

- Add vuetify and mdi icon font into main.js
```
/** main.js */
import { createApp } from 'vue'
import { createPinia } from 'pinia'

import App from './App.vue'
import router from './router'
import './assets/main.css'

import '@mdi/font/css/materialdesignicons.css' /** mdi font */
import { vuetify } from './plugins/vuetify' /** import vuetify */

const app = createApp(App)

app.use(createPinia())
app.use(router)
app.use(vuetify) /** add vuetify */

app.mount('#app')
```