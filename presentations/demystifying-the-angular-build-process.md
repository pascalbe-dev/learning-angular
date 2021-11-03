---
theme: gaia
paginate: true
backgroundImage: url('https://marp.app/assets/hero-background.svg')
marp: true
---

<!-- ![bg left:40% 80%](https://marp.app/assets/marp.svg) -->

# **Demystifying the angular build process**

- from "npm build" to a production-ready javascript bundle
- explanations about some not obvious behaviors

---

### `npm build` to `nx build` command

- npm can run scripts defined in the `package.json` via `npm run <scriptname>`

```json
  "scripts": {
    "build": "nx build",
    "test": "nx test"
  },
```

- `npm build` is a shorthand for `npm run build`
  - ❗this does not work for all scripts (only standard ones)

---

### more about `npm scripts`

- npm scripts can make use of locally installed executable
  - even when the package is not installed globally
  - -> running scripts via `npm run` makes sure, the dev dependency version is used
  - -> this is how our CI job works without installing packages globally
- if you want to pass flags to the inner command, you have to use `-- --flag`
  - -> `npm run start -- --open`

---

### `npm install` vs `npm ci`

- `npm ci` does not change the package-lock.json and is faster if no node_modules are there already -> perfect for CI job
- `npm install` might change package-lock.json depending on the constraints in the package.json

---

### `nx build` to builder / executor execution

- `angular.json` is the build configuration file for angular apps
- the file consists of different **projects** (= app or library)
  - each project can have multiple **targets**
  - each target consists of
    - **a builder**: package, which contains the code to execute the target
    - **options**: the default configuration for the builder
    - **different configurations**: e.g. production

---

### more about nx commands

- to execute a target for a project run `nx run <projectname>:<targetname>:<configuration?>`
  - configuration can be omitted (-> uses default)
  - shorthand: `nx <targetname> <projectname>` (only standard commands)
- `nx affected --target <targetname>` can run the target for every changed lib / app
- `nx affected --target <targetname> --all` can run the target for all libs / apps

---

### good to know about nx

- 🆒 since a recent nx version, it supports moving project configuration into project directory
- 🆒 nx supports computation caching
- nx overrides the ng command

---

### what does nx build do?

- call `@angular-devkit/build-angular:browser` builder / executor
- use webpack under the hood to orchestrate build
  - compile angular templates to javascript / typescript
  - transpile typescript to javascript (based on tsconfig.json)
  - compile scss to css
  - create js bundles and optimize based on configuration (tree shaking, dead code elimination, minification, ...) ([Link](https://angular.io/guide/workspace-config#optimization-configuration))

---

### custom builders / executors

- nx provides the **run-commands-executor** to write shell-based executors ([Link](https://nx.dev/l/a/workspace/run-commands-executor))
  - -> simple way to extend the build process (e.g. with docker image building)
  - -> works with affected
- you can also implement custom builders / executors in your workspace ([Link](https://nx.dev/l/a/executors/creating-custom-builders))
  - -> good way to extend the build process with complex tools
  - -> works with affected
