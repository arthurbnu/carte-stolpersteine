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
                    {{ toFrenchDate(stolperstein.dateNaissanceLabel?.value) }} ({{ stolperstein.lieuNaissanceLabel?.value}})
                    -
                    {{ toFrenchDate(stolperstein.dateMortLabel?.value) }} ({{ stolperstein.lieuMortLabel?.value }})
                </p>

                <p>
                    Lieu·x de détention : {{ stolperstein.lieuDetentionLabels?.value }}
                </p>

                <img v-if="stolperstein.image" :src="stolperstein.image.value"
                    :alt="stolperstein.stolpersteinLabel.value"
                    class="my-3 max-w-[70vw] max-h-[60vh] md:max-w-[70%] md:max-h-[20%]" />
            </li>
        </ul>
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
    },
})

const allClickedMarkers = inject("allClickedMarkers");

</script>

<style>
body {
    overflow-x: hidden;
}
</style>