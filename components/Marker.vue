<template>
    <l-marker :lat-lng="coords" class="">
        <l-tooltip>
            {{ title }}
        </l-tooltip>
        <l-popup>
             <slot></slot>
        </l-popup>
    </l-marker>
</template>

<script setup>

const props = defineProps({
    title: {
        type: String,
        required: true
    },
    coords: {
        type: Array,
        required: true
    }
})

</script>

<style>

@media (hover: none) {
    .leaflet-tooltip-pane{
        display: none;
    }
}

@media (hover: hover) {
    .leaflet-popup-pane{
        display: none;
    }
}
</style>