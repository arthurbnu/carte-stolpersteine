<template>

    <main>
        <section class="w-[100vw] h-[100vh]">
            <LMap v-if="!pending" ref="map" :zoom="zoom" @click="clickedMarker = null"
                :center="[sparqlResult[0].latitude.value, sparqlResult[0].longitude.value]"
                :use-global-leaflet="false">
                <LTileLayer url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
                    attribution="&amp;copy; <a href=&quot;https://www.openstreetmap.org/&quot;>OpenStreetMap</a> contributors"
                    layer-type="base" name="OpenStreetMap" />
                <Marker v-for="stolperstein in sparqlResult" @click="clickedMarker = stolperstein"
                    :coords="[stolperstein.latitude.value, stolperstein.longitude.value]"
                    :title="stolperstein.stolpersteinLabel.value" :place="stolperstein.stolpersteinLabel.value">
                </Marker>
                <!-- cercle pour mettre en évidence du marqueur cliqué  -->
                <l-circle-marker v-if = "clickedMarker" 
                    :lat-lng="[clickedMarker.latitude.value, clickedMarker.longitude.value]" 
                    :radius="6" color="green">
                </l-circle-marker>
            </LMap>
        </section>

        <StolDetail v-if="clickedMarker" :stolperstein="clickedMarker" />

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

const request = `
#defaultView:Map
SELECT ?stolperstein ?stolpersteinLabel ?coords ?latitude ?longitude WHERE {
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],fr". }
  ?stolperstein (wdt:P31/wdt:P279*) wd:Q26703203.
  ?stolperstein wdt:P131 wd:Q22741.
  ?stolperstein wdt:P625 ?coords.
  BIND(geof:latitude(?coords) AS ?latitude)
  BIND(geof:longitude(?coords) AS ?longitude)
}`

const url = `https://query.wikidata.org/sparql?query=${encodeURIComponent(request)}&format=json`
const headers = { 'Accept': 'application/json' }
const { data: sparqlResult, error, pending, execute: refresh } = await useFetch(url, { headers: headers, server: false, transform: res => res.results.bindings });

</script>