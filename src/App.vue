<template>
    <div>
        <div class='right-panel'>
            <h1>
                Tynnyrityökalu
            </h1>

            <br>

            <draggable v-model='route'>
                <div
                    v-for='(point, index) in route'
                    :key='"p"+index'
                    class='list-item'
                >
                    <div class='item-header'>
                        <div>{{ point }}</div>
                        <div @click='removePoint(index)' class='remove-button'>poista</div>
                    </div>
                </div>

            </draggable>

            <br><br>

            <label>Tynnyrin nimi</label>
            <input v-model='routeName' />

            <label>Kuljettaja</label>
            <select v-model='driver'>
                <option v-for='(d, i) in drivers'
                    :key='"driver"+i'
                    :label='d.name'
                    :value='d.value'
                />
            </select>

            <label>Ajopäivä</label>
            <input readonly value='ajopäivä tähän kalenterilla' />

            <label>Kuljetusyritys</label>
            <input readonly value='kuljetusyritys jos käytössä' />

            <button>Luo reitti</button>
        </div>

        <l-map
            style='height: 100vh;'
            :zoom='zoom'
            :center='center'
            @click='mapClickHandler'
        >
            <l-tile-layer
                url='https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png'
            />

            <l-marker
                v-for='(marker, index) in markers'
                :key='index'
                :latLng='marker.latLng'
                @click='() => markerClickHandler(marker, index)'
            >
                <l-icon
                    :icon-size='[30, 40]'
                    :icon-anchor='[15, 40]'
                >
                    <div
                        class='marker'
                    >
                        {{ index }}
                    </div>
                </l-icon>
            </l-marker>

            <l-polyline
                :latLngs='routePath'
            />

        </l-map>
    </div>
</template>

<script>
import { LMap, LTileLayer, LMarker, LIcon, LPolyline } from 'vue2-leaflet';
import draggable from 'vuedraggable';

export default {
    components: { LMap, LTileLayer, LMarker, LIcon, draggable, LPolyline },

    data() {
        return {
            center: [62.6029, 29.7608],
            zoom: 15,
            markers: [],
            route: [],
            routeName: 'tynnyri-' + new Date().toISOString().slice(0, 10),
            driver: -1,
            drivers: [
                { name: 'Kuski1', value: 0 },
                { name: 'Kuski2', value: 1 },
                { name: 'Kuski3', value: 2 },
                { name: 'Kuski4', value: 3 },
                { name: 'Kuski5', value: 4 },
            ]
        }
    },

    mounted() {
        this.mapClickHandler({
            originalEvent: {
                ctrlKey: true
            },
            latlng: {
                lat: this.center[0],
                lng: this.center[1]
            }
        });

        this.route = [0, 2, 5, 6, 10, 12, 13, 18];
    },

    methods: {
        mapClickHandler(event) {
            const latLng = event.latlng;
            
            if (event.originalEvent.shiftKey) {
                this.markers.push({
                    latLng: latLng
                });
            }

            if (event.originalEvent.ctrlKey) {
                for (let i = 0; i < 20; i++) {
                    this.markers.push({
                        latLng: {
                            lat: latLng.lat + Math.random() / 2 / 50,
                            lng: latLng.lng + Math.random() / 1 / 50
                        }
                    });
                }
            }
        },
        markerClickHandler(marker, index) {
            if (!this.route.includes(index)) {
                this.route.push(index);
            } else {
                this.route.splice(this.route.findIndex(e => e == index), 1);
            }
        },

        handleRouteChange(a,b) {
            console.log(a,b);
        },

        removePoint(index) {
            console.log(index);
            this.route.splice(index, 1);
        }
    },

    computed: {
        routePath() {
            return this.route.map(i => this.markers[i].latLng);
        }
    }

}
</script>

<style>


button, input, select {
    width: 100%;
    padding: 10px 0;
    margin: 10px 0;
}

select {
    width: 100%;
    padding: 10px;
}

h1 {
    padding: 10px 0;
}

input {
    padding: 10px;
    width: calc(100% - 25px);
}

* {
    padding: 0;
    margin: 0;
    font-family: Arial;
}

body {
    overflow: hidden;
}

.marker {
    background: rgba(0, 0, 50, 0.8);
    clip-path: polygon(50% 100%, 57% 92%, 66% 83%, 74% 75%, 84% 64%, 88% 59%, 93% 51%, 96% 45%, 97% 40%, 98% 36%, 98% 32%, 97% 28%, 96% 25%, 94% 21%, 90% 16%, 87% 13%, 81% 9%, 76% 6%, 69% 3%, 62% 1%, 56% 0%, 50% 0%, 50% 0%, 44% 0%, 38% 1%, 31% 3%, 24% 6%, 19% 9%, 13% 13%, 10% 16%, 6% 21%, 4% 25%, 3% 28%, 2% 32%, 2% 36%, 3% 40%, 4% 45%, 7% 51%, 12% 59%, 16% 64%, 26% 75%, 34% 83%, 43% 92%, 50% 100%);

    width: 30px;
    height: 40px;
    line-height: 30px;
    text-align: center;
    color: white;
}
.marker:hover {
    background: rgba(255, 0, 0, 0.8);
}

.right-panel {
    position: absolute;
    right: 0;
    top: 0;
    padding: 10px;
    width: 300px;
    height: 100vh;
    background: white;
    z-index: 10000;
}

.list-item {
    padding: 15px;
    border-bottom: 1px solid #999;
}

.list-item:hover {
    background: rgba(0, 0, 50, 0.8);
    color: white;
}

.item-header {
    display: flex;
    justify-content: space-between;
}

.remove-button {
    cursor: pointer;
}

.remove-button:hover {
    color: rgba(255, 0, 0, 0.8);
}
</style>
