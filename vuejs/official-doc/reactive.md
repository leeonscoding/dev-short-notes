## recommand way is declaring reactive state using the ref() function
```html
<script>
    import { ref } from 'vue'

    export default {
        setup() {
            const count = ref(0)

            function inc() {
                count.value++;
            }

            return {
                count,
                inc
            };
        }

    }
    
</script>

<template>
    <div>{{ count }}</div>
    <button @click="inc">+</button>
</template>
```
* ref() takes an argument
* returns the arg wrapped within a ref object.
* We can access the arg using the .value property
* `setup` is a special hook dedicated for the Composition API.
* Don't need .value in template, refs are automatically unwrapped inside template.

## Using SFC(single file component)
Avoid verbose setup() hook using `<script setup>`
```html
<script setup>
    import { ref } from 'vue'

    const count = ref(0)

    function inc() {
        count.value++;
    }
</script>

<template>
    <div>{{ count }}</div>
    <button @click="inc">+</button>
</template>
```
* top level imports, variables, functions declared inside `<script setup>` are automaically usable in the template.
* when we change a ref's value later from template, vue automatically detects the  change and updates dom accordingly.
* dependency-tracking based reactivity. when a component is rendered first time, vue tracks every ref that was used during the last render. Later, when a ref is changed/mutated, it will trigger a re-render for the compoent.
* using .value, vue can detect/track a ref is mutated/changed.
* under the hood, vue uses getter/setter of the ref's .value

## Deep reactivity
refs can hold any value type, including deeply nested objects, arrays, Map etc.
```html
<script setup>
    import { ref } from 'vue'

    const obj = ref({
        nest: {
            count: 0
        },
        arr: ['foo', 'bar']
    })

    function trigger() {
        obj.value.nest.count++
        obj.value.arr.push(obj.value.nest.count)
    }
    
</script>

<template>
    <h1>p3</h1>
    <div>{{ obj.nest.count }}</div>
    <ul>
        <li v-for="item in obj.arr">{{ item }}</li>
    </ul>
    <button @click="trigger">+</button>
</template>

<style>
</style>
```

## shalloref
For large size object, this function makes the .value only reactive. This is used for performance optimization.
```html
<script setup>
    import { ref, shallowRef } from 'vue'

    const obj = shallowRef({
        nest: {
            count: 0
        },
        arr: ['foo', 'bar']
    })

    function trigger() {
        const newCount = obj.value.nest.count + 1
        const newArr = [...obj.value.arr, newCount]

        obj.value = {
            nest: {
                count: newCount
            },
            arr: newArr
        }
    }
    
</script>

<template>
    <h1>p3</h1>
    <div>{{ obj.nest.count }}</div>
    <ul>
        <li v-for="item in obj.arr">{{ item }}</li>
    </ul>
    <button @click="trigger">+</button>
</template>

<style>
</style>
```

*When we mutate a reactive state, the DOM is updated automatically in asynchronously. No matter how many state changes, vue updates only once when nextTick calls. We can also call it.*

```html
<script setup>
    import { nextTick, ref } from 'vue'

    const count = ref(0)

    async function inc() {
        count.value++;
        await nextTick()
    }
    
</script>

<template>
    <div>{{ count }}</div>
    <button @click="inc">+</button>
</template>

<style>
</style>
```