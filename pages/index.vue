<template>

    <main>
        <!-- <input type="text" v-model="currentCity" id = "current-city"/> -->
         <select v-model="currentCity" id = "current-city">
            <option v-for="city in cities" :value="city.id">{{ city.name }}</option>
        </select>
        <section class="w-[100vw] h-[100vh]" :class = "{'animate-pulse': pending}">
            <LMap
                v-if = "sparqlResult" 
                ref="map" :zoom="zoom" 
                @click="clickedMarker = null"
                :center="[sparqlResult[0].latitude.value, sparqlResult[0].longitude.value]"
                :use-global-leaflet="false">
                <LTileLayer url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
                    attribution="&amp;copy; <a href=&quot;https://www.openstreetmap.org/&quot;>OpenStreetMap</a> contributors"
                    layer-type="base" name="OpenStreetMap" />
                <Marker v-for="stolperstein in sparqlResult" @click="clickedMarker = stolperstein"
                    :coords="[stolperstein.latitude.value, stolperstein.longitude.value]"
                    :title="stolperstein.stolpersteinLabel.value">
                    
                    <StolDetail v-if="clickedMarker" :stolperstein="clickedMarker" class="block md:hidden w-[90vw]"/>

                </Marker>
                <!-- cercle pour mettre en évidence du marqueur cliqué  -->
                <l-circle-marker v-if = "clickedMarker" 
                    :lat-lng="[clickedMarker.latitude.value, clickedMarker.longitude.value]" 
                    :radius="6" color="green">
                </l-circle-marker>
            </LMap>
        </section>

        <StolDetail v-if="clickedMarker" :stolperstein="clickedMarker"
            class = "hidden md:block absolute z-[1000] right-0 top-0 w-[25vw] h-full !mt-0 p-4 bg-white/90 transition-all duration-500 ease-in-out will-change-[right] border-gray-300 border-l-8 border-solid"
        />

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
const center = ref([47.413220, -1.219482]);

const cities = [
    {name: 'Saverne', id: 'Q22741'},
    {name: 'Strasbourg', id : 'Q6602'},
    {name: 'Bouxwiller ', id: 'Q22740'},
    // {name: 'Benfeld', id: 'Q22776'},
    // {name: 'Balbrom', id: 'Q22793'},
    // {name: 'Barr ', id: 'Q22758'},
    // {name: 'Bischwiller ', id: 'Q22431'},
    // {name: 'Gries ', id: 'Q22481'},
]

const currentCity = ref(cities[0].id);


const request = computed( () => 
`#defaultView:Map
    SELECT ?stolperstein ?stolpersteinLabel ?coords ?latitude ?longitude ?image WHERE {
      SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],fr". }
      ?stolperstein (wdt:P31/wdt:P279*) wd:Q26703203;
        wdt:P131 wd:${currentCity.value};
        wdt:P625 ?coords.
        OPTIONAL { ?stolperstein wdt:P18 ?image}
      BIND(geof:latitude(?coords) AS ?latitude)
      BIND(geof:longitude(?coords) AS ?longitude)
    }`
)

const url = computed(() => `https://query.wikidata.org/sparql?query=${encodeURIComponent(request.value)}&format=json`)
const headers = { 'Accept': 'application/json' }
const { data: sparqlResult, error, pending, execute: refresh } = await useFetch(url, { headers: headers, server: false, transform: res => res.results.bindings });

</script>

<style scoped>

#current-city{
    position: fixed;
    top: 20px;
    left: 48vw;
    z-index: 1000;
}

 @media screen and (max-width: 640px) {
    #current-city{
        left: unset;
        right: 10px;
    }
    
  } 
 

</style>

<style>

.leaflet-popup{
    width: 90vw;
}
</style>