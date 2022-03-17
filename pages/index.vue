<template>
  <div class="main">
    <v-overlay :value="loading" style="z-index: 9999">
      <v-progress-circular indeterminate color="primary"></v-progress-circular>
    </v-overlay>
    <div class="route-list pa-2">
      <h1 class="text-h5 text-sm-h4">Az Országos Kéktúra útvonalváltozásai</h1>
      <p class="text-body-2 text-sm-body-1">
        Ezen a honlapon összehasonlíthatod, hogy merre vezetett az Országos
        Kéktúra útvonala különböző, tetszőlegesen választott időpontokban.
      </p>
      <v-dialog v-model="dialog" width="500">
        <template #activator="{ on, attrs }">
          <v-btn color="primary" outlined block small v-bind="attrs" v-on="on">
            További információk
          </v-btn>
        </template>

        <v-card class="pt-3">
          <v-card-text>
            <h4>Mi ez?</h4>
            A honlap az
            <a href="https://openstreetmap.org">OpenStreetMap</a> adatbázisa
            alapján mutatja meg, hogy merre vezetett az Országos Kéktúra
            útvonala különböző időpontokban. Az útvonalat 2013-ban vitték fel
            először egyben az adatbázisba, ezért ekkori a legkorábbi dátum, amit
            ki lehet választani. Az útvonalak véletlenszerűen kiválasztott
            színnel jelennek meg.
            <h4 class="mt-2">Ki csinálta?</h4>
            A honlapot Hann András készítette. Személyes honlapja a
            <a href="https://hann.io">hann.io</a> címen, míg túrázással
            kapcsolatos podcastja, a <b>Túranapló</b> a
            <a href="https://podcast.hiking.hu/">podcast.hiking.hu</a> címen
            (vagy akár például a
            <a href="https://open.spotify.com/show/5pxn5dbj4u6Ax5eLk3QTDA"
              >Spotifyon</a
            >) található meg.
            <h4 class="mt-2">Ötletem van, hibát találtam. Mit csináljak?</h4>
            Írj egy emailt a [keresztnevem]@hann.io címre! <br />
            A honlap forráskódját
            <a href="https://github.com/andrashann/kektura-valtozasok">
              itt találod</a
            >.
          </v-card-text>

          <v-divider></v-divider>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="primary" text @click="dialog = false"> Ok </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <v-divider class="mt-4 mb-3"> </v-divider>

      <h3>Útvonalváltozatok</h3>
      <div v-if="processedQuery">
        <v-card v-for="(route, index) in routes" :key="index" class="my-2 pa-2">
          <v-row>
            <v-col cols="5" sm="12" align="center">
              <v-menu
                v-model="route.dateMenu"
                :close-on-content-click="false"
                :nudge-right="40"
                transition="scale-transition"
                offset-y
                min-width="auto"
              >
                <template #activator="{ on, attrs }">
                  <v-text-field
                    v-model="route.date"
                    prepend-icon="mdi-calendar"
                    readonly
                    v-bind="attrs"
                    hide-details="auto"
                    class="pt-0"
                    v-on="on"
                  ></v-text-field>
                </template>
                <v-date-picker
                  v-model="route.date"
                  :min="dates[0]"
                  :max="today.toISOString().substring(0, 10)"
                  @input="dateChanged(index)"
                ></v-date-picker>
              </v-menu>
            </v-col>
            <v-col cols="7" sm="12">
              <v-menu offset-y :close-on-content-click="false">
                <template #activator="{ on }">
                  <v-btn :color="route.color" v-on="on"> &nbsp; </v-btn>
                </template>
                <v-color-picker
                  v-model="route.color"
                  :value="route.color"
                  buttons
                  class="mx-auto"
                >
                </v-color-picker>
              </v-menu>
              <v-btn :disabled="index == 0" @click="moveUp(index)">
                <v-icon dark> mdi-arrow-up-bold-outline </v-icon>
              </v-btn>
              <v-btn
                :disabled="index == routes.length - 1"
                @click="moveDown(index)"
              >
                <v-icon dark> mdi-arrow-down-bold-outline </v-icon>
              </v-btn>
              <v-btn @click="routes.splice(index, 1)">
                <v-icon dark> mdi-delete-outline </v-icon>
              </v-btn>
            </v-col>
          </v-row>
        </v-card>
      </div>

      <v-btn color="primary" block @click="newRoute">
        <v-icon dark> mdi-plus </v-icon> Hozzáadás
      </v-btn>
      <p class="text-center text-body-2 mt-5">
        <a :href="permalinkUrl"
          >Permalink ezekhez a dátumokhoz, színekhez, térképkivágathoz
        </a>
      </p>
      <p class="text-center text-caption mt-5">
        Honlap: &copy; Hann András, 2022, GPL-3 licensz.
        <a href="https://github.com/andrashann/kektura-valtozasok"
          >GitHub repo</a
        >.
        <br />
        Adatok: &copy;
        <a href="https://openstreetmap.org">OpenStreetMap</a> közreműködők, ODBL
        licensz.
      </p>
    </div>

    <div class="route-map">
      <client-only>
        <l-map
          ref="theMap"
          :zoom="mapZoom"
          :center="mapCenter"
          @update:zoom="updateMapZoom($event)"
          @update:center="updateMapCenter($event)"
        >
          <l-tile-layer
            url="https://tile.openstreetmap.org/{z}/{x}/{y}.png"
            attribution="&copy; <a href='http://osm.org/copyright'>OpenStreetMap</a> contributors"
          >
          </l-tile-layer>
          <l-control-scale
            position="bottomleft"
            :imperial="false"
            :metric="true"
          ></l-control-scale>
          <l-polyline
            v-for="(route, index) in routes"
            :key="index"
            :lat-lngs="route.line"
            :color="route.color"
          >
            <l-popup>{{ route.date }}</l-popup>
          </l-polyline>
        </l-map>
      </client-only>
    </div>
  </div>
