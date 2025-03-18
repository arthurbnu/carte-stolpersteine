<template>
    <section :class="{ 'right-[-25vw]': !mounted }" class="max-h-[40vh] lg:max-h-[100vh] overflow-y-auto">
        <ul >
            <li v-for="stolperstein in allClickedMarkers" :key="stolperstein.stolperstein.value" class="p-1">
                <h2 class="text-lg my-3">{{ stolperstein.stolpersteinLabel.value }}</h2>
                <p>
                    Voir sur Wikidata :
                    <a :href="stolperstein.person.value" target="_blank" class="text-blue-500 underline mr-2">
                        {{ stolperstein.personLabel.value }}
                    </a>
                    <a :href="stolperstein.stolperstein.value" target="_blank" class="text-blue-500 underline">
                        le Stolperstein
                    </a>
                </p>

                <p>
                    {{ toFrenchDate(stolperstein.dateNaissanceLabels?.value) }} ({{ stolperstein.lieuNaissanceLabels?.value}})
                    -
                    {{ toFrenchDate(stolperstein.dateMortLabels?.value) }} ({{ stolperstein.lieuMortLabels?.value }})
                </p>

                <p v-if = "stolperstein.lieuDetentionLabels?.value">
                    Lieu·x de détention : {{ stolperstein.lieuDetentionLabels.value}}
                </p>

                <img v-for="(image, id) in getImages(stolperstein)" :src="image" :key="id"
                    :alt="stolperstein.stolpersteinLabel.value"
                    class="my-3 max-w-[70vw] max-h-[60vh] md:max-w-[70%] md:max-h-[20%]" />
            </li>
        </ul>
    </section>
</template>

<script setup>

const dateOptions = { year: 'numeric', month: 'long', day: 'numeric' };
const valueSeparator = " | ";
const toFrenchDate = date => date.split(valueSeparator).map(d => new Date(d).toLocaleDateString('fr-FR', dateOptions)).join(valueSeparator);


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
    },
})

const getImages = stol => stol.imageLabels?.value.split(valueSeparator).filter(img => img !== "")

const allClickedMarkers = inject("allClickedMarkers");

</script>

<style>
body {
    overflow-x: hidden;
}
</style>