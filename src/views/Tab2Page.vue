<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>KCheck2 - Výsledky hľadania</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">KCheck2 - Výsledky hľadania</ion-title>
        </ion-toolbar>
      </ion-header>
      <ion-list id="container">
        <ion-item
            v-for="(result) in results"
            :key="result.uicCode"
            @click="openDetail(result)">

          <ul class="reset stations first last">
            <li class="itemSt active">
              <ion-text class="reset time" color="primary">{{ formatTime(result.departureTimestamp) }}</ion-text>
              <ion-text class="station" color="medium">{{
                  result.routeSegments[0].trainStops[0].trainStation.name
                }}
              </ion-text>
              <ion-text class="ion-float-right" color="medium">{{timeConvert(result.duration)}}{{result.length}}km, {{getNumTransfer(result.routeSegments.length - 1)}}</ion-text>
            </li>
            <li class="itemSt active last">
              <ion-text class="reset time" color="primary">{{ formatTime(result.arrivalTimestamp) }}</ion-text>
              <ion-text class="station" color="medium">{{ nameOfTrainStopsLastSegment(result) }}</ion-text>
            </li>
          </ul>
        </ion-item>
      </ion-list>
    </ion-content>
  </ion-page>
</template>

<script>
import {defineComponent} from 'vue';
import {IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonText, IonList, IonItem} from '@ionic/vue';
import {useRoute, useRouter} from "vue-router";
import {format} from 'date-fns';

export default defineComponent({
  name: 'Tab2Page',
  components: {IonHeader, IonToolbar, IonTitle, IonContent, IonPage, IonText, IonList, IonItem},
  data() {
  },
  methods: {
    getNumTransfer(numRouteSegments) {
      if (numRouteSegments === 0) {
        return "0 prestupov"
      } else if(numRouteSegments === 1) {
        return "1 prestup"
      } else if(numRouteSegments > 1 && numRouteSegments < 5) {
        return numRouteSegments + " prestupy"
      } else {
        return numRouteSegments + " prestpov"
      }
    },
    timeConvert(n) {
      const hours = (n / 60);
      const rhours = Math.floor(hours);
      const minutes = (hours - rhours) * 60;
      const rminutes = Math.round(minutes);
      return `${rhours}  h, ${rminutes} min, `
    },
    formatTime(timestamp) {
      return format(timestamp, 'HH:mm');
    },
    numberOfSegments(result) {
      return result.routeSegments.length
    },
    nameOfTrainStopsLastSegment(result) {
      return result.routeSegments[this.numberOfSegments(result) - 1].trainStops[result.routeSegments[this.numberOfSegments(result) - 1].trainStops.length - 1].trainStation.name
    },
    openDetail(item) {
      this.router.push({name: 'Tab3', key: item.departureTimestamp, params: {data: JSON.stringify(item)}})
    }
  },
  setup() {
    const route = useRoute();
    const router = useRouter();
    console.log(JSON.parse(route.params.data))
    let results = JSON.parse(route.params.data)
    return {
      results,
      router
    }
  }
});
</script>

<style>
#container {
  text-align: center;
  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;
  color: #8c8c8c;
  margin: 0;
}

.time:before {
  content: "";
  width: 10px;
  height: 10px;
  border-radius: 5px;
  background: #c0c4c7;
  position: absolute;
  left: 3px;
  top: auto;
  display: block;
  margin: 4px 0 0 0;
}

.itemSt.active .time:before {
  background: #eb445a;
}

.itemSt:first-child:before {
  top: 0.5em;
}

.stations .itemSt:before {
  content: "";
  width: 2px;
  background: #eb445a;
  position: absolute;
  left: 7px;
  top: 0;
  bottom: 0;
  display: block;
  margin: 0;
  z-index: 1;
}

.stations .itemSt:first-child:before {
  top: 0.5em;
}

.itemSt.last:before {
  bottom: 10px;
}

.itemSt .time {
  width: 47px;
  float: left;
  margin-left: -64px;
  text-align: right;
  font-weight: bold;
}

.reset {
  border: none;
  background: none;
  margin: 0;
  padding: 0;
  width: 100%;
}

.itemSt .station {
  margin: 0;
}

.itemSt {
  padding: 0 0 2px 80px;
  clear: both;
  position: relative;
}

ion-item:hover {
  cursor: pointer;
}
</style>
