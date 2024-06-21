## Text interplation using Mustache or double curly braces
```html
<script setup>
    const msg = 'test message'
</script>

<template>
    <h1>{{ msg }}</h1>
</template>
```

## Raw html using v-html
Mustache can interpret only plain text. For html output use v-html
```html
<script setup>
    const p1 = `<span style="color: red">This should be red.</span>`
</script>

<template>
    <p>using v-hmtl: <span v-html="p1"></span></p>
</template>
```
*Note: v-html is a directive prefixed with v-*

## html attribute binding & shorthand
Mustache can't be used inside html attributes. Use v-bind:<attribute> or in shorthand :<attribute>
```html
<script setup>
</script>

<template>
    <div v-bind:id="green_id">dynamic bind 1</div>
    <div :id="green_id">dynamic bind 2</div>
</template>

<style>
  #green_id {
    color: green;
  }
</style>
```
For same name of the attribute, there is furthur shorthand. For example, if a div id has the name 'id' then
```html
    <div :id>further shorthand</div>
```

## boolean attributes
Do this with v-bind but v-bind works differently here. If disabled is true, then place disabled="" otherwise place nothing.
```html
<script setup>
    let isDisabled = true
</script>

<template>
    <button :disabled="isDisabled">disabled test</button>
</template>
```

## Dynamically binding multiple html attributes
For example, bind id/class/style to a div at a same time then use v-bind.
```html
<script setup>
    const multipleAttr = {
        id: 'green_id',
        class: 'yll_cls'
    }
</script>

<template>
    <p v-bind="multipleAttr">multiple attr example</p>
</template>

<style>
  #green_id {
    color: green;
  }
  .yll_cls {
    background-color: yellow;
    border: 1px solid black;
  }
</style>
```

## Example of valid JS expressions
```html
<script setup>
    let isDisabled = true
    let number = 1
    const msg = 'test message'
</script>

<template>
    <p>{{ number + 1 }}</p>
    <p>{{ isDisabled ? 'yes' : 'no' }}</p>
    <p>{{ msg.toUpperCase() }}</p>
</template>
```
*Invalid expressions: declare a variable, control flow*

## Restricted global access
Inside expressions, we can use globals like Math, Date but not window. There is a list where this access is defined. This list is inside the vue's Application api and the file is app.config.globalProperties.

## Directive arguments
```html
<script setup>
    const url = `https://vuejs.org/guide/essentials/template-syntax.html`

    const warning = () => {
        alert('test warning')
    }
</script>

<template>
    <a :href="url">click to doc</a>
    <div @click="warning">click it</div>
</template>
```
