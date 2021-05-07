<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Creer un compte</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Creer un compte</ion-title>
        </ion-toolbar>
      </ion-header>

      <div class="container">
        <form @submit="handleSubmit">
          <ion-list>
            <ion-item>
              <ion-label position="floating">Email</ion-label>
              <ion-input
                @ion-input="onEmailChange"
                v-model="formData.email"
                name="email"
              ></ion-input>
            </ion-item>
            <ion-label class="error">{{ errorEmail }}</ion-label>
            <ion-item>
              <ion-label position="floating">Mot de passe</ion-label>
              <ion-input
                type="password"
                @ion-input="onPwdChange"
                v-model="formData.password"
                name="email"
              ></ion-input>
            </ion-item>
            <ion-label class="error">{{ errorPwd }}</ion-label>
          </ion-list>
          <ion-button expand="block" type="submit">S'enregistrer</ion-button>
        </form>
        <div class="mt">
          <ion-button
            @click="gotoList"
            color="danger"
            expand="block"
            type="submit"
            >Se connecter</ion-button
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
  IonButton,
  toastController,
} from "@ionic/vue";
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
    IonLabel,
    IonInput,
    IonButton,
  },
  data: function () {
    return {
      formData: {
        email: "",
        password: "",
      },
      errorEmail: "",
      errorPwd: "",
    };
  },
  methods: {
    // Comme il utilise du typescript plutot que du JS classique, il faut typer un peu les
    // variables. Mais si tu hesites sur le type, utilise any d'abord...
    handleSubmit: function (event: any) {
      event.preventDefault();
      const { email, password } = this.formData;
      if (email.length !== 0 && password.length !== 0) {
          this.addUser(email, password);
      } else {
          this.openToast('Veuillez remplir les champs !');
      }
    },
    onEmailChange: function (event: any) {
      const v = event.target.value;
      if (/.+@.+\..{2,8}/.test(v)) {
        this.errorEmail = "";
      } else {
        this.errorEmail = "Adresse email non valide !";
      }
    },
    onPwdChange: function (event: any) {
      const v = event.target.value;
      const test1 = /[a-z]+/.test(v);
      const test2 = /[A-Z]+/.test(v);
      const test3 = /.{8,}/.test(v);
      if (test1 && test2 && test3) {
        this.errorPwd = "";
      } else {
        this.errorPwd = "Mot de passe non valide !";
      }
    },
    async openToast(msg: string) {
      const toast = await toastController.create({
        message: msg,
        duration: 3000,
      });
      return toast.present();
    },
  },
  setup: function () {
    const addUser = async (email: string, pwd: string) => {
      const inDb = await Storage.get({ key: "users" });
      let users: any[];
      if (!inDb.value) {
        users = [];
      } else {
        users = JSON.parse(inDb.value as string);
      }
      console.log(inDb)
      users.push({ email, password: pwd });
      Storage.set({ 
          key: 'users',
          value: JSON.stringify(users)
      });
    };

    return {
        addUser
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

.error {
  font-size: 0.75rem;
  color: red;
}
</style>