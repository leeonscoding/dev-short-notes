# Layouts
* Reusable layout components.
* Create a folder called layouts under the project root directory.
* For a default layout create a file named as default.vue
* for displaying the correspond page content use vue component slot
* No change in pages, vue automatically resolve it.
* by navigating any page, we will see the default layout along with page content.

The default.vue content
```html
<template>
    <header>this is header</header>
    <nav>this is nav part</nav>
    <footer>this is footer</footer>

    <div>
        <slot/>
    </div>
</template>
```

# custom layouts
* create a layout file under layouts folder. for example, for the products page, we may need different layout. let's create the layout.vue file under the layout folder.
* attach the layout in the specific page using definePageMeta.
```html
<template>
    <div>
        <h1>Product list</h1>
    </div>
</template>

<script setup>
    definePageMeta({
        layout: 'product'
    })
</script>

<style lang="scss" scoped>
    h1 {
        background-color: aqua;
    }
</style>
```

# Add tailwind css
* Go to nuxt modules page in browser. follow instructions. I've found the following
```bash
yarn add --dev @nuxtjs/tailwindcss
```
* add tailwindcss in the modules array in nuxt.config.ts
```typescript
export default defineNuxtConfig({
  devtools: { enabled: true },
  modules: [
    '@nuxtjs/tailwindcss'
  ]
})
```

* add some code
```html
<template>
    <header class="shadow-sm bg-white">
        <nav class="container mx-auto p-4 flex justify-between">
            <ul class="flex gap-4">
                <li><NuxtLink to="/">Home</NuxtLink></li>
                <li><NuxtLink to="/about">About</NuxtLink></li>
                <li><NuxtLink to="/products">Product list</NuxtLink></li>
            </ul>
        </nav>
    </header>
    
    <footer>this is footer</footer>

    <div>
        <slot/>
    </div>
</template>
```

# add custom css
create a folder named assets, then add another folder named css and then add a file named tailwind.css
```
\
| - assets
|    | - css
|    |  |   - tailwind.css
|    |  |
```

a sample code of tailwind.css

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

body {
    @apply bg-gray-50;
}
```