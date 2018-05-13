# mc_lighting_ui

> New and improved alternative web UI for McLighting.

This web UI was completely rebuilt from scratch unsing the reactive vus.js and Vuetify UI framework. It's still in preview and not completed. You can use it in parallel with the current UI if you want.

It has the following new features:
* Fast and reactive UI.
* Responsive UI that adapts to display size.
* Everything on one page, change colors for effects while running using the color wheel.
* New color wheel.
* New websocket connector that automatically reconnects.
* Configurable:
  * Dark mode
  * Type of color wheel
  * Effects can be hidden
* You can add additional McLighting nodes that can be controlled together with the master.

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report
```

For a detailed explanation on how things work, check out the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).
