<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>KCheck2 - Detail
          <ion-badge class="ion-margin-start" :color="color">{{ statusTrain }}</ion-badge>
        </ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">KCheck2 - Detail</ion-title>
        </ion-toolbar>
      </ion-header>
      <ion-list v-for="(routeSegment) in results.routeSegments"
                :key="routeSegment.arrivalTimestamp" class="ion-margin">
        <ion-list-header lines="inset">
          <ion-text color="medium">{{ getTrainType(routeSegment.train.type) }}{{ routeSegment.train.number }}</ion-text>
        </ion-list-header>
        <ion-item :key="componentKey">
          <ul class="reset stationsDet first last">
            <li v-for="(result, index) in routeSegment.trainStops" :key="result.departureTimestamp"
                class="itemSt active"
                :class="{ free: result.free, last: index===routeSegment.trainStops.length-1 }">
              <ion-text v-if="typeof result.arrivalTimestamp != null" class="resetDet timeDet arrival"
                        v-bind:class="[ index===0 ? 'ion-color ion-color-medium' : 'ion-color ion-color-primary' ]">
                {{ formatTime(result.arrivalTimestamp) }}
              </ion-text>
              <ion-text v-if="typeof result.departureTimestamp != null" class="resetDet timeDet departure"
                        v-bind:class="[ index===routeSegment.trainStops.length-1 ? 'ion-color ion-color-medium' : 'ion-color ion-color-primary' ]">
                {{ formatTime(result.departureTimestamp) }}
              </ion-text>
              <ion-text class="station" color="medium">{{ result.trainStation.name }}</ion-text>
            </li>
          </ul>
        </ion-item>
      </ion-list>


    </ion-content>
  </ion-page>
</template>

<script>
import {defineComponent} from 'vue';
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  loadingController,
  IonList,
  IonListHeader,
  IonItem,
  IonBadge,
  IonText
} from '@ionic/vue';
import {format} from "date-fns";
import {useRoute} from "vue-router";
import {Http} from "@capacitor-community/http";

