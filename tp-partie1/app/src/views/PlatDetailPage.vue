<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-back-button default-href="/tabs/menu" text="Menu" />
        </ion-buttons>
        <ion-title>Détail du plat</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <!-- 1. Chargement en cours -->
      <div v-if="chargement" class="ion-text-center ion-padding">
        <ion-spinner />
      </div>


      <div v-else-if="erreur" class="ion-padding">
        <ion-text color="danger">
          <p>{{ erreur }}</p>
        </ion-text>
        <ion-button expand="block" @click="charger">Réessayer</ion-button>
      </div>

      <!-- 3. Le plat -->
      <ion-card v-else-if="plat">
        <ion-card-header>
          <ion-card-subtitle>{{ plat.categorie?.nom ?? 'Sans catégorie' }}</ion-card-subtitle>
          <ion-card-title>{{ plat.nom }}</ion-card-title>
        </ion-card-header>

        <ion-card-content>
          <p>{{ plat.description }}</p>
          <p>
            <ion-text color="primary">
              <strong>{{ formatAriary(plat.prix) }}</strong>
            </ion-text>
            <ion-badge v-if="!plat.disponible" color="medium">Épuisé</ion-badge>
          </p>

          <ion-button expand="block" :disabled="!plat.disponible"
                      @click="ajouterAuPanier">
            <ion-icon slot="start" :icon="addCircleOutline" />
            Ajouter au panier
          </ion-button>
        </ion-card-content>
      </ion-card>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { useRoute } from 'vue-router';
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonButtons, IonBackButton,
  IonCard, IonCardHeader, IonCardSubtitle, IonCardTitle, IonCardContent,
  IonText, IonBadge, IonSpinner, IonButton, IonIcon,
} from '@ionic/vue';
import { addCircleOutline } from 'ionicons/icons';

import api from '@/services/api';
import { formatAriary } from '@/utils/format';
import { usePanier } from '@/composables/usePanier';
import { useIonRouter } from '@ionic/vue';

const route = useRoute();
const id = route.params.id;
const ionRouter = useIonRouter();
const { ajouter } = usePanier();

function ajouterAuPanier() {
  ajouter(plat.value);
  ionRouter.back();
}

const plat = ref(null);
const chargement = ref(true);
const erreur = ref(null);

async function charger() {
  chargement.value = true;
  erreur.value = null;

  try {
    plat.value = await api.getPlat(id);
  } catch (e) {
    if (e.statut === 404) {
      erreur.value = "Ce plat n'existe pas.";
    } else {
      erreur.value = "Impossible de joindre l'API. Est-elle lancée ?";
    }
    console.error(e);
  } finally {
    chargement.value = false;
  }
}

onMounted(charger);
</script>
