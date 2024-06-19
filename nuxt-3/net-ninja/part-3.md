# fetch data
using useFetch. example:

```html
<template>
    <div>
        <h1>Product list</h1>

        <div>
            <ul v-for="p in products">
                <li class="p-4">
                    <NuxtLink :to="`/products/${p.id}`">{{ p.title }}</NuxtLink>
                </li>
            </ul>
        </div>
    </div>
</template>

<script setup>
    definePageMeta({
        layout: 'product'
    })

    const { data: products } = await useFetch('https://fakestoreapi.com/products')
</script>

<style lang="scss" scoped>
    h1 {
        background-color: aqua;
    }
</style>
```