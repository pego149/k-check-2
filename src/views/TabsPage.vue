<template>
  <ion-page>
    <ion-tabs>
      <ion-router-outlet></ion-router-outlet>
      <ion-tab-bar slot="bottom">
        <ion-tab-button :disabled="true" tab="tab1" href="/tabs/tab1" @click="tab2disabled=true;tab3disabled=true">
          <ion-icon :icon="searchOutline"/>
          <ion-label>Vyhľadávanie spojenia</ion-label>
        </ion-tab-button>

        <ion-tab-button :disabled="tab2disabled" tab="tab2" href="/tabs/tab2" @click="tab3disabled=true">
          <ion-icon :icon="analyticsOutline"/>
          <ion-label>Výsledky hľadania</ion-label>
        </ion-tab-button>

        <ion-tab-button :disabled="tab3disabled" tab="tab3" href="/tabs/tab3">
          <ion-icon :icon="trainOutline"/>
          <ion-label>Detail</ion-label>
        </ion-tab-button>
      </ion-tab-bar>
    </ion-tabs>
  </ion-page>
</template>

<script>
import {defineComponent} from 'vue';
import {IonTabBar, IonTabButton, IonTabs, IonLabel, IonIcon, IonPage, IonRouterOutlet} from '@ionic/vue';
import {searchOutline, analyticsOutline, trainOutline} from 'ionicons/icons';


export default defineComponent({
  name: 'TabsPage',
  components: {IonLabel, IonTabs, IonTabBar, IonTabButton, IonIcon, IonPage, IonRouterOutlet},
  setup() {
    return {
      searchOutline,
      analyticsOutline,
      trainOutline,
    }
  },
  data() {
    return {
      tab2disabled: true,
      tab3disabled: true
    }
  }
});
</script>
