<template>
  <div id="app">
    <div class="row no-gutters">
      <!-- 選擇地區 -->
      <div class="toolbox col-sm-3 p-2 bg-white">
        <div class="form-group d-flex">
          <label for="city" class="col-form-label mr-2 text-right">縣市</label>
          <div class="flex-fill">
            <select id="city" v-model="cityName" class="form-control">
              <option v-for="option in cityData" :key="option.id" :value="option.name">
                {{ option.name }}
              </option>
            </select>
          </div>
        </div>
        <div class="form-group d-flex">
          <label for="dist" class="col-form-label mr-2 text-right">地區</label>
          <div class="flex-fill">
            <select id="dist" v-model="district" @change="updateMarkers" class="form-control" placeholder="請選擇區域">
              <!-- 製作下拉選單 -->
              <option value="" disabled>
                請選擇區域
              </option>
              <option v-for="option in cityData.find((city) => city.name === cityName).districts" :key="option.id" :value="option">
                {{ option.name }}
              </option>
            </select>
          </div>
        </div>
      </div>

      <!-- 顯示地圖和 UBike 站點 -->
      <div class="col-sm-9">
        <div id="map"></div>
      </div>
    </div>
  </div>
</template>

<script>
import L from 'leaflet'
import cityData from './assets/taipaicity.json'

export default {
  name: 'App',
  data() {
    return {
      userPos: [],
      cityData,
      cityName: '臺北市',
      district: '',
      bikeData: [],
      map: null,
      yourMarker: null,
      bikeMarkers: []
    }
  },
  computed: {
    ubikes() {
      const filteData = this.bikeData.filter((bike) => {
        return bike.sarea === this.district.name
      })
      return filteData
    }
  },
  async created() {
    const { data } = await this.$http.get('https://tcgbusfs.blob.core.windows.net/blobyoubike/YouBikeTP.json')
    this.bikeData = Object.keys(data.retVal).map((key) => data.retVal[key])
  },
  mounted() {
    this.getPos()
  },
  methods: {
    getPos() {
      navigator.geolocation.getCurrentPosition((pos) => {
        this.userPos = [pos.coords.latitude, pos.coords.longitude]
        this.initMap()
      })
    },
    initMap() {
      this.map = L.map('map', {
        center: this.userPos,
        zoom: 16
      })
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        maxZoom: 18,
        attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, ' +
          '<a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, ' +
          'Imagery © <a href="https://www.mapbox.com/">Mapbox</a>'
      }).addTo(this.map)
      this.yourMarker = L.marker(this.userPos).addTo(this.map).bindPopup('<b>你</b>').openPopup()
    },
    updateMarkers() {
      this.removeMarkers()
      this.bikeMarkers = this.ubikes.map((bike) => {
        return L.marker([bike.lat, bike.lng]).addTo(this.map).bindPopup(
          `<p><strong style="font-size: 20px;">${bike.sna}</strong></p>
          <strong style="font-size: 16px; color: #d45345;">可租借車輛剩餘：${bike.sbi} 台</strong><br>
          可停空位剩餘: ${bike.bemp}<br>
          <small>最後更新時間: ${bike.mday}</small>`
        )
      })
      this.moveMap()
    },
    removeMarkers() {
      this.bikeMarkers.forEach((layer) => {
        this.map.removeLayer(layer)
      })
      // 搜尋所有圖層，如果塗層來自 Marker (L.Marker) 就移除他
      // this.map.eachLayer((layer) => {
      //   console.log(layer)
      //   if (layer instanceof L.Marker) {
      //     this.map.removeLayer(layer)
      //   }
      // })
    },
    moveMap() {
      this.map.flyTo([this.district.latitude, this.district.longitude], 14)
    }
  }
}
</script>

<style lang="scss">
  #map{
    height: 100vh;
  }
</style>
