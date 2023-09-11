# Using Angular Modules & Optimizing Apps

* **declarations**: components, pipes or directives wich our module uses
* **imports**: other modules (we import everything other module exports)
* **providers**: services; everything we provid here will be provided for a whole app (in case of app.module)
* **bootstrap**: main component

---

* You should import `import { CommonModule } from '@angular/common';` in each module, but not in app.module (it already provided by BrowserModule).
* You **MUST NOT** declare components, pipes or directives in more than one module! You can use shared module.
* **DO NOT** provide Services in Shared Modules! Especially not, if you plan to use them in Lazy Loaded Modules!

---

**Preload Lazy Loaded Modules:**
```ts
import { PreloadAllModules } from '@angular/router';
...
RouterModule.forRoot(appRoutes, {preloadingStrategy: PreloadAllModules})
```