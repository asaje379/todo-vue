<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Se connecter</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Se connecter</ion-title>
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
          <ion-button expand="block" type="submit">Se connecter</ion-button>
        </form>
        <div class="mt">
          <ion-button
            @click="gotoList"
            color="danger"
            expand="block"
            type="submit"
            >Creer un compte</ion-button
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
    handleSubmit: async function (event: any) {
      event.preventDefault();
      const { email, password } = this.formData;
      if (email.length !== 0 && password.length !== 0) {
        const result = await this.login(email, password);
        let msg = "";
        switch (result) {
          case -1:
            msg = "Vous devez creer un compte avant de vous connecter !";
            break;
          case -2:
            msg = "Identifiants non valide !";
            break;
          case 0:
            msg = "Felicitations !";
            break;
        }
        this.openToast(msg);
      } else {
        this.openToast("Veuillez remplir les champs !");
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
    // -1: doit creer un compte
    // -2: echec de l'authentification
    const login = async (email: string, pwd: string) => {
      const inDB = await Storage.get({ key: "users" });
      if (!inDB.value) {
        return -1;
      } else {
        const users = JSON.parse(inDB.value as string);
        let success = false;
        for (const el of users) {
          if (el.email === email && el.password === pwd) {
            success = true;
            break;
          }
        }
        return !success ? -2 : 0;
      }
    };

    return {
      login,
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