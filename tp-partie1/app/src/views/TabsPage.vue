<template>
  <ion-page>
    <ion-tabs>
      <ion-router-outlet />

      <!-- La barre d'onglets, en bas de l'écran -->
      <ion-tab-bar slot="bottom">
        <ion-tab-button tab="menu" href="/tabs/menu">
          <ion-icon :icon="restaurantOutline" />
          <ion-label>Menu</ion-label>
        </ion-tab-button>

        <ion-tab-button tab="panier" href="/tabs/panier">
          <ion-icon :icon="cartOutline" />
          <ion-label>Panier</ion-label>
          <ion-badge v-if="nombreArticles > 0" color="danger">{{ nombreArticles }}</ion-badge>
        </ion-tab-button>

        <ion-tab-button tab="commandes" href="/tabs/commandes">
          <ion-icon :icon="receiptOutline" />
          <ion-label>Commandes</ion-label>
        </ion-tab-button>
      </ion-tab-bar>
    </ion-tabs>
  </ion-page>
</template>

<script setup>
import { IonPage, IonTabs, IonRouterOutlet, IonTabBar, IonTabButton, IonIcon, IonLabel, IonBadge } from '@ionic/vue';
import { restaurantOutline, cartOutline, receiptOutline } from 'ionicons/icons';
import { usePanier } from '@/composables/usePanier';

const { nombreArticles } = usePanier();
</script>