export default defineComponent({
  name: 'Tab3Page',
  components: {
    IonHeader,
    IonToolbar,
    IonTitle,
    IonContent,
    IonPage,
    IonList,
    IonListHeader,
    IonItem,
    IonBadge,
    IonText
  },
  data() {
    return {
      offer: "",
      color: "danger",
      statusTrain: "PLNÉ",
      componentKey: 0
    }
  },
  methods: {
    getTrainType(trainType) {
      if (trainType === 1) {
        return "Vlak: Os "
      } else if (trainType === 2) {
        return "Vlak: RR "
      } else if (trainType === 4) {
        return "Vlak: R "
      } else if (trainType === 16) {
        return "Vlak: Ex "
      } else if (trainType === 2048) {
        return "Vlak: rj "
      } else if (trainType === 4096) {
        return "Vlak: IC "
      } else if (trainType === 8192) {
        return "Vlak: SC "
      } else if (trainType === 16384) {
        return "Vlak: EC "
      } else if (trainType === 131072) {
        return "Vlak: EN "
      } else if (trainType === 524288) {
        return "Vlak: rjx "
      } else if (trainType === 536870912) {
        return "Vlak: REX "
      } else {
        return "Vlak: Unk "
      }
    },
    formatTime(timestamp) {
      if (timestamp === null) return;
      return format(timestamp, 'HH:mm');
    },
    async getPricing() {
      const loading = await loadingController
          .create({
            message: 'Načítavam dostupnosť...',
            duration: this.timeout,
          });
      await loading.present();
      let selfRefs = []
      for (let i = 0; i < this.results.routeSelfRefs.length; i++) {
        selfRefs.push(this.results.routeSelfRefs[i].selfRef)
      }
      await Http.request({
        method: 'GET',
        url: 'https://app.zssk.sk/api/v2/route/pricing/?travelDate=' + this.results.departureTimestamp + '&selfRef=' + selfRefs + '&ageCategory=102&ageCategoryDiscount=600&freeTransportDiscount=true&order=0',
        headers: {
          'x-api-key': 'PDh^2-$-M]8(dG8E+Q,FR}zsfz"Q~:N2pp\\ykmg9ZEgKVrh42PHS?^sQ6<3;X,?-',
          'Access-Control-Allow-Origin': '*'
        }
      }).then(async ({data}) => {
        this.offer = data.optionPricings[0].passengers[0].offer
        let isFree = await this.getFinalPricing(this.results.arrivalTimestamp, this.results.departureTimestamp, this.results.duration, this.results.length, this.offer, this.results.routeSegments, this.results.routeSelfRefs)
        console.log(isFree)
        if (isFree) {
          this.statusTrain = "Voľné"
          this.color = "success"
          console.log(data)
          this.finalpricing = data
          for (let i = 0; i < this.results.routeSegments.length; i++) {
            for (let j = 0; j < this.results.routeSegments[i].trainStops.length; j++) {
              this.results.routeSegments[i].trainStops[j].free = true
            }
          }
          this.forceRerender()
        } else {
          for (let i = 0; i < this.results.routeSegments.length; i++) {
            for (let j = 0; j < this.results.routeSegments[i].trainStops.length - 1; j++) {
              let isSegmentFree = await this.getSegmentStatus(this.results.routeSegments[i].trainStops[j].departureTimestamp, this.results.routeSegments[i].trainStops[j + 1].arrivalTimestamp, this.results.routeSegments[i].trainStops[j].trainStation.uicCode, this.results.routeSegments[i].trainStops[j + 1].trainStation.uicCode)
              if (isSegmentFree) {
                this.results.routeSegments[i].trainStops[j].free = true
                this.results.routeSegments[i].trainStops[j + 1].free = true
              }
              this.forceRerender()
              console.log(this.results)
            }
          }
        }
      })
      await loading.dismiss();
    },
    async getFinalPricing(arrTime, depTime, dur, len, offer, rs, rsr) {
      const data = {
        "allowedPaymentModes": [],
        "arrivalTimestamp": arrTime,
        "departureTimestamp": depTime,
        "duration": dur,
        "finalOfferExpiration": 0,
        "id": 0,
        "infoForCurrentConnection": 0,
        "infoForNextConnection": 0,
        "infoForPreviousConnection": 0,
        "isSummaryExpanded": false,
        "length": len,
        "loaderResponseSize": 0,
        "loaderType": 0,
        "passengers": [
          {
            "ageCategoryDiscountId": 600,
            "ageCategoryId": 102,
            "firstName": "Test",
            "freeTransportDiscount": true,
            "isUser": true,
            "lastName": "Test",
            "offer": offer,
            "order": 0,
            "priceInLong": 0,
            "reservation": [
              {
                "isEdited": true,
                "placeNumber": -1,
                "relativePosition": 8,
                "reservationCategory": -1,
                "reservationType": 0,
                "segmentOrder": 0,
                "trainClass": 1,
                "wagonNumber": -1
              }
            ],
            "ticketRequired": true,
            "zsskRegistrationNumber": 0,
            "reservationPlaces": []
          }
        ],
        "priceFrom": 0,
        "reservationOnly": false,
        "routeSegments": rs,
        "routeSelfRefs": rsr,
        "state": 0,
        "tickets": [],
        "timeForCurrentConnection": 0,
        "timeForNextConnection": 0,
        "timeForPreviousConnection": 0,
        "type": "FULL_CUSTOMER_ACCOUNT"
      }
      let statusRet
      await Http.request({
        method: 'POST',
        url: 'https://app.zssk.sk/api/v2/route/final-pricing/',
        data: data,
        headers: {
          'x-api-key': 'PDh^2-$-M]8(dG8E+Q,FR}zsfz"Q~:N2pp\\ykmg9ZEgKVrh42PHS?^sQ6<3;X,?-',
          'Access-Control-Allow-Origin': '*',
          'Content-Type': 'application/json'
        }
      }).then(({status}) => {
        console.log(status === 200)
        statusRet = status
      })
      return statusRet === 200;
    },
    forceRerender() {
      this.componentKey += 1;
    },
    async getSegmentStatus(departureTime, arrivalTime, UicFrom, UicTo) {
      let retBool
      await Http.request({
        method: 'GET',
        url: 'https://app.zssk.sk/api/v1/route?fromStation=' + UicFrom + '&toStation=' + UicTo + '&departure=true&ageCategory=102&ageCategoryDiscount=600&travelDate=' + departureTime,
        headers: {
          'x-api-key': 'PDh^2-$-M]8(dG8E+Q,FR}zsfz"Q~:N2pp\\ykmg9ZEgKVrh42PHS?^sQ6<3;X,?-',
          'Access-Control-Allow-Origin': '*'
        }
      }).then(async ({data}) => {
        let selfRefs = []
        for (var i = 0; i < data[0].routeSelfRefs.length; i++) {
          selfRefs.push(data[0].routeSelfRefs[i].selfRef)
        }
        let duration = data[0].duration
        let length = data[0].length
        let rs = data[0].routeSegments
        let rsr = data[0].routeSelfRefs
        await Http.request({
          method: 'GET',
          url: 'https://app.zssk.sk/api/v2/route/pricing/?travelDate=' + departureTime + '&selfRef=' + selfRefs + '&ageCategory=102&ageCategoryDiscount=600&freeTransportDiscount=true&order=0',
          headers: {
            'x-api-key': 'PDh^2-$-M]8(dG8E+Q,FR}zsfz"Q~:N2pp\\ykmg9ZEgKVrh42PHS?^sQ6<3;X,?-',
            'Access-Control-Allow-Origin': '*'
          }
        }).then(async ({data}) => {
          let offer = data.optionPricings[0].passengers[0].offer
          retBool = await this.getFinalPricing(arrivalTime, departureTime, duration, length, offer, rs, rsr);
        })
      })
      return retBool
    }
  },
  setup() {
    const route = useRoute();
    let results = JSON.parse(route.params.data)
    for (let i = 0; i < results.routeSegments.length; i++) {
      for (let j = 0; j < results.routeSegments[i].trainStops.length; j++) {
        results.routeSegments[i].trainStops[j].free = false
      }
    }
    return {
      results
    }
  },
  mounted() {
    this.getPricing()
  },
})
;
</script>

