<template>

    <main>
        <!-- <input type="text" v-model="currentCity" id = "current-city"/> -->
         <!-- <fieldset class="block lg:flex"> -->
         <fieldset class="flex">
                <select v-model="currentCity" id="current-city" class = "max-w-[35vw]">
                    <option v-for="city in citiesResult" :key="city.id" :value="city.id" :title="city.count + ' résultats'">{{ city.name }}</option>
                </select>
                <br>
                <div class = " lg:inline">
                    <input type = "text" id = "search" v-model = "search" placeholder= "👤 rechercher" list="stolperstein-list" class = "max-w-[35vw]"/>
                    <datalist id="stolperstein-list">
                        <option v-for="stolperstein in filteredResults" :key="stolperstein.stolperstein.value" :value="stolperstein.personLabel.value"></option>
                    </datalist>
                    <span class="hidden lg:inline text-gray-500 ml-2">{{ filteredResults?.length }} résultats</span>
                </div>
        </fieldset>
        <section class="w-[100vw] h-[100vh]" :class="{ 'animate-pulse': pending }">
            <LMap v-if="sparqlResult?.length > 0" ref="map" :zoom="zoom" @click="clickedMarker = null"
                :center="centerPoint" :use-global-leaflet="false" >
                <LTileLayer url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
                    attribution="&amp;copy; <a href=&quot;https://www.openstreetmap.org/&quot;>OpenStreetMap</a> contributors"
                    layer-type="base" name="OpenStreetMap" />
                <Marker v-for="stolperstein in filteredResults" :key="stolperstein.stolperstein.value"
                    @click="handleMarkerClick(stolperstein)" :highlight="clickedMarker === stolperstein"
                    :coords="[stolperstein.latitude.value, stolperstein.longitude.value]"
                    :has-image="test && !!stolperstein.image"
                    :is-group="stolperstein.people"
                    :title="stolperstein.people ?? stolperstein.personLabel.value">

                    <StolDetail v-if="clickedMarker" :stolperstein="clickedMarker"
                        :allClickedMarkers="allClickedMarkers" class="min-w-[90%]" />

                </Marker>

                <l-marker v-if="userPosition" :lat-lng="userPosition">
                    <l-tooltip>
                        Vous êtes ici
                    </l-tooltip>
                </l-marker>
                <!-- cercle pour mettre en évidence du marqueur cliqué  -->
                <!-- <l-circle-marker v-if="clickedMarker"
                    :lat-lng="[clickedMarker.latitude.value, clickedMarker.longitude.value]" :radius="6" color="green">
                </l-circle-marker> -->
            </LMap>
        </section>

        <StolDetail v-if="clickedMarker" :stolperstein="clickedMarker"
            class="hidden lg:block absolute z-[1000] right-0 top-0 w-[25vw] h-full !mt-0 p-4 bg-white/90 transition-all duration-500 ease-in-out will-change-[right] border-gray-300 border-l-8 border-solid" />

        <section class="hidden">
            <pre>
                {{ sparqlResult }}
            </pre>
        </section>

    </main>

</template>

<script setup>

const search = ref('')
const getStrVal = s => s.value.trim().toUpperCase()
const filteredResults = computed(() => sparqlResult.value?.filter(stolp => getStrVal(stolp.personLabel).includes(getStrVal(search)) ))

const clickedImg = ref(null);
const clickedMarker = ref(null);
// Tous les marqueurs de stolpersteine qui ont les mêmes coordonnées que le marqueur cliqué
const allClickedMarkers = ref([]);

const setBrightness = (img, value = 1) => {
    if (img) 
        img.style.filter = `brightness(${value})`
}

const handleMarkerClick = async stolperstein => {
    setBrightness(clickedImg.value)
    await nextTick()
    clickedImg.value = document.activeElement
    setBrightness(clickedImg.value, 1.2)
    clickedMarker.value = stolperstein
    const sameCoords = sparqlResult.value.filter(stolp => stolp.latitude.value === stolperstein.latitude.value && stolp.longitude.value === stolperstein.longitude.value)
    allClickedMarkers.value = sameCoords
}

watchEffect(() => {
    if (! clickedMarker.value && clickedImg.value) 
        setBrightness(clickedImg.value)
})

provide("allClickedMarkers", allClickedMarkers);

// paramètre test si url avec ?test=1
const test = !!useRoute().query.test

const zoom = ref(15);
// const center = ref([47.413220, -1.219482]);

const cities = [
    { name: 'Strasbourg', id: 'Q6602' },
    { name: 'Saverne', id: 'Q22741' },
    { name: 'Bouxwiller ', id: 'Q22740' },
]

const currentCity = ref(cities[0].id);

