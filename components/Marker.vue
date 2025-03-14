<template>
    <l-marker :lat-lng="coords" class="">
        <!-- :icon-url="highlight ? 'stolperstein-highlight.png' : 'stolperstein.png'" -->
        <l-icon 
            :icon-url="hasImage ? 'stolperstein-avec-photo.png' : 'stolperstein.png'"
            :icon-size="[30, 30]" :icon-anchor="[15, 25]"
            :tooltip-anchor="[10, -10]">
        </l-icon>
        <!-- <div>
            <img src="/stolperstein.png" alt="Stolperstein" width="2" height="2" class="h-7"/>
            <div class="">
                {{ title }}
            </div>
        </div> -->
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
    },
    hasImage: {
        type: Boolean,
        default: false
    }
})

const iconUrl = ref('stolperstein.png');

watch(() => props.highlight, h => iconUrl.value = h ? 'stolperstein-highlight.png' : 'stolperstein.png');


</script>

<style>
img.leaflet-marker-icon {
    transition: filter 0.2s ease-in-out;
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