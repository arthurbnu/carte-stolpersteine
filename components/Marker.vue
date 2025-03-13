<template>
    <l-marker :lat-lng="coords" class="">
       <l-icon :icon-url="highlight ? 'stolperstein-highlight.png' : 'stolperstein.png'"
        :icon-size="[30, 30]" :icon-anchor="[15, 25]"
            :tooltip-anchor="[10, -10]"></l-icon>
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
    },
    highlight: {
        type: Boolean,
        default: true
    }
})

const iconUrl = ref('stolperstein.png');

watch(() => props.highlight, h => iconUrl.value = h ? 'stolperstein-highlight.png' : 'stolperstein.png');


</script>

<style>
img.leaflet-marker-icon {
    transition: all 0.2s ease-in-out;
    &:focus {
        filter: brightness(1.1);
    }
}

@media (hover: none) {
    .leaflet-tooltip-pane {
        display: none;
    }
}

/* @media (hover: hover) {
    .leaflet-popup-pane{
        display: none;
    }
} */

/*  tailwind lg */
@media (min-width: 1024px) {
    .leaflet-popup-pane {
        display: none;
    }
}
</style>