const request = computed(() =>
`#title: Stolpersteine
SELECT ?stolperstein ?stolpersteinLabel ?coords ?latitude ?longitude ?person ?personLabel 
(GROUP_CONCAT(DISTINCT ?lieuDetentionLabel; SEPARATOR = " | ") AS ?lieuDetentionLabels) 
(GROUP_CONCAT(DISTINCT ?lieuNaissanceLabel; SEPARATOR = " | ") AS ?lieuNaissanceLabels) 
(GROUP_CONCAT(DISTINCT ?lieuMortLabel; SEPARATOR = " | ") AS ?lieuMortLabels) 
(GROUP_CONCAT(DISTINCT ?dateNaissanceLabel; SEPARATOR = " | ") AS ?dateNaissanceLabels) 
(GROUP_CONCAT(DISTINCT ?dateMortLabel; SEPARATOR = " | ") AS ?dateMortLabels) 
(GROUP_CONCAT(DISTINCT ?imageLabel; SEPARATOR = " | ") AS ?imageLabels) 
WHERE {
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],fr". }
  ?stolperstein (wdt:P31/(wdt:P279*)) wd:Q26703203;
    wdt:P131 wd:${currentCity.value};
    wdt:P547 ?person;
    wdt:P625 ?coords.
  OPTIONAL { ?person wdt:P19 ?lieuNaissance. }
  OPTIONAL { ?person wdt:P20 ?lieuMort. }
  OPTIONAL { ?person wdt:P569 ?dateNaissance. }
  OPTIONAL { ?person wdt:P570 ?dateMort. }
  OPTIONAL { ?person wdt:P2632 ?lieuDetention. }
  OPTIONAL { ?stolperstein wdt:P18 ?image. }
  SERVICE wikibase:label {
    bd:serviceParam wikibase:language "fr".
    ?lieuDetention rdfs:label ?lieuDetentionLabel.
    ?lieuNaissance rdfs:label ?lieuNaissanceLabel.
    ?lieuMort rdfs:label ?lieuMortLabel.
    ?dateNaissance rdfs:label ?dateNaissanceLabel.
    ?dateMort rdfs:label ?dateMortLabel.
    ?image rdfs:label ?imageLabel.
  }
  BIND(geof:latitude(?coords) AS ?latitude)
  BIND(geof:longitude(?coords) AS ?longitude)
}
GROUP BY ?stolperstein ?stolpersteinLabel ?coords ?latitude ?longitude ?person ?personLabel   `
)

const url = computed(() => `https://query.wikidata.org/sparql?query=${encodeURIComponent(request.value)}&format=json`)
const headers = { 'Accept': 'application/json' }
const { data: sparqlResult, error, pending, execute: refresh } = await useFetch(url, { headers: headers, server: false, transform: res => res.results.bindings });


// Récupération de la liste des villes du Bas-Rhin ayant des stolpersteine
const citiesRequest = `
#title: Villes du Bas-Rhin ayant des stolpersteine
SELECT ?ville ?villeLabel (COUNT(?stolperstein) AS ?stolpersteinCount) WHERE {
  SERVICE wikibase:label { bd:serviceParam wikibase:language "fr". }
  ?stolperstein (wdt:P31/(wdt:P279*)) wd:Q26703203;
    wdt:P625 ?coords;
    wdt:P131 ?ville.
  ?ville wdt:P131 wd:Q12717.
}
GROUP BY ?ville ?villeLabel
ORDER BY ?villeLabel
`

const { data: citiesResult, error: citiesError, status: citiesStatus, execute: citiesRefresh } =
    await useFetch("https://query.wikidata.org/sparql", {
        headers,
        params: { query: citiesRequest, format: 'json' },
        server: false,
        transform: res => res.results.bindings.map(city => ({ name: city.villeLabel.value, id: city.ville.value.replace('http://www.wikidata.org/entity/', ''), count: city.stolpersteinCount.value }))
    });

// Calcule le centre de la carte
const centerPoint = computed(() => {
    if (filteredResults.value?.length > 0) {
        const usedResults = filteredResults.value?.length === 1 ? filteredResults.value : sparqlResult.value;
        const lat = usedResults.reduce((acc, stolperstein) => acc + parseFloat(stolperstein.latitude.value), 0) / usedResults.length;
        const lon = usedResults.reduce((acc, stolperstein) => acc + parseFloat(stolperstein.longitude.value), 0) / usedResults.length;
        return [lat, lon];
    }
})

const userPosition = ref(null);
onMounted(() => {
    if (navigator.geolocation) {
        const watchId = navigator.geolocation.watchPosition(position => {
            userPosition.value = [position.coords.latitude, position.coords.longitude];
        });
    }
});

watch(currentCity, () => search.value = '');

// watch(search, () => {
//     if (search.value) {
//         zoom.value = 15;
//     }
// })

watchEffect(() => {
    // on vérifie si certains points sont au même endroit

    if (sparqlResult.value?.length > 0) {
    console.log(sparqlResult.value);
    // console.log(Object.groupBy(sparqlResult.value, s => s.person.value));

        const coordCounts = {};
        sparqlResult.value.forEach(stolperstein => {
            const coord = `${stolperstein.latitude.value},${stolperstein.longitude.value}`;
            if (coordCounts[coord]) {
                coordCounts[coord]++;
            } else {
                coordCounts[coord] = 1;
            }
        });
        const duplicates = Object.keys(coordCounts).filter(coord => coordCounts[coord] > 1);

        // On met les noms des personnes dans le champ people
        duplicates.forEach(coord => {
            const sameCoords = sparqlResult.value.filter(stolp => `${stolp.latitude.value},${stolp.longitude.value}` === coord);
            const people = sameCoords.map(stolp => stolp.personLabel.value).join(', ');
            sameCoords.forEach(stolp => {
                stolp.people = people
            });
        });
    }
});



</script>

<style scoped>
fieldset{
    gap: 15px;
    position: fixed;
    top: 20px;
    left: 48vw;
    z-index: 1000;
}

@media screen and (max-width: 1024px) {
    fieldset {
        /* left: unset;
        right: 10px; */
        top: 48px;
        left: 55px;
    }
    /*
    #current-city {
        left: unset;
        right: 10px;
    }
        */

}
</style>

<style>
/* on redéfinit la largeur calculée par leaflet qui ne prend pas en compte le contenu dynamique */
.leaflet-popup-content {
    width: 80vw !important;
}
</style>