<template>
  <div>
    <div id="map" class="map" />
    <button @click="openModal">UUID</button>
    <button @click="armDrone">Arm Drone</button>
    <button @click="takeOff">Takeoff</button>
    <button @click="land">Land</button>
  </div>

  <div id="modal-js-example" class="modal">
    <div class="modal-background"></div>

    <div class="modal-content">
      <div class="box">
        <p>Enter your sim UUID:</p>
        <div class="field is-grouped">
          <p class="control is-expanded">
            <input class="input" type="text" placeholder="uuid" ref="uuid" />
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
import { defineComponent, ref, onMounted } from "vue";
import { Loader } from "@googlemaps/js-api-loader";

export default defineComponent({
  setup() {
    const map = ref<HTMLDivElement>();
    const uuid = ref<HTMLInputElement>();
    const HOST = 'https://sim.intelligentquads.com/'

    const loader = new Loader({
      apiKey: "AIzaSyA5dRMkbJ6t_cQqrCIuekJd4nJyVzEeSdY",
      version: "weekly",
    });

    onMounted(async () => {
      await loader
        .load()
        .then((google) => {
          map.value = new google.maps.Map(document.getElementById("map") as HTMLElement, {
            center: { lat: 32.8085988, lng: -97.1563347 },
            zoom: 12,
          });

          map.value.addListener("click", (event: any) => {
            new google.maps.Marker({
              position: { lat: event.latLng.lat(), lng: event.latLng.lng() },
              map: map.value,
            });
            gotoLocation(event.latLng);
          });
        })
        .catch((error) => {
          console.log(error);
        });

      document.getElementById("modal-js-example")?.classList.add("is-active");
    });

    const armDrone = () => {
      console.log('arm');
      const response = fetch(
        `${HOST}${uuid.value.value}/mavlink`,
        {
          method: "POST",
          body: JSON.stringify({
            header: {
              system_id: 255,
              component_id: 240,
              sequence: 0,
            },
            message: {
              type: "COMMAND_LONG",
              param1: 1.0,
              param2: 0.0,
              param3: 0.0,
              param4: 0.0,
              param5: 0.0,
              param6: 0.0,
              param7: 0.0,
              command: {
                type: "MAV_CMD_COMPONENT_ARM_DISARM",
              },
              target_system: 1,
              target_component: 1,
              confirmation: 1,
            },
          }),
        }
      );
    };

    const getDroneLocation = async () => {
      const response = await fetch(
        `${HOST}${uuid.value.value}/mavlink/vehicles/1/components/1/messages/GLOBAL_POSITION_INT`
      );
      const json = await response.json();

      const lat = json.message.lat * 1e-7;
      const lon = json.message.lon * 1e-7;

      const marker = new google.maps.Marker({
        position: { lat: lat, lng: lon },
        map: map.value,
      });

      map.value.setCenter({ lat: lat, lng: lon });

      closeModal();
    };

    const gotoLocation = (latlng: google.maps.LatLng) => {
      console.log(
        `going to location lat: ${latlng.lat() * 1e7}, lng: ${latlng.lng() * 1e7}`
      );
      const response = fetch(
        `${HOST}${uuid.value.value}/mavlink`,
        {
          method: "POST",
          body: JSON.stringify({
            header: {
              system_id: 255,
              component_id: 240,
              sequence: 0,
            },
            message: {
              type: "COMMAND_INT",
              target_system: 1,
              target_component: 1,
              frame: {
                type: "MAV_FRAME_GLOBAL",
              },
              current: 0,
              autocontinue: 0,
              command: {
                type: "MAV_CMD_DO_REPOSITION",
              },
              param1: -1.0,
              param2: 1,
              param3: 0.0,
              param4: 0.0,
              x: parseInt(latlng.lat() * 1e7),
              y: parseInt(latlng.lng() * 1e7),
              z: 591,
            },
          }),
        }
      );
    };

    const takeOff = () => {
      const response = fetch(
        `${HOST}${uuid.value.value}/mavlink`,
        {
          method: "POST",
          body: JSON.stringify({
            header: {
              system_id: 255,
              component_id: 240,
              sequence: 0,
            },
            message: {
              type: "COMMAND_LONG",
              param1: 0.0,
              param2: 0.0,
              param3: 0.0,
              param4: 0.0,
              param5: 0.0,
              param6: 0.0,
              param7: 100.0,
              command: {
                type: "MAV_CMD_NAV_TAKEOFF",
              },
              target_system: 1,
              target_component: 1,
              confirmation: 0,
            },
          }),
        }
      );
    };

    const land = () => {
      const response = fetch(
        `${HOST}${uuid.value.value}/mavlink`,
        {
          method: "POST",
          body: JSON.stringify({
            header: {
              system_id: 255,
              component_id: 240,
              sequence: 0,
            },
            message: {
              type: "COMMAND_LONG",
              param1: 0.0,
              param2: 0.0,
              param3: 0.0,
              param4: 0.0,
              param5: 0.0,
              param6: 0.0,
              param7: 0.0,
              command: {
                type: "MAV_CMD_NAV_LAND",
              },
              target_system: 1,
              target_component: 1,
              confirmation: 0,
            },
          }),
        }
      );
    };

    const openModal = () => {
      document.getElementById("modal-js-example")?.classList.add("is-active");
    };

    const closeModal = () => {
      document.getElementById("modal-js-example")?.classList.remove("is-active");
    };

    return {
      map,
      uuid,
      armDrone,
      takeOff,
      land,
      getDroneLocation,
      openModal,
      closeModal,
    };
  },

  methods: {
    openModal() {
      document.getElementById("modal-js-example")?.classList.add("is-active");
    },

    closeModal() {
      document.getElementById("modal-js-example")?.classList.remove("is-active");
    },
  },
});
</script>

<style scoped>
#map {
  width: 100%;
  height: 90vh;
  margin: 0;
  padding: 0;
}
</style>
