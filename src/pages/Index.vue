<template>
  <q-page class="flex column" :class="bgClass">
    <q-dialog v-model="showDialog">
      <q-card style="width: 300px">
        <q-card-section>
          <div class="text-h6 q-pb-none">Désolé, cette ville n'existe pas.</div>
        </q-card-section>

        <q-card-actions align="right" class="bg-white text-teal">
          <q-btn flat label="OK" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search"
        @keyup.enter="getWeatherBySearch()"
        placeholder="Search"
        dark
        borderless
      >
        <template v-slot:before>
          <q-icon @click="getLocation" name="my_location" />
        </template>

        <template v-slot:append>
          <q-btn @click="getWeatherBySearch()" round dense flat icon="search" />
        </template>
      </q-input>
    </div>

    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">
          {{ weatherData.name }}
        </div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
      </div>
      <div class="col text-center">
        <img
          :src="
            `https://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`
          "
          alt="icon"
        />
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">Quasar<br />Weather</div>
        <q-btn @click="getLocation" class="col" flat>
          <q-icon left size="3em" name="my_location" />
          <div>Find my location</div>
        </q-btn>
      </div>
    </template>

    <div class="col skyline"></div>
  </q-page>
</template>

<script>
export default {
  name: "PageIndex",
  data() {
    return {
      search: "",
      weatherData: null,
      lat: null,
      lon: null,
      apiUrl: "https://api.openweathermap.org/data/2.5/weather",
      apiKey: API_KEY,
      showDialog: false
    };
  },
  methods: {
    getLocation() {
      this.$q.loading.show();
      if (this.$q.platform.is.electron) {
        this.$axios
          .get("https://freegeoip.app/json/")
          .then(res => {
            this.lat = res.data.latitude;
            this.lon = res.data.longitude;
            this.getWeatherByCoords();
          })
          .catch(err => {
            this.$q.loading.hide();
          });
      } else {
        navigator.geolocation.getCurrentPosition(position => {
          this.lat = position.coords.latitude;
          this.lon = position.coords.longitude;
          this.getWeatherByCoords();
        });
      }
    },
    getWeatherByCoords() {
      this.$q.loading.show();
      this.$axios(
        `${this.apiUrl}?lat=${this.lat}&lon=${this.lon}&appid=${this.apiKey}&units=metric&lang=fr`
      )
        .then(res => {
          this.weatherData = res.data;
          this.$q.loading.hide();
        })
        .catch(err => {
          console.error(err);
          this.$q.loading.hide();
        });
    },
    getWeatherBySearch() {
      this.$q.loading.show();
      this.$axios(
        `${this.apiUrl}?q=${this.search}&appid=${this.apiKey}&units=metric&lang=fr`
      )
        .then(res => {
          this.weatherData = res.data;
          this.$q.loading.hide();
        })
        .catch(err => {
          console.error(err);
          this.$q.loading.hide();
          this.showDialog = true;
        });
    }
  },
  computed: {
    // eslint-disable-next-line vue/return-in-computed-property
    bgClass() {
      if (this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith("n")) {
          return "bg-night";
        } else {
          return "bg-day";
        }
      }
    }
  }
};
</script>

<style lang="scss" scoped>
.q-page {
  background: linear-gradient(to bottom, #136a8a, #267871);

  &.bg-night {
    background: linear-gradient(to bottom, #232526, #414345);
  }

  &.bg-day {
    background: linear-gradient(to bottom, #00b4db, #0083b0);
  }
}
.degree {
  top: -44px;
}
.skyline {
  flex: 0 0 100px;
  background: url("../assets/skyline.png");
  background-size: contain;
  background-position: center bottom;
}
</style>
