# install
```bash
npx nuxi@latest init <project-name>
```

# VScode extensions
* Vue official
* Vue VSCode Snippets

# Files
* nuxt configuration in nuxt.config.ts
* default root config in app.vue file

# Run dev server
```bash
npm run dev
```

# Adding pages
* create a folder called pages in the root directory.
* create a file with .vue extension which is the page. For example: /pages/about.vue
* Now we can delete the default app.vue component. Everything can now inside the pages folder.
* For homepage, should add the index.vue file. For index.vue, the route is the base path(/)

# Demo index.vue file
use shortcut vbase-3 to generate code
```html
<template>
    <div>
        <h1>test</h1>
    </div>
</template>

<script>
export default {
    setup () {
        

        return {}
    }
}
</script>

<style lang="scss" scoped>

</style>
```

# install preprocessor
```bash
npm install -D sass
```

and rerun dev server

## Note: 
* If we don't want scss, then delete lang="scss" part
* Nuxt creates routes based on our file name.

# Subfolders => sub routes
* Subfolders inside the pages folder treats as sub directories

For example, create a folder products under pages folder. create a file named hello.vue inside the products folder. The route then /products/hello

Also, if we create index.vue then the route is /products

# Route parameters
For example, the route is /products/{id}. Here id is different for each products. We need a same component for each products with different ids.

create a file under products folder. the name should be wrapped by square braces. Inside braces, the name is the changeable item. In this case it is product id. So, the file name is [id].vue

## Extract route params
```html
<template>
    <div>
        <h1>Details for {{ id }}</h1>
    </div>
</template>

<script setup>
    const {id} = useRoute().params;
</script>

<style lang="scss" scoped>

</style>
```

Destructure params from useRoute().params

## String templating using double curly braces.

# NuxtLink
Using anchor tag, it will refresh the page and get component from server. If we need only render component then use NuxtLink. This is the spa behavior we need. Once get a component from server we only re-render in client.
```html
<template>
    <div>
        <h1>test</h1>

        <ul>
            <li><NuxtLink to="/">Home</NuxtLink></li>
            <li><NuxtLink to="/about">About</NuxtLink></li>
            <li><NuxtLink to="/products">Product list</NuxtLink></li>
        </ul>
    </div>
</template>

<script>
export default {
    setup () {
        

        return {}
    }
}
</script>

<style lang="scss" scoped>

</style>
```