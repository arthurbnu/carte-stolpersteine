<template>

    <main>
        <!-- <input type="text" v-model="currentCity" id = "current-city"/> -->
        <select v-model="currentCity" id="current-city">
            <!-- <option v-for="city in citiesResult" :value="city.id">{{ city.name }}</option> -->
            <option v-for="city in citiesResult" :value="city.id">{{ city.name }}</option>
        </select>
        <section class="w-[100vw] h-[100vh]" :class="{ 'animate-pulse': pending }">
            <LMap v-if="sparqlResult?.length > 0" ref="map" :zoom="zoom" @click="clickedMarker = null"
                :center="centerPoint"
                :use-global-leaflet="false"
                >
                <LTileLayer url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
                    attribution="&amp;copy; <a href=&quot;https://www.openstreetmap.org/&quot;>OpenStreetMap</a> contributors"
                    layer-type="base" name="OpenStreetMap" />
                <Marker v-for="stolperstein in sparqlResult" :key="stolperstein.stolperstein.value"
                    @click="clickedMarker = stolperstein"
                    :highlight="clickedMarker === stolperstein"
                    :coords="[stolperstein.latitude.value, stolperstein.longitude.value]"
                    :title="stolperstein.personLabel.value">

                    <StolDetail v-if="clickedMarker" :stolperstein="clickedMarker" class="min-w-[90%]" />

                </Marker>

                <l-marker v-if="userPosition" :lat-lng="userPosition" />
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

const clickedMarker = ref(null);

const zoom = ref(15);
// const center = ref([47.413220, -1.219482]);

const cities = [
    { name: 'Strasbourg', id: 'Q6602' },
    { name: 'Saverne', id: 'Q22741' },
    { name: 'Bouxwiller ', id: 'Q22740' },
]

const currentCity = ref(cities[0].id);

const request = computed(() =>
    `#defaultView:
    SELECT ?stolperstein ?stolpersteinLabel ?coords ?latitude ?longitude ?image ?person ?personLabel  ?dateNaissanceLabel ?lieuNaissanceLabel ?dateMortLabel ?lieuMortLabel  
# ?year ?month ?day
(GROUP_CONCAT(DISTINCT ?lieuDetentionLabel; SEPARATOR = " | ") AS ?lieuDetentionLabels) 

WHERE {
      SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],fr". }
      ?stolperstein (wdt:P31/wdt:P279*) wd:Q26703203;
        wdt:P131 wd:${currentCity.value};
        wdt:P547 ?person;
        wdt:P625 ?coords.
      
#       ?person wdt:P735 ?prenom;
#               wdt:P734 ?nom.
      OPTIONAL {?person wdt:P19 ?lieuNaissance}
      OPTIONAL {?person wdt:P20 ?lieuMort}

  OPTIONAL {?person wdt:P569 ?dateNaissance
                      OPTIONAL {?person wdt:P570 ?dateMort}
.
#                bind (year(?dateNaissance) as ?year)
#                bind (month(?dateNaissance) as ?month)
#                bind (day(?dateNaissance) as ?day)
               }
      OPTIONAL {?person wdt:P2632 ?lieuDetention}
      OPTIONAL { ?stolperstein wdt:P18 ?image}
    
      SERVICE wikibase:label {
        bd:serviceParam wikibase:language "fr".
        ?lieuDetention rdfs:label ?lieuDetentionLabel.
      }
  
      BIND(geof:latitude(?coords) AS ?latitude)
      BIND(geof:longitude(?coords) AS ?longitude)
    }

GROUP BY ?stolperstein ?stolpersteinLabel ?coords ?latitude ?longitude ?image ?person ?personLabel ?dateNaissanceLabel ?lieuNaissanceLabel ?dateMortLabel ?lieuMortLabel   `
)

const url = computed(() => `https://query.wikidata.org/sparql?query=${encodeURIComponent(request.value)}&format=json`)
const headers = { 'Accept': 'application/json' }
const { data: sparqlResult, error, pending, execute: refresh } = await useFetch(url, { headers: headers, server: false, transform: res => res.results.bindings });


// Récupération de la liste des villes du Bas-Rhin ayant des stolpersteine
const citiesRequest = `
#title: Villes du Bas-Rhin ayant des stolpersteine
SELECT DISTINCT ?ville ?villeLabel WHERE {
  SERVICE wikibase:label { bd:serviceParam wikibase:language "fr". }
  ?stolperstein (wdt:P31/(wdt:P279*)) wd:Q26703203;
    wdt:P625 ?coords;
    wdt:P131 ?ville.
  ?ville wdt:P131 wd:Q12717.
}
ORDER BY ?villeLabel  
`

const { data: citiesResult, error: citiesError, status: citiesStatus, execute: citiesRefresh } =
    await useFetch("https://query.wikidata.org/sparql", {
        headers,
        params: { query: citiesRequest, format: 'json' },
        server: false,
        transform: res => res.results.bindings.map(city => ({ name: city.villeLabel.value, id: city.ville.value.replace('http://www.wikidata.org/entity/', '') }))
    });
// console.log(citiesResult.value);

// Calcule le centre de la carte
const centerPoint = computed(() => {
    if (sparqlResult.value?.length > 0) {
        const lat = sparqlResult.value.reduce((acc, stolperstein) => acc + parseFloat(stolperstein.latitude.value), 0) / sparqlResult.value.length;
        const lon = sparqlResult.value.reduce((acc, stolperstein) => acc + parseFloat(stolperstein.longitude.value), 0) / sparqlResult.value.length;
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



</script>

<style scoped>
#current-city {
    position: fixed;
    top: 20px;
    left: 48vw;
    z-index: 1000;
}

@media screen and (max-width: 640px) {
    #current-city {
        left: unset;
        right: 10px;
    }

}
</style>

<style>
.leaflet-popup {
    width: 90vw;
}
</style>