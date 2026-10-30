<template>
  <ion-page>
    <ion-header>
      <ion-toolbar color="primary">
        <ion-title>Commandes</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content>
      <ion-refresher slot="fixed" @ion-refresh="rafraichir">
        <ion-refresher-content />
      </ion-refresher>

      <!-- 1. Chargement -->
      <div v-if="chargement" class="ion-text-center ion-padding">
        <ion-spinner />
      </div>

      <!-- 2. Erreur -->
      <div v-else-if="erreur" class="ion-padding">
        <ion-text color="danger"><p>{{ erreur }}</p></ion-text>
        <ion-button expand="block" @click="charger">Réessayer</ion-button>
      </div>

      <!-- 3. Aucune commande -->
      <div v-else-if="commandes.length === 0" class="ion-text-center ion-padding">
        <ion-icon :icon="receiptOutline" size="large" color="medium" />
        <p>Aucune commande pour le moment.</p>
        <ion-button router-link="/tabs/menu">Voir le menu</ion-button>
      </div>

      <!-- 4. Liste des commandes -->
      <ion-list v-else>
        <ion-card v-for="c in commandes" :key="c.id">
          <ion-card-header>
            <ion-card-subtitle>
              Commande n°{{ c.id }} · Table {{ c.table }}
            </ion-card-subtitle>

            <ion-card-title>{{ formatAriary(c.total) }}</ion-card-title>

            <ion-badge :color="couleurStatut(c.statut)">
              {{ libelleStatut(c.statut) }}
            </ion-badge>
          </ion-card-header>

          <ion-card-content>
            <p>
              <ion-icon :icon="timeOutline" /> {{ formatDate(c.date) }}
            </p>
            <p>{{ resumeLignes(c.lignes) }}</p>
          </ion-card-content>
        </ion-card>
      </ion-list>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { ref } from 'vue';
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonRefresher,
  IonRefresherContent, IonSpinner, IonText, IonButton, IonList, IonCard,
  IonCardHeader, IonCardSubtitle, IonCardTitle, IonCardContent, IonBadge, IonIcon,
  onIonViewWillEnter,
} from '@ionic/vue';
import { receiptOutline, timeOutline } from 'ionicons/icons';
import api from '@/services/api';
import { formatAriary, formatDate } from '@/utils/format';

const commandes = ref([]);
const chargement = ref(true);
const erreur = ref(null);

async function charger() {
  chargement.value = true;
  erreur.value = null;

  try {
    commandes.value = await api.getCommandes();
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

onIonViewWillEnter(charger);

function resumeLignes(lignes) {
  return lignes.map((l) => `${l.quantite} × ${l.nom}`).join(', ');
}

const STATUTS = {
  en_attente: { libelle: 'En attente', couleur: 'warning' },
  en_preparation: { libelle: 'En préparation', couleur: 'primary' },
  prete: { libelle: 'Prête', couleur: 'success' },
  servie: { libelle: 'Servie', couleur: 'medium' },
  annulee: { libelle: 'Annulée', couleur: 'danger' },
};

function libelleStatut(statut) {
  return STATUTS[statut]?.libelle ?? statut;
}

function couleurStatut(statut) {
  return STATUTS[statut]?.couleur ?? 'medium';
}
</script>
