<template>
    <l-marker :lat-lng="coords" class="">
        <l-icon
            icon-url="stolperstein.png"
            :icon-size="[30, 30]"
            :icon-anchor="[15, 35]"
            :tooltip-anchor="[10, -20]"
      

        ></l-icon>
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