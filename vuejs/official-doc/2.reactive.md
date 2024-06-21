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

## shallowRef
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

## reactive()
Another way to declare the reactive state. Reactive uses JS proxies instead of original JS objects.
```javascript
// JS proxy example
const obj = {
    msg1: 'hello',
    msg2: 'world' 
}

const handler = {}
const proxy = new Proxy(obj, handler)

console.log(proxy == obj); //false

proxy.msg1 = 'new hello'

console.log(obj.msg1) //new hello
```
Proxies acts like the original objects but they aren't.

reactive() makes an object itself instead wrapping inner values.

```html
<script setup>
    import { reactive } from 'vue'

    const state = reactive({count: 0})

    function inc() {
        state.count++
    }
    
</script>

<template>
    <div>{{ state.count }}</div>
    <button @click="inc">+</button>
</template>

<style>
</style>
```

## Reactive proxy vs original
* returned value from reactive() is a proxy of the original object which isn't equal to the original object.

```html
<script setup>
    import { reactive } from 'vue'

    const raw = {count: 0}
    const proxy = reactive(raw)

    console.log(proxy === raw) //false
    
</script>

<template>
</template>
```

* Only the proxy reactive, mutating the original object will not trigger updates.

* We can create a proxy of a proxy. Both are same.
```html
<script setup>
    import { reactive } from 'vue'

    const raw = {count: 0}
    const proxy = reactive(raw)

    console.log(reactive(raw) === proxy) //true
    console.log(reactive(proxy) === proxy) //true
    
</script>

<template>
</template>
```

* nested objects inside a reactive object are also proxies
```html
<script setup>
    import { reactive } from 'vue'

    const raw = {count: 0}
    const proxy = reactive({})
    proxy.nested = raw

    console.log(proxy.nested === raw) //false
    
</script>

<template>
</template>
```

## limitations of reactive()
* only for object types(objects, array, maps, sets). can't hold premitive types(string, number, boolean)
* can't replace entire object
* not destructure friendly: once destructure, lose the reactive connection.

## use ref() as reactive() object property
ref() automatically unwrapped when accessed/mutated as a property of a reactive object
```html
<script setup>
    import { ref, reactive } from 'vue'

    const count = ref(0)
    const state = reactive({
        count
    })

    console.log(state.count)

    state.count = 1

    console.log(count.value)
</script>

<template>
</template>
```

If we assign a new ref() then it will replace the old ref()
```html
<script setup>
    import { ref, reactive } from 'vue'

    const count = ref(0)
    const state = reactive({
        count
    })

    console.log(state.count)

    state.count = 1

    console.log(count.value)

    const newCount = ref(5)
    state.count = newCount

    console.log(state.count)
    console.log(count.value)

</script>

<template>
</template>
```

## No unwarp for arrays and collections
```html
<script setup>
    import { ref, reactive } from 'vue'

    const numbers = reactive([ref(20), ref(9), ref(30)])

    //need .value here
    console.log(numbers[1].value)

</script>
```

## unwrapping non top-level properties
```html
<script setup>
    import { ref } from 'vue'

    const object = {id: ref(5)}

</script>

<template>
    <div>{{ object.id + 1 }}</div>
</template>
```
this code will not work. the `object.id` will not unwrapped. we have to destructure the id. so, id can be the top-level property.
```html
<script setup>
    import { ref } from 'vue'

    const object = {id: ref(5)}
    const { id } = object

</script>

<template>
    <div>{{ id + 1 }}</div>
</template>
```