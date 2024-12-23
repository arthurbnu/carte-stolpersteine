<template>
    <l-marker :lat-lng="coords">
        <l-tooltip>
            {{ title }}
        </l-tooltip>
    </l-marker>
</template>

<script setup>

const props = defineProps({
    title: {
        type: String,
        required: true
    },
    place: {
        type: String,
        required: true
    },
    coords: {
        type: Array,
        required: true
    }
})

</script>