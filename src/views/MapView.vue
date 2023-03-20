<template>
  <div>
    <div id="map" class="map" />
    <button @click="openModal">UUID</button>
    <button @click="armDrone">Arm Drone</button>
  </div>

  <div id="modal-js-example" class="modal">
    <div class="modal-background"></div>

    <div class="modal-content">
      <div class="box">
        <p>Enter your sim UUID:</p>
        <div class="field is-grouped">
          <p class="control is-expanded">
            <input class="input" type="text" placeholder="uuid" v-model="uuid" />
          </p>
          <p class="control">
            <a class="button is-info" @click="getDroneLocation"> Go </a>
          </p>
        </div>
      </div>
    </div>

    <button class="modal-close is-large" aria-label="close" @click="closeModal"></button>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted } from 'vue'
import { Loader } from '@googlemaps/js-api-loader'

const UUID = ''
const map: google.maps.Map = ref([])

export default defineComponent({
  data() {
    return {
      uuid: null
    }
  },

  setup() {
    console.log('setup called')
    const loader = new Loader({
      apiKey: 'AIzaSyA5dRMkbJ6t_cQqrCIuekJd4nJyVzEeSdY',
      version: 'weekly'
    })

    onMounted(async () => {
      await loader
        .load()
        .then((google) => {
          map.value = new google.maps.Map(document.getElementById('map'), {
            center: { lat: 32.8085988, lng: -97.1563347 },
            zoom: 12
          })

          map.value.addListener('click', (event) => {
            console.log(event.latLng.lng())
            new google.maps.Marker({
              position: { lat: event.latLng.lat(), lng: event.latLng.lng() },
              map: map.value
            })
          })
        })
        .catch((error) => {
          console.log(error)
        })

      document.getElementById('modal-js-example')?.classList.add('is-active')
    })
  },

  methods: {
    async armDrone() {
      console.log('fetching data')

      const response = await fetch(`https://sim.intelligentquads.com/${this.uuid}/mavlink`)

      const json = await response.json()

      console.log(json)
    },

    async getDroneLocation(element) {
      const response = await fetch(
        `https://sim.intelligentquads.com/${this.uuid}/mavlink/vehicles/1/components/1/messages/GLOBAL_POSITION_INT`
      )
      const json = await response.json()

      const lat = json.message.lat * 1e-7
      const lon = json.message.lon * 1e-7

      const marker = new google.maps.Marker({
        position: { lat: lat, lng: lon },
        map: map.value
      })

      map.value.setCenter({ lat: lat, lng: lon })

      console.log(marker)

      this.closeModal(element)
    },

    openModal() {
      document.getElementById('modal-js-example')?.classList.add('is-active')
    },

    closeModal() {
      document.getElementById('modal-js-example')?.classList.remove('is-active')
    }
  }
})
</script>

<style scoped>
#map {
  width: 100%;
  height: 600px;
  margin: 0;
  padding: 0;
}
</style>
