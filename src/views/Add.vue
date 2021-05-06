<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Ajouter une tache</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Ajouter une tache</ion-title>
        </ion-toolbar>
      </ion-header>

      <div class="container">
        <form @submit="handleSubmit">
          <ion-list>
            <ion-item>
              <ion-label position="floating">Intitule de la tache</ion-label>
              <ion-input v-model="formData.label" name="label"></ion-input>
            </ion-item>
            <ion-item>
              <ion-label position="floating">Choisir la date</ion-label>
              <ion-datetime
                v-model="formData.date"
                name="date"
                display-format="DD/MM/YYYY"
              ></ion-datetime>
            </ion-item>
          </ion-list>
          <ion-button expand="block" type="submit">Creer la tache</ion-button>
        </form>
        <div class="mt">
          <ion-button @click="gotoList" color="danger" expand="block" type="submit"
            >Retour a la liste</ion-button
          >
        </div>
      </div>
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
  IonList,
  IonItem,
  IonLabel,
  IonInput,
  IonDatetime,
  IonButton,
  toastController,
} from "@ionic/vue";
import { defineComponent } from "vue";
import { useRouter } from "vue-router";

// On import les Plugins... Ils sont disponibles par defaut
import { Plugins } from "@capacitor/core";
// Tu peux utiliser Plugins.Storage mais c'est soulant a ecrire donc, tu
// peux destructuer Plugins et recuperer Storage uniquement
const { Storage } = Plugins;

export default defineComponent({
  name: "Home",
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
    IonList,
    IonItem,
    IonLabel,
    IonInput,
    IonDatetime,
    IonButton,
  },
  data: function () {
    return {
      formData: {
        label: "",
        date: "",
      },
    };
  },
  methods: {
    // Comme il utilise du typescript plutot que du JS classique, il faut typer un peu les
    // variables. Mais si tu hesites sur le type, utilise any d'abord...
    handleSubmit: function (event: any) {
      event.preventDefault();
      if (this.formData.label.length === 0 || this.formData.date === "") {
        alert("Veuillez remplir tous les champs ...");
      } else {
        // On utilisera capacitor et native storage pour stocker les donnees ...
        // { ... }  est utilise pour creer une copie de l'objet pour eviter les problemes de references
        this.saveData("todos", { ...this.formData });
        this.openToast();
        this.formData = {
          label: "",
          date: "",
        };
      }
    },
    async openToast() {
      const toast = await toastController.create({
        message: "Tache ajoutee avec success.",
        duration: 3000,
      });
      return toast.present();
    },

    gotoList: function() {
        this.router.push('/home');
    }
  },
  setup() {
    // Async/await pour les donnees asynchrones
    const saveData = async (key: string, data: any) => {
      // On recupere le table
      const dataInDb: string | null = (await Storage.get({ key: "todos" }))
        .value;
      let todos;
      if (dataInDb) {
        todos = JSON.parse(dataInDb);
      } else {
        todos = [];
      }

      // On insere les donnees de cette facon
      // Mais comme la donnee est un objet, il faut la convertir en chaine de caractere

      // On ajoute la donnee au tableau
      todos.push(data);

      // On convertit todos en str
      const strData = JSON.stringify(todos);
      await Storage.set({
        key, // Totologie --- les deux sont equivalents
        value: strData,
      });
    };

    const router = useRouter();

    return {
      saveData,
      router
    };
  },
});
</script>

<style scoped>
.container {
  padding: 5%;
}

.mt {
  margin-top: 10px;
}
</style>