<template>
  <div id="container">
    <strong>{{ name }}</strong>
    <ion-item class="ion-margin-top">
      <ion-label>Odkiaľ</ion-label>
      <ion-input placeholder="Odkiaľ"
                 @ionBlur="onBlurFrom"
                 @ionFocus="onFocusFrom"
                 v-model="from"
                 @ionChange="onChangeFrom"></ion-input>
    </ion-item>
    <ion-list
        v-if="isOpenFrom"
        v-show="isOpenFrom">
      <ion-item
          v-for="(resultF) in resultsFrom"
          :key="resultF.uicCode"
          @click="setResultFrom(resultF)"
          class="autocomplete-result">
        {{ resultF.name }}
      </ion-item>
    </ion-list>
    <ion-item>
      <ion-label>Kam</ion-label>
      <ion-input placeholder="Odkiaľ"
                 @ionBlur="onBlurTo"
                 @ionFocus="onFocusTo"
                 v-model="to"
                 @ionChange="onChangeTo"></ion-input>
    </ion-item>
    <ion-list
        v-if="isOpenTo"
        v-show="isOpenTo">
      <ion-item
          v-for="(result) in resultsTo"
          :key="result.uicCode"
          @click="setResultTo(result)"
          class="autocomplete-result">
        {{ result.name }}
      </ion-item>
    </ion-list>
    <ion-item @click="setOpen(true)">
      <ion-label>Dátum a čas</ion-label>
      <ion-input v-model="date1"></ion-input>
      <ion-modal :is-open="isOpenRef"
                 css-class="my-custom-class"
                 @didDismiss="setOpen(false)">
        <ion-header>
          <ion-toolbar>
            <ion-title>Zadajte dátum a čas</ion-title>
          </ion-toolbar>
        </ion-header>
        <ion-page>
          <ion-datetime ref="customDatetime" style="margin: 55px auto auto auto;" first-day-of-week="1"
                        @ionChange="(ev) => date1 = formatDate(ev.detail.value)">
            <ion-buttons slot="buttons">
              <ion-button @click="confirm()">Uložiť</ion-button>
              <ion-button @click="reset()">Resetovať</ion-button>
            </ion-buttons>
          </ion-datetime>
        </ion-page>
      </ion-modal>
    </ion-item>
    <ion-button @click="searchResults">Hľadať</ion-button>
  </div>
</template>

<script>
import {defineComponent, ref} from 'vue';
import {
  IonInput,
  IonDatetime,
  IonLabel,
  IonItem,
  IonModal,
  IonPage,
  IonButton,
  IonButtons,
  IonTitle,
  IonHeader,
  IonToolbar,
  IonList,
  loadingController, useBackButton, alertController
} from '@ionic/vue';
import {format, parseISO} from 'date-fns';
import '@capacitor-community/http';
import {Plugins} from '@capacitor/core';
const { App } = Plugins;
import { useRouter } from 'vue-router';

const {Http} = Plugins;

