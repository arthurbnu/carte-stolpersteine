<template>
    <section :class = "{'right-[-25vw]' : !mounted}">
        <h2 class="text-lg">{{ stolperstein.stolpersteinLabel.value }}</h2>
        <p>
            <a :href="stolperstein.stolperstein.value" target="_blank" class = "text-blue-500 underline">
                Voir sur Wikidata
            </a>
            <img v-if = "stolperstein.image" :src="stolperstein.image.value" :alt="stolperstein.stolpersteinLabel.value" 
                class = "my-3 max-w-[80vw] max-h-[60vh] md:max-w-[70%] md:max-h-[20%]"/>
        </p>
    </section>
</template>

<script setup>

const mounted = ref(false);

onMounted(() => {
    setTimeout(() => {
        mounted.value = true;
    }, 100);
})

const props = defineProps({
    stolperstein: {
        type: Object,
        required: true
    }
})

</script>

<style>
body{
    overflow-x: hidden;
}
</style>