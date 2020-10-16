<template>
  <view class="carPage" v-if="carPageState">
    <nb-header>
      <nb-left>
        <nb-button transparent :on-press="goBack">
          <nb-text>Back</nb-text>
        </nb-button>
      </nb-left>
      <nb-body>
        <nb-title>{{ data.data[0].kenteken }}</nb-title>
      </nb-body>
      <nb-right>
        <nb-button transparent>
      </nb-button>
      </nb-right>
    </nb-header>
    <nb-content>
      <nb-list>
        <nb-list-item itemDivider>
          <nb-text>ALGEMENE INFORMATIE</nb-text>
        </nb-list-item>
        <nb-list-item>
          <nb-text>Kenteken: {{ data.data[0].kenteken }}</nb-text>
        </nb-list-item>
        <nb-list-item>
          <nb-text>Merk: {{ data.data[0].merk }}</nb-text>
        </nb-list-item>
        <nb-list-item>
          <nb-text>Type: {{ data.data[0].handelsbenaming }}</nb-text>
        </nb-list-item>
        <nb-list-item>
          <nb-text>Soort: {{ data.data[0].voertuigsoort }}</nb-text>
        </nb-list-item>
        <nb-list-item>
          <nb-text>Kleur: {{ data.data[0].eerste_kleur }}</nb-text>
        </nb-list-item>
        <nb-list-item>
          <nb-text>Bouwjaar: {{ datumEersteToelanting }}</nb-text>
        </nb-list-item>
        <nb-list-item>
          <nb-text>Nieuwprijs: {{ covertPirce }}</nb-text>
        </nb-list-item>
        <nb-list-item>
          <nb-text>Brandstof: {{ fuelData.data[0].brandstof_omschrijving }}</nb-text>
        </nb-list-item>
        <nb-list-item>
          <nb-text>Vermogen: {{ fuelData.data[0].nettomaximumvermogen }} kW / {{ Math.round(fuelData.data[0].nettomaximumvermogen *  1.362)}} PK</nb-text>
        </nb-list-item>
        <nb-list-item>
          <nb-text>Aantal deuren: {{ data.data[0].aantal_deuren }}</nb-text>
        </nb-list-item>
        <nb-list-item>
          <nb-text>Aantal zitplekken: {{ data.data[0].aantal_zitplaatsen }}</nb-text>
        </nb-list-item>
        <nb-list-item>
          <nb-text>Inrichting: {{ data.data[0].inrichting }}</nb-text>
        </nb-list-item>
        <nb-list-item>
          <nb-text>APK tot: {{ apk }}</nb-text>
        </nb-list-item>
        <nb-list-item itemDivider>
          <nb-text>MOTOR EN MILIEU</nb-text>
        </nb-list-item>
        <nb-list-item>
          <nb-text>Vermogen: {{ fuelData.data[0].nettomaximumvermogen }} kW / {{ Math.round(fuelData.data[0].nettomaximumvermogen *  1.362)}} PK</nb-text>
        </nb-list-item>
        <nb-list-item>
          <nb-text>Aantal cilinders: {{ data.data[0].aantal_cilinders }}</nb-text>
        </nb-list-item>
        <nb-list-item>
          <nb-text>Cilinderinhoud: {{ data.data[0].cilinderinhoud }}cc</nb-text>
        </nb-list-item>
        <nb-list-item>
          <nb-text>Verbruik stad: {{ fuelData.data[0].brandstofverbruik_stad }}l/100km</nb-text>
        </nb-list-item>
        <nb-list-item>
          <nb-text>Verbruik snelweg: {{ fuelData.data[0].brandstofverbruik_buiten }}l/100km</nb-text>
        </nb-list-item>
      </nb-list>
    </nb-content>
  </view>
</template>

<script>
import EventBus from '../eventBus';
import * as Haptics from 'expo-haptics';


export default {
  data() {
    return {
      data: [],
      fuelData: [],
      carPageState: false
    }
  },
  computed: {
    apk() {
      let dateString = this.data.data[0].vervaldatum_apk
      let year = dateString.substring(0, 4)
      let month = dateString.substring(4, 6)
      let day = dateString.substring(6, 8)

      return `${day}-${month}-${year}`
    },
    datumEersteToelanting() {
      let dateString = this.data.data[0].datum_eerste_toelating
      let year = dateString.substring(0, 4)
      let month = dateString.substring(4, 6)
      let day = dateString.substring(6, 8)

      return `${day}-${month}-${year}`
    },
    covertPirce() {
      if (this.data.data[0].catalogusprijs != undefined) {
        let price = this.data.data[0].catalogusprijs

        return new Intl.NumberFormat('de-DE', { style: 'currency', currency: 'EUR' }).format(price)
      } else {
        return 'Geen prijs beschrikbaar'
      }

    }
  },
  mounted() {
    console.log(this.data.data)
    EventBus.$on('carData', (data) => {
      this.data = data
    })

    EventBus.$on('fuelData', (data) => {
      this.fuelData = data
    })

    EventBus.$on('carPageState', (data) => {
      this.carPageState = data
    })
  },
  methods: {
    goBack() {
			Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success)
      this.carPageState = false
    }
  }
}
</script>

<style>
  .carPage {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: white;
  }
</style>