export default defineComponent({
  name: 'SearchForm',
  components: {
    IonInput,
    IonDatetime,
    IonLabel,
    IonItem,
    IonModal,
    IonPage,
    IonButton,
    IonButtons,
    IonTitle,
    IonHeader,
    IonToolbar,
    IonList
  },
  props: {
    name: String
  },

  data() {
    return {
      from: '',
      fromUic: 0,
      to: '',
      toUic: 0,
      resultsFrom: [],
      resultsTo: [],
      isOpenFrom: false,
      isOpenTo: false,
      isFocusedFrom: false,
      isFocusedTo: false,
      dateTimestamp: new Date().getTime(),
    };
  },
  methods: {
    setResultFrom(result) {
      this.from = result.name;
      this.fromUic = result.uicCode;
      this.isOpenFrom = false;
    },
    async onBlurFrom() {
      this.isFocusedFrom = false
      await new Promise(resolve => setTimeout(resolve, 200));
      this.isOpenFrom = false
    },
    onFocusFrom() {
      this.isFocusedFrom = true
      if (this.from.length > 2) {
        this.isOpenFrom = true
      }
    },
    async onChangeFrom() {
      this.isOpenFrom = true;
      if (this.isFocusedFrom) {
        if (this.from.length > 2) {
          await Http.request({
            method: 'GET',
            url: 'https://app.zssk.sk/api/v1/station/name/' + this.from,
            headers: {
              'x-api-key': 'PDh^2-$-M]8(dG8E+Q,FR}zsfz"Q~:N2pp\\ykmg9ZEgKVrh42PHS?^sQ6<3;X,?-',
              'Access-Control-Allow-Origin': '*'
            }
          })
              .then(({data}) => {
                this.fromUic = 0
                this.resultsFrom = data
              })
        } else {
          this.resultsFrom = []
        }
      }
    },
    setResultTo(result) {
      this.to = result.name;
      this.toUic = result.uicCode;
      this.isOpenTo = false;
    },
    async onBlurTo() {
      this.isFocusedTo = false
      await new Promise(resolve => setTimeout(resolve, 200));
      this.isOpenTo = false
    },
    onFocusTo() {
      this.isFocusedTo = true
      if (this.to.length > 2) {
        this.isOpenTo = true
      }
    },
    async onChangeTo() {
      this.isOpenTo = true;
      if (this.isFocusedTo) {
        if (this.to.length > 2) {
          await Http.request({
            method: 'GET',
            url: 'https://app.zssk.sk/api/v1/station/name/' + this.to,
            headers: {
              'x-api-key': 'PDh^2-$-M]8(dG8E+Q,FR}zsfz"Q~:N2pp\\ykmg9ZEgKVrh42PHS?^sQ6<3;X,?-',
              'Access-Control-Allow-Origin': '*'
            }
          }).then(({data}) => {
            this.toUic = 0
            this.resultsTo = data
          })
        } else {
          this.resultsTo = []
        }
      }
    },

    getParent(name) {
      let p = this.$parent;
      while(typeof p !== 'undefined') {
        if (p.$options.name === name) {
          return p;
        } else {
          p = p.$parent;
        }
      }
      return false;
    },
    formatDate(value) {
      this.dateTimestamp = new Date(value);
      this.dateTimestamp = this.dateTimestamp.getTime();
      return format(parseISO(value), 'dd. MMM yyyy, HH:mm');
    },
    async searchResults() {
      const loading = await loadingController
          .create({
            message: 'Vyhľadávam spojenia...',
            duration: this.timeout,
          });

      if (this.fromUic === 0) {
        if (this.resultsFrom.length > 0) {
          this.fromUic = this.resultsFrom[0].uicCode
        }
      }
      if (this.toUic === 0) {
        if (this.resultsTo.length > 0) {
          this.toUic = this.resultsTo[0].uicCode
        }
      }
      if (this.toUic === 0 || this.fromUic === 0) {
        const alert = await alertController.create({
          header: 'Chyba vo vyhľadávaní',
          message: 'Skontrolujte, či ste zadali všetky potrebné údaje',
          buttons: ['Ok'],
        });

        await alert.present();
        return;
      }
      await loading.present();
      await Http.request({
        method: 'GET',
        url: 'https://app.zssk.sk/api/v1/route?fromStation=' + this.fromUic + '&toStation=' + this.toUic + '&departure=true&ageCategory=102&ageCategoryDiscount=600&travelDate=' + this.dateTimestamp,
        headers: {
          'x-api-key': 'PDh^2-$-M]8(dG8E+Q,FR}zsfz"Q~:N2pp\\ykmg9ZEgKVrh42PHS?^sQ6<3;X,?-',
          'Access-Control-Allow-Origin': '*'
        }
      }).then(({data}) => {
        this.router.push({name: 'Tab2', params:{data: JSON.stringify(data)}})
      })
      await loading.dismiss();
    }
  },

  setup() {
    const router = useRouter();
    const customDatetime = ref();
    const date1 = format(new Date(), 'dd. MMM yyyy, HH:mm');

    const confirm = () => {
      if (customDatetime.value === undefined) return;

      customDatetime.value.$el.confirm();
      setOpen(false)
    };

    const reset = () => {
      if (customDatetime.value === undefined) return;

      customDatetime.value.$el.reset();
    };

    const isOpenRef = ref(false);
    const setOpen = (state) => isOpenRef.value = state;
    useBackButton(-1, () => {
      App.exitApp();
    });

    return {
      customDatetime,
      confirm,
      reset,
      date1,
      isOpenRef,
      setOpen,
      router
    }
  }
});

</script>

<style scoped>
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

#container a {
  text-decoration: none;
}

ion-list {
  position: absolute;
  width: inherit;
  overflow-y: hidden;
  max-height: 150%;
  z-index: 999;
  min-width: 300px;
  border: 1px solid rgb(34, 34, 34);
}

ion-item img {
  max-height: 2.5rem;
  width: auto;
  margin: auto;
  display: block;
}

ion-item:hover {
  cursor: pointer;
}
</style>
