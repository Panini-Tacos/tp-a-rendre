<template>
  <ion-page>
    <ion-header>
      <ion-toolbar color="primary">
        <ion-title>Panier</ion-title>
        <ion-buttons slot="end">
          <ion-button v-if="lignes.length > 0" color="light" @click="vider">
            Vider
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <!-- Panier vide -->
      <div v-if="lignes.length === 0" class="ion-text-center ion-padding">
        <ion-icon :icon="cartOutline" size="large" color="medium" />
        <p>Votre panier est vide.</p>
        <ion-button router-link="/tabs/menu">Voir le menu</ion-button>
      </div>

      <!-- Panier rempli -->
      <template v-else>
        <ion-list>
          <ion-item v-for="ligne in lignes" :key="ligne.plat.id">
            <ion-label>
              <h2>{{ ligne.plat.nom }}</h2>
              <p>{{ formatAriary(ligne.plat.prix) }} × {{ ligne.quantite }}</p>
            </ion-label>

            <div slot="end" class="ion-display-flex ion-align-items-center">
              <ion-button fill="clear" color="medium"
                          @click="changerQuantite(ligne.plat.id, -1)">
                <ion-icon slot="icon-only" :icon="removeCircleOutline" />
              </ion-button>

              <ion-text><strong>{{ ligne.quantite }}</strong></ion-text>

              <ion-button fill="clear" color="primary"
                          @click="changerQuantite(ligne.plat.id, 1)">
                <ion-icon slot="icon-only" :icon="addCircleOutline" />
              </ion-button>
            </div>
          </ion-item>
        </ion-list>

        <ion-item lines="none">
          <ion-label><strong>Total</strong></ion-label>
          <ion-text slot="end" color="primary">
            <strong>{{ formatAriary(total) }}</strong>
          </ion-text>
        </ion-item>

        <!-- Formulaire d'envoi -->
        <ion-input v-model="table" type="number" label="Numéro de table"
                   label-placement="floating" fill="outline"
                   :class="{ 'ion-invalid ion-touched': erreurTable }"
                   :error-text="erreurTable"
                   @ion-input="erreurTable = ''" />

        <ion-button expand="block" class="ion-margin-top"
                    :disabled="envoi" @click="confirmer">
          Envoyer la commande
        </ion-button>
      </template>
    </ion-content>
  </ion-page>
</template>

<script setup>
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonButtons, IonButton,
  IonList, IonItem, IonLabel, IonText, IonIcon, IonInput,
  alertController, loadingController, toastController, useIonRouter,
} from '@ionic/vue';
import { cartOutline, addCircleOutline, removeCircleOutline } from 'ionicons/icons';
import { ref } from 'vue';
import api from '@/services/api';
import { formatAriary } from '@/utils/format';
import { usePanier } from '@/composables/usePanier';

const { lignes, changerQuantite, vider, total } = usePanier();
const ionRouter = useIonRouter();

const table = ref('');
const erreurTable = ref('');
const envoi = ref(false);

function tableValide() {
  const n = Number(table.value);
  if (table.value === '' || !Number.isInteger(n) || n < 1 || n > 20) {
    erreurTable.value = 'Le numéro de table doit être un entier entre 1 et 20.';
    return false;
  }
  erreurTable.value = '';
  return true;
}

async function confirmer() {
  if (!tableValide()) return;

  const alerte = await alertController.create({
    header: 'Confirmer la commande',
    message: `Table ${table.value} : ${formatAriary(total.value)}`,
    buttons: [
      { text: 'Annuler', role: 'cancel' },
      { text: 'Envoyer', role: 'confirm' },
    ],
  });
  await alerte.present();

  const { role } = await alerte.onDidDismiss();
  if (role === 'confirm') await envoyer();
}

async function envoyer() {

  const chargement = await loadingController.create({
    message: 'Envoi de la commande…',
  });
  await chargement.present();

  envoi.value = true;

  try {
    const commande = {
      table: Number(table.value),
      lignes: lignes.value.map((l) => ({
        plat_id: l.plat.id,
        quantite: l.quantite,
      })),
    };

    const creee = await api.creerCommande(commande);

    vider();
    table.value = '';
    erreurTable.value = '';

    const toast = await toastController.create({
      message: `Commande n°${creee.id} envoyée`,
      duration: 2000,
      position: 'top',
      color: 'success',
    });
    await toast.present();

    ionRouter.navigate('/tabs/commandes', 'forward', 'replace');
  } catch (e) {

    if (e.statut === 422 && e.details?.table) {
      erreurTable.value = e.details.table;
    } else {

      const toast = await toastController.create({
        message: e.message,
        duration: 2500,
        position: 'top',
        color: 'danger',
      });
      await toast.present();
    }
  } finally {

    envoi.value = false;
    await chargement.dismiss();
  }
}
</script>
