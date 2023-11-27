
<script>
import TheWelcome from '@/components/TheWelcome.vue'
import * as THREE from 'three'


export default {
    data() {
        return {
            msg: 'Welcome to Your Vue.js + TypeScript App'
        }
    },
    async mounted() {

    },
}
</script>

<template>
    <main>{{ msg }}
        <TheWelcome />
        <div id="container"></div>
    </main>
</template>
<style>
#container {
    width: 100%;
    height: 600px;
}
</style>