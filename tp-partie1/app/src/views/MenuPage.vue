<template>
  <!-- ion-page : l'enveloppe obligatoire de chaque page -->
  <ion-page>
    <!-- En-tête fixe en haut de l'écran -->
    <ion-header>
      <ion-toolbar color="primary">
        <ion-title>Menu du jour</ion-title>
      </ion-toolbar>
    </ion-header>

    <!-- Zone qui défile -->
    <ion-content>

      <ion-refresher slot="fixed" @ion-refresh="rafraichir">
        <ion-refresher-content />
      </ion-refresher>

        <!-- menu categories -->
      <ion-segment v-model="categorieChoisie" scrollable @ion-change="charger">
        <ion-segment-button value="toutes"><ion-label>Tout</ion-label></ion-segment-button>
        <ion-segment-button v-for="c in categories" :key="c.id" :value="String(c.id)">
          <ion-label>{{ c.nom }}</ion-label>
        </ion-segment-button>
      </ion-segment>

      <!-- recherche -->
      <ion-toolbar>
        <ion-searchbar v-model="recherche" placeholder="Rechercher un plat" :debounce="250" />
      </ion-toolbar>
      
      <!-- 1. Chargement en cours -->
      <div v-if="chargement" class="ion-text-center ion-padding">
        <ion-spinner />
      </div>

      <!-- 2. Erreur -->
      <div v-else-if="erreur" class="ion-padding">
        <ion-text color="danger">
          <p>{{ erreur }}</p>
        </ion-text>
        <ion-button expand="block" @click="charger">Réessayer</ion-button>
      </div>

      <!-- 3. La liste des plats -->
      <ion-list v-else>
        <ion-item v-if="platsCategorieFiltres.length == 0">
            <ion-label>Aucun plats correspendants</ion-label>
        </ion-item>
        <ion-item v-for="plat in platsCategorieFiltres" :key="plat.id" :router-link="`/tabs/menu/${plat.id}`">
          <ion-label>
            <h2>{{ plat.nom }}</h2>
            <p>{{ plat.categorie.nom }}</p>
            <p>{{plat.description}}</p>
          </ion-label>
          <ion-text color="primary">{{ formatAriary(plat.prix) }}</ion-text>
          <ion-badge v-if="!plat.disponible" color="medium" slot="end">Épuisé</ion-badge>
          <ion-button slot="end" fill="clear" :disabled="!plat.disponible"
                      @click.stop.prevent="ajouterAuPanier(plat)">
            <ion-icon slot="icon-only" :icon="addCircleOutline" />
          </ion-button>
        </ion-item>
      </ion-list>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent,IonSegment,IonSegmentButton,IonRefresher,IonRefresherContent,
  IonList, IonItem, IonLabel, IonSpinner, IonText, IonButton, IonBadge,IonSearchbar, IonIcon,
  toastController,
} from '@ionic/vue';
import { addCircleOutline } from 'ionicons/icons';
import api from '@/services/api';
import { formatAriary } from '@/utils/format';
import { usePanier } from '@/composables/usePanier';

const { ajouter } = usePanier();

async function ajouterAuPanier(plat) {
  ajouter(plat);

  const toast = await toastController.create({
    message: `${plat.nom} ajouté au panier`,
    duration: 1500,
    position: 'top',
    color: 'secondary',
  });
  
  await toast.present();
}

const plats = ref([]);
const categories = ref([]);
const categorieChoisie = ref('toutes');
const chargement = ref(true);
const erreur = ref(null);
const recherche = ref('');
const platsFiltres = computed(() => plats.value.filter((p) => p.nom.toLowerCase().includes(recherche.value.toLowerCase())));
const platsCategorieFiltres = computed(() => categorieChoisie.value === 'toutes' ? platsFiltres.value : platsFiltres.value.filter(p => p.categorie_id == categorieChoisie.value));

async function charger() {
  chargement.value = true;
  erreur.value = null;

  try {
    plats.value = await api.getPlats();
    categories.value = await api.getCategories();
  } catch (e) {
    erreur.value = "Impossible de joindre l'API. Est-elle lancée ?";
    console.error(e);
  } finally {
    chargement.value = false;
  }
}

async function rafraichir(event) {
  await charger();
  event.target.complete(); // arrête l'animation
}

onMounted(charger);
</script>