</template>

<script>
import clone from 'just-clone'

export default {
  name: 'IndexPage',
  data() {
    return {
      dialog: false,
      today: new Date(),
      mapZoom: 7,
      currentMapZoom: 7,
      mapCenter: [47.4979, 19.0402],
      currentMapCenter: { lat: 47.4979, lng: 19.0402 },
      processedQuery: false,
      routes: [
        {
          date: '2013-06-25',
          color: '#' + Math.floor(Math.random() * 16777215).toString(16) + 'ff',
          line: [[[0, 0]]],
        },
        {
          date: '2022-01-01',
          color: '#' + Math.floor(Math.random() * 16777215).toString(16) + 'ff',
          line: [[[0, 0]]],
        },
      ],
      dates: ['2000-01-01'],
      loading: true,
    }
  },
  computed: {
    permalinkUrl() {
      return (
        '?d=' +
        this.routes.map((r) => r.date).join(',') +
        '&c=' +
        this.routes.map((r) => r.color.substring(1, 7)).join(',') +
        '&o=' +
        this.currentMapCenter.lat.toString().substring(0, 9) +
        ',' +
        this.currentMapCenter.lng.toString().substring(0, 9) +
        '&z=' +
        this.currentMapZoom
      )
    },
  },
  watch: {
    mapCenter(newVal) {
      this.$refs.theMap.mapObject.setView(newVal, this.mapZoom)
    },
  },
  async created() {
    this.routes[this.routes.length - 1].date = this.today
      .toISOString()
      .substring(0, 10)
    const response = await fetch(
      'https://storage.googleapis.com/storage/v1/b/kektura-history/o'
    )
    const bucketInfo = await response.json()
    this.dates = bucketInfo.items.map((i) => i.name.substring(4, 14))
    this.dates.sort()

    if (this.$route.query.d) {
      this.routes = []
      for (const d of this.$route.query.d.split(',')) {
        this.routes.push({
          date: d,
          color: '#' + Math.floor(Math.random() * 16777215).toString(16) + 'ff',
          line: [[[0, 0]]],
        })
      }
    }
    if (this.$route.query.c) {
      const splitColors = this.$route.query.c.split(',')
      for (let i = 0; i < splitColors.length; i++) {
        if (i < this.routes.length) {
          this.routes[i].color = '#' + splitColors[i]
        }
      }
    }
    if (this.$route.query.z) {
      this.mapZoom = Number(this.$route.query.z)
    }
    if (this.$route.query.o) {
      const coords = this.$route.query.o.split(',').map((c) => Number(c))
      this.mapCenter = coords
    }

    this.processedQuery = true

    for (let i = 0; i < this.routes.length; i++) {
      this.routes[i].line = this.geoJsonToLine(
        await this.getGeoJSON(this.routes[i].date)
      )
    }
    this.loading = false
  },
  methods: {
    updateMapZoom(e) {
      this.currentMapZoom = e
    },
    updateMapCenter(e) {
      this.currentMapCenter = e
    },
    async newRoute() {
      this.loading = true
      this.routes.push({
        date: this.today.toISOString().substring(0, 10),
        color: '#' + Math.floor(Math.random() * 16777215).toString(16) + 'ff',
        line: this.geoJsonToLine(
          await this.getGeoJSON(this.today.toISOString().substring(0, 10))
        ),
      })
      this.loading = false
    },
    moveUp(i) {
      const r = this.routes[i]
      this.routes.splice(i, 1)
      this.routes.splice(i - 1, 0, r)
    },
    moveDown(i) {
      const r = this.routes[i]
      this.routes.splice(i, 1)
      this.routes.splice(i + 1, 0, r)
    },
    async dateChanged(i) {
      this.loading = true
      this.routes[i].dateMenu = false
      this.routes[i].line = this.geoJsonToLine(
        await this.getGeoJSON(this.routes[i].date)
      )
      this.loading = false
    },

    async getGeoJSON(date) {
      const allDates = clone(this.dates)
      let selectedDate = allDates[0]
      for (const d of allDates) {
        if (d < date) {
          selectedDate = d
        }
      }

      const urlResponse = await fetch(
        'https://www.googleapis.com/storage/v1/b/kektura-history/o/okt-' +
          selectedDate +
          '.geojson'
      )
      const urlData = await urlResponse.json()

      const response = await fetch(urlData.mediaLink)
      const data = await response.json()
      return data
    },
    geoJsonToLine(geoJSON) {
      if (geoJSON.features[0].geometry.type === 'MultiLineString') {
        return geoJSON.features[0].geometry.coordinates.map((l) =>
          l.map((c) => c.reverse())
        )
      } else if (geoJSON.features[0].geometry.type === 'LineString') {
        return geoJSON.features[0].geometry.coordinates.map((c) => c.reverse())
      }
    },
  },
}
</script>

<style scoped>
.main {
  height: 100vh;
  width: 100vw;
  display: grid;
  grid-template-columns: 400px auto;
  grid-template-rows: 100%;
  grid-template-areas: 'l m';
  position: fixed;
}

@media only screen and (max-width: 650px) {
  .main {
    grid-template-columns: auto;
    grid-template-rows: 45% auto;
    grid-row-gap: 2px;
    grid-template-areas:
      'l'
      'm';
  }
}

.route-list {
  grid-area: l;
  overflow-y: scroll;
}

.route-map {
  grid-area: m;
}
</style>
