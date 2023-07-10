<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>KCheck2</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">KCheck2</ion-title>
        </ion-toolbar>
      </ion-header>
    
      <SearchForm name="Hľadajte spojenie" />
    </ion-content>
  </ion-page>
</template>

<script>
import { defineComponent } from 'vue';
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent } from '@ionic/vue';
import SearchForm from '@/components/SearchForm.vue';

export default  defineComponent({
  name: 'Tab1Page',
  components: { SearchForm, IonHeader, IonToolbar, IonTitle, IonContent, IonPage }
});
</script>
