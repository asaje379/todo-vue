<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Todos</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Todos</ion-title>
        </ion-toolbar>
      </ion-header>

      <div class="container">
        <ion-list>
          <ion-item v-for="todo in todos" :key="todo.label">
            <ion-label>{{ todo.date }}</ion-label>
            <ion-label>{{ todo.label }}</ion-label>
          </ion-item>
        </ion-list>
      </div>

      <!-- eslint-disable-next-line -->
      <ion-fab vertical="bottom" horizontal="end" slot="fixed">
        <ion-fab-button router-link="/add">
          <ion-icon :icon="add" />
        </ion-fab-button>
      </ion-fab>
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
  IonFab,
  IonFabButton,
  IonIcon,
} from "@ionic/vue";
import { add } from "ionicons/icons";
import { defineComponent } from "vue";
import { Plugins } from "@capacitor/core";

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
    IonFab,
    IonFabButton,
    IonIcon,
  },
  data: function () {
    return {
      todos: [],
    };
  },
  // La function mounted est appele au chargement du composant
  // C'est donc a ce moment que j'appelle la fonction getData qui ne me retourne
  // pas les donnees immediatement. Donc, une fois que je suis sur que les donnees
  // sont la je les passe a ma variable 'todos'
  mounted: function() {
    this.getData().then(values => {
      // La methode map applique un algorithme a tous les elements d'un tableau.
      // L'algorithme transforme alos chaque element du tableau... tu peux lire la doc
      this.todos = values.map((x: any) => {
        return {
          label: x.label,
          date: new Date(x.date).toLocaleDateString()
        }
      })
    })
  },
  setup() {
    const getData = async function() {
      console.log('Getting data')
      const dataInDb: any = await Storage.get({ key: "todos" });
      const todos = JSON.parse(dataInDb.value);
      return todos;
    };

    return {
      add,
      getData
    };
  },
});
</script>

<style scoped>
</style>