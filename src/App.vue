<template>
  <div>

    <link rel="stylesheet"
      href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:opsz,wght,FILL,GRAD@24,400,0,0" />
    <navbar class="navbar-fixed" v-if="showNavbar" />

    <div id="app">
      <router-view />
    </div>
  </div>
</template>

<script>

import Navbar from './components/navbar_unten.vue'
import { initializeApp } from "firebase/app";
import { getMessaging, onMessage } from "firebase/messaging";
import { useStore } from 'vuex';

import { computed } from 'vue';
import { useRoute } from 'vue-router';



export default {
  components: {
    Navbar,

  },
  setup() {
    const store = useStore();







    /* 
        onBeforeUnmount(() => {
            console.log(userId)
          SocketService.disconnect(currentUser.id);
        }); */

    // eslint-disable-next-line
    const route = useRoute();

    const showNavbar = computed(() => store.state.showNavbar);
    const firebaseConfig = {
      apiKey: "AIzaSyDjimJJcGiL1O_6Swpzp2d5xaA5-AI4CpI",
      authDomain: "procon-14ef5.firebaseapp.com",
      projectId: "procon-14ef5",
      storageBucket: "procon-14ef5.appspot.com",
      messagingSenderId: "598092499566",
      appId: "1:598092499566:web:7bdfebff0c1dd13a2df47f",
      measurementId: "G-QX73B8G7V6"
    };

    // Initialize Firebase
    const app = initializeApp(firebaseConfig);// eslint-disable-line no-unused-vars



    // Get registration token. Initially this makes a network call, once retrieved
    // subsequent calls to getToken will return from cache.
    const messaging = getMessaging();

    onMessage(messaging, (payload) => {
      console.log('Message received. ', payload);
      const notificationObject = JSON.parse(payload.data.objekt);

      try {
        store.commit("ADD_NOTIFICATION_TO_USER", notificationObject);

      } catch (error) {
        console.error(error)
      }
      // ...
    });



    return {
      store,
      showNavbar,
    };
  }

} 
</script>

<style lang="scss">
.bookmark-page {
  padding-top: 10px;
}

body {
  margin: 0;
  padding: 0;
}

svg.svg-inline--fa.fa-arrow-left.fa-lg {
  height: 16px;
  width: 15px;
}

#app p,
span,
h5,
h4,
h3,
h2,
h1 {
  font-family: Arial, Helvetica, sans-serif;
}

.navbar-fixed {
  position: fixed;
  bottom: 0;
  height: 35px;
  width: 100%;
  z-index: 999;
  background-color: #ffffff;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-bottom: 5px;
  padding-top: 5px;
}

#app {
  font-family: Arial, Helvetica, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #000000;
  margin-bottom: 60px;

  /* Platz für die Navbar reservieren */
}

#app .topic-text {
  font-family: Arial, Helvetica, sans-serif;




}

#app .benachrichtigungTopic .topic-box {

  display: flex;
  flex-direction: column;
  padding: 0px;
  margin: 0px auto;
  background-color: #ffffff;
  border-radius: 10px;
  box-shadow: 0 0px 0px rgba(1, 18, 251, 0.1);
  max-width: 100%;
  transition: transform 0.3s ease-in-out;
}

#app .comment-text {
  font-family: Arial, Helvetica, sans-serif;
  font-size: 12px;
  margin-bottom: 0px;
  padding-left: 60px;
  padding-right: 10px;
  margin-top: 0%;
  text-align: left;
}



#app .conversation-prompt {
  display: flex;
  max-height: 1%;
  text-align: center;
  justify-content: center;
}

.comment-content {
  font-size: 14px;
  text-align: justify;
  word-break: break-word;
}

p#topictext {
  text-align: justify;
  font-size: 14px;
}

#antworttext {
  text-align: justify;
}

/* Restlicher CSS-Code bleibt unverändert */
/* ... */
</style>
