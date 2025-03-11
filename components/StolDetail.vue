<template>
    <section :class = "{'right-[-25vw]' : !mounted}">
        <!-- <h2 class="text-lg">{{ stolperstein.personLabel.value }}</h2> -->
        <h2 class="text-lg my-3">{{ stolperstein.stolpersteinLabel.value }}</h2>
        <p>
            Voir sur Wikidata :
            <a :href="stolperstein.person.value" target="_blank" class = "text-blue-500 underline mr-2">
                {{ stolperstein.personLabel.value }}
            </a>
            <a :href="stolperstein.stolperstein.value" target="_blank" class = "text-blue-500 underline">
                le Stolperstein 
            </a>
        </p>


        <p>
           Né·e le {{ toFrenchDate(stolperstein.dateNaissanceLabel?.value) }} à {{ stolperstein.lieuNaissanceLabel?.value }}
        </p>

        <p>
            Lieu·x de détention : {{ stolperstein.lieuDetentionLabels?.value }}
        </p>

            <img v-if = "stolperstein.image" :src="stolperstein.image.value" :alt="stolperstein.stolpersteinLabel.value" 
                class = "my-3 max-w-[80vw] max-h-[60vh] md:max-w-[70%] md:max-h-[20%]"/>
    </section>
</template>

<script setup>

const mounted = ref(false);

const toFrenchDate = (date) => {
    const options = { year: 'numeric', month: 'long', day: 'numeric' };
    return new Date(date).toLocaleDateString('fr-FR', options);
}

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