<style>
.timeDet:before {
  content: "";
  width: 10px;
  height: 10px;
  border-radius: 5px;
  background: #c0c4c7;
  position: absolute;
  left: 64px;
  top: auto;
  display: block;
  margin: 4px 0 0 0;
}

.itemSt.active .timeDet:before {
  background: #eb445a;
}

.itemSt.free .timeDet:before {
  background: #2dd36f;
}

.itemSt:first-child:before {
  top: 0.5em;
}

.stationsDet .itemSt:before {
  content: "";
  width: 2px;
  background: #eb445a;
  position: absolute;
  left: 51px;
  top: -7px;
  bottom: -4px;
  display: block;
  margin: 0;
  z-index: 1;
}

.stationsDet .free:before {
  background: #2dd36f;
  top: 12px;
}

.stationsDet .itemSt:first-child:before {
  top: 0.5em;
}

.stationsDet .itemSt.last:before {
  bottom: 10px;
}

.stationsDet .itemSt .timeDet {
  width: 47px;
  float: left;
  margin-left: -64px;
  text-align: right;
  font-weight: bold;
}

.resetDet {
  border: none;
  background: none;
  margin: 0;
  padding: 0;
}

.stationsDet .itemSt .station {
  margin: 0;
}

.stationsDet .itemSt {
  padding: 0 0 2px 110px;
  clear: both;
  position: relative;
}

.stationsDet .itemSt .arrival {
  width: 58px;
  position: absolute;
  top: auto;
  left: 47px;
  font-weight: bold;
}

.stationsDet .itemSt .departure {
  width: 120px;
  position: absolute;
  top: auto;
  left: 47px;
  font-weight: bold;
}

.stationsDet {
  margin-top: 10px;
  margin-bottom: 10px;
}
</style>
