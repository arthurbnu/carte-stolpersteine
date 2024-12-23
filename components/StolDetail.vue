<template>
    <section 
        class = "absolute z-[1000] right-0 top-0 w-[25vw] h-full !mt-0 p-4 bg-white/90 transition-all duration-500 ease-in-out will-change-[right]"
        :class = "{'right-[-25vw]' : !mounted}">
        <h2>{{ stolperstein.stolpersteinLabel.value }}</h2>
        <p>
            <a :href="stolperstein.stolperstein.value" target="_blank" class = "text-blue-500 underline">
                Voir sur Wikidata</a>
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