
<template>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <div v-if="topic" class="topic-box" :style="{ border: getBorderColor() }">
    <div class="notificantionanzeige" v-if="!disableelementsforNotifications">
      <div class="author-info">
        <div class="left-content">

          <h3>{{ topic.title }}</h3>


        </div>

        <!--Buttons oben rechts-->
        <div class="right-content">
          <p>{{ $store.getters.formattedCreatedAt(topic.createdAt) }}</p>
          <button class="share-button" @click="shareContent">
            <font-awesome-icon :icon="['fas', 'share-nodes']" class="icon"
              :style="{ color: iconColor(currentUser.farbe) }" />
          </button>
          <!--Save Button-->
          <button @click="saveChanges" class="savebutton">

            <font-awesome-icon :icon="isSaved(topic.path, currentUser.id) ? ['fas', 'bookmark'] : ['far', 'bookmark']"
              class="icon" :style="{ color: iconColor(currentUser.farbe) }" />
          </button>


        </div>

      </div>
      <div class="kategorie">
        <p style=" font-size: 11px;" class="topic-category">{{ ' ' + topic.category.sub }}</p>
      </div>

    </div>

    <div class="topic-content" @click="goToTopic">

      <p class="topic-text " id="topictext">{{ trimmedTopicText }}</p>
    </div>

    <!-- hier fängt der untere teil unterhalb des textes an -->
    <div class="notificantionanzeige" v-if="!disableelementsforNotifications">
      <div class="author-info">

        <div class="left-content">
          <div class="profilbild" @click="goToProfile(topic.author)">
            <img :src="author.profileImage" alt="Author Profile Image" class="author-image" />
          </div>
          <div class="author">

          </div>
          <div class="author-category">
            <span class="author-name" style="line-height: 0.8;">{{ author.name }} </span>


          </div>

        </div>


      </div>

      <div class="balken">
        <div class="like-bar" :class="{ liked: hasLikedTopic || hasDislikedTopic }"
          :style="{ width: upvotePercentage + '%' }" :title="upvotePercentage + '%'" :interactive="true">
          <p v-if="hasLikedTopic || hasDislikedTopic" class="bar-text">{{ upvotePercentage + '%' }}</p>
        </div>
        <div class="dislike-bar" :class="{ disliked: hasLikedTopic || hasDislikedTopic }"
          :style="{ width: downvotePercentage + '%' }" :title="downvotePercentage + '%'" :interactive="true">
          <p v-if="hasLikedTopic || hasDislikedTopic" class="bar-text">{{ downvotePercentage + '%' }}</p>
        </div>
      </div>


      <!--Like Button-->
      <div class="interaction-bar">

        <div class="vote">

          <button @click="like" class="like-button">
            <font-awesome-icon :icon="hasLikedTopic ? ['fas', 'thumbs-up'] : ['far', 'thumbs-up']" class="icon"
              :style="{ color: iconColor(currentUser.farbe) }" />
            <p :style="{ color: iconColor(currentUser.farbe) }">{{ topic.upvotes }}</p>
          </button>
          <button @click="dislike" class="like-button">
            <font-awesome-icon :icon="hasDislikedTopic ? ['fas', 'thumbs-down'] : ['far', 'thumbs-down']" class="icon"
              :style="{ color: iconColor(currentUser.farbe) }" />
            <p :style="{ color: iconColor(currentUser.farbe) }">{{ topic.downvotes }}</p>
          </button>



        </div>

        <!--ConPro Button-->
        <div v-if="!disableelements" class="disableFromBookmark">
          <div class="tab-selection">
            <button @click="updateTabAndColor('pro')" :class="{ 'active-tab': selectedTab === 'pro' }">Pro</button>
            <button @click="updateTabAndColor('contra')"
              :class="{ 'active-tab': selectedTab === 'contra' }">Contra</button>

          </div>


          <div v-if="selectedTab === 'pro'" class="kommentare">
            <CommentBox v-for="comment in sortedComments('pro').slice(0, 2)" :key="comment.id" :comment="comment"
              :topic="id" />
          </div>

          <div v-else-if="selectedTab === 'contra'" class="kommentare">
            <CommentBox v-for="comment in sortedComments('contra').slice(0, 2)" :key="comment.id" :comment="comment"
              :topic="id" />
          </div>
        </div>
        <!-- Anzeige, wenn keine Kommentare vorhanden sind -->
        <div v-else>
          <p>Noch keine Kommentare vorhanden.</p>
        </div>



        <!--Konversation Button-->
        <div class="conversation-prompt">
          <button @click="goToTopic" :style="{ color: iconColor(currentUser.farbe) }" class="join-button"><span>Show more
              <font-awesome-icon :icon="['far', 'comments']" class="icon" @click="goToTopic" /> </span>
          </button>

        </div>



      </div>


    </div>
  </div>

  <!---->














  <div v-else>
    <!-- Placeholder content while topic is loading, or error message if topic couldn't be loaded -->
    <p>Loading topic...</p>
  </div>
</template>

<script>
import { mapGetters, mapMutations, mapActions } from 'vuex';
import { iconColor } from './farben';
import { useStore } from 'vuex'; // Importiere das useStore-Hook
import { computed, watchEffect } from 'vue';
import { onMounted } from 'vue';
import CommentBox from './CommentBox';
import { useRouter } from 'vue-router';



export default {
  components: {
    CommentBox,

  },

  setup(props) {



    const store = useStore(); // Erhalte Zugriff auf den Vuex-Store
    const router = useRouter();
    const topic = computed(() => store.getters.getTopicById(props.id));
    // Zugriff auf den currentUser aus dem Vuex-Store

    const author = store.getters.getUserById(topic.value.author);
    console.log(topic.value.author)
    const currentUser = computed(() => store.state.currentUser);

    const selectedTab = computed(() => store.state.selectedTab);
    const selectedTabColor = computed(() => store.state.selectedTabColor);
    const trimmedTopicText = computed(() => {
      if (props.disableelementsforNotifications && topic.value.text.length > 96) {
        return topic.value.text.substring(0, 96) + '...';
      }
      return topic.value.text;
    });

    watchEffect(() => {
      document.documentElement.style.setProperty('--selectedTabColor', selectedTabColor.value);
    });

    const isBarTooSmall = computed(() => {
      const minWidthThreshold = 10;
      return (
        props.topic.upvotePercentage < minWidthThreshold ||
        props.topic.downvotePercentage < minWidthThreshold
      );
    });






    const goToProfile = () => {
      console.log("klickt")
      console.log(currentUser.value)
      console.log()
      if (currentUser.value.id == topic.value.author) {
        router.push(`/profil/${topic.value.author}`);
      }
      else {
        router.push(`/profile/${topic.value.author}`);
      }

    }





    return {
      iconColor,
      currentUser, // Mache den currentUser verfügbar
      topicUrl: '',
      selectedTab,
      selectedTabColor,
      isBarTooSmall,
      onMounted,
      goToProfile,
      topic,
      author,
      trimmedTopicText,
    };
  },

  props: {
    id: {
      type: String,
      required: true,
    },
    disableelements: {

    },
    isDownVoted: {

    },
    isUpVoted: {

    },
    disableelementsforNotifications: {
      default: false,
    }
  },
  computed: {
    ...mapGetters(['getTopicById']),


    hasLikedTopic() {
      return this.currentUser.haslikedtopic.includes(this.topic.id);
    },
    hasDislikedTopic() {
      return this.currentUser.hasdislikedtopic.includes(this.topic.id);
    },
    sortedComments() {
      return (type) => {
        const commentType = type === 'pro' ? 'proComments' : 'contraComments';
        return this.topic[commentType]
          .slice()
          .sort((a, b) => b.upvotes - a.upvotes);
      };
    },

    upvotePercentage() {
      const totalVotes = this.topic.upvotes + this.topic.downvotes;
      return totalVotes === 0 ? 0 : Math.round((this.topic.upvotes / totalVotes) * 100);
    },
    downvotePercentage() {
      return 100 - this.upvotePercentage;
    },



  },
  data() {
    return {
      popupGroup: null,
    };
  },


  methods: {
    ...mapMutations(['TOGGLE_LIKE', 'TOGGLE_DISLIKE', 'ADD_TOPIC_TO_SAVES',]), // Import mutations
    ...mapActions(['fetchComments', 'addCommentToTopic', 'selectTab',]),


    getBorderColor() {
      if (this.isUpVoted) {
        return '1.5px solid green'; // Setze den Rand auf Grün, wenn isUpVoted true ist
      } else if (this.isDownVoted) {
        return '1.5px solid red'; // Setze den Rand auf Rot, wenn isDownVoted true ist
      } else {
        return 'transparent'; // Setze den Rand auf transparent, wenn weder isUpVoted noch isDownVoted true sind
      }
    },

    updateTabAndColor(tab) {

      this.$store.dispatch('selectTab', tab); // Action aufrufen
      this.$store.dispatch('updateSelectedTabColor'); // Action aufrufen
    },



    //Save button logik
    saveChanges() {
      console.log("kolleg")
      const path = this.topic.path;
      const currentUserId = this.currentUser.id;


      this.$store.dispatch('addtopicToSaves', { path, currentUserId })


    },
    isSaved(topicId,) {
      return this.$store.getters.isTopicSaved(topicId);
    },

    // Funktion zur Berechnung des Prozentsatzes
    shareContent() {
      console.log(this.topic.title)
      const shareData = {
        title: this.topic.title,
        text: 'Check out this interesting topic!\n',
        url: this.getTopicUrl(),
      };

      if (navigator.share) {
        navigator.share(shareData)
          .then(() => {
            console.log('Inhalt erfolgreich geteilt.');
          })
          .catch((error) => {

            console.error('Fehler beim Teilen:', error);
          });
      } else {
        console.warn('Der Browser unterstützt den "Native Share" nicht.');
      }
    },





    getTopicUrl() {
      return this.$router.resolve(`/topic/${this.id}`).href;
    },


    animateButton(target) {
      target.animate(
        [
          // keyframes
          { transform: 'scale(1)' },
          { transform: 'scale(1.3)' },
          { transform: 'scale(1)' }
        ],
        {
          // timing options
          duration: 400,
          easing: 'ease-in-out'
        }
      );
    },

    dislike(event) {
      const userId = this.currentUser.id;
      this.$store.dispatch('toggleDislikeAction', { topicId: this.id, userId, notificationType: "topicdislike", zielId: this.topic.author, benachrichtigungsElementId: this.id });
      const dislikeButton = event.target;
      this.animateButton(dislikeButton);
    },


    like(event) {
      const userId = this.currentUser.id;
      console.log("Ziel ID " + this.topic.author)
      this.$store.dispatch('toggleLikeAction', { topicId: this.id, userId, notificationType: "topiclike", zielId: this.topic.author, benachrichtigungsElementId: this.id });
      const likeButton = event.target;
      this.animateButton(likeButton);
    },

    goToTopic(event) {
      const targetElement = event.target;

      // Überprüfen, ob auf das Bild oder die Box geklickt wurde
      if (
        targetElement.closest('.topic-image') ||
        targetElement.closest('.topic-content') ||
        targetElement.closest('.topic-text') ||
        targetElement.closest('.join-button') ||
        targetElement.closest('.interaction-bar')
      ) {
        this.$router.push(`/topic/${this.id}`);
      }
    },

    // Vuex-Mutation zum Aktualisieren der Likes aufrufen

    showPopup(group) {
      this.popupGroup = group;
    },
    ...mapMutations(['UPDATE_LIKES']), // Füge die Mutation-Map hinzu, um die Vuex-Mutation aufzurufen
  },


};
</script>



<style lang="scss" scoped >
.right-content {
  max-height: 42px;
}

.topic-category {
  margin: 0px;
  padding-bottom: 5px;
  border-bottom: 1px solid #b0b0b0;
  text-align: left;
}

h3 {
  margin: 0px;
  padding-top: 10px;
  padding-bottom: 10px;
  overflow-wrap: break-word;
  word-wrap: break-word;
}

.tab-selection {
  display: flex;
  justify-content: space-evenly;
  background-color: white;


  button {
    padding: 10px 20px;
    font-size: 16px;
    color: #333;
    background-color: transparent;
    border: none;
    cursor: pointer;
    transition: all 0.3s ease;
    position: relative;

    &:before {
      content: "";
      position: absolute;
      bottom: 0;
      left: 0;
      width: 100%;
      height: 3px;
      background-color: transparent;
      transition: background-color 0.3s ease;
    }





    &.active-tab {
      color: var(--selectedTabColor);
      font-weight: bold;

      &:before {
        background-color: var(--selectedTabColor);
      }
    }
  }
}



.tab-content-enter-active,
.tab-content-leave-active {
  transition: opacity 0.5s ease-in-out;
}

.tab-content-enter,
.tab-content-leave-to

/* .tab-content-leave-active in <2.1.8 */
  {
  opacity: 0;
}

.savebutton {
  background-color: transparent;
}

.right-content {
  display: flex;
  flex-direction: row;
  min-width: 30%;
  justify-content: flex-end;


}

.author-category {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  padding-left: 1em;

}



.save-button {
  background-color: transparent;
}


.bar-text {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  margin: 0;
  color: white;
}



.like-bar,
.dislike-bar {
  height: 10px;
  transition: width 0.3s ease-in-out;

}

.balken {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  margin-top: 10px;
  width: 100%;
  height: 20px;

  .like-bar,
  .dislike-bar {
    height: 100%;
    width: 50%;
    transition: background-color 0.3s ease-in-out;
  }

  .like-bar {
    background-color: grey;

    /* Anfangsfarbe */
    &.liked {
      background-color: green;
      /* Bei Like */
    }
  }

  .dislike-bar {
    background-color: grey;

    /* Anfangsfarbe */
    &.disliked {
      background-color: red;
      /* Bei Dislike */
    }
  }
}





.conversation-prompt {
  background-color: transparent;
}


button.like-button {
  background-color: transparent;
}

.topic-info {
  margin-top: 10px;
  display: flex;
  justify-content: space-between;
  font-size: 14px;
  color: #a39b9b;
}

.topic-box {
  display: flex;
  flex-direction: column;
  padding: 8px;
  margin: 10px auto;
  background-color: #ffffff;
  border-radius: 10px;
  box-shadow: 0 15px 20px rgba(1, 18, 251, 0.1);
  max-width: 90%;
  transition: transform 0.3s ease-in-out;




  .topic-image {
    min-width: 80%;

    object-fit: cover;
  }



  .topic-text {
    margin: 10px 0;
    text-align: justify;
    color: #333;
    font-size: 16px;

  }


  .join-button {
    background-color: transparent;
    max-height: 20%;
  }

  button {

    border-radius: 5px;
    border: none;


    font-size: 20px;
    cursor: pointer;
    transition: background-color 0.3s ease;
    display: flex;
    align-items: center;


  }

  .vote {
    display: flex;
    justify-content: space-around;
    max-height: 30px;

    p {
      margin-left: 0.5em;
      font-size: 15px;
    }
  }




  .author-image {
    width: 20px;
    height: 20px;
    border-radius: 50%;

  }

  .author-name {
    font-weight: bold;
    font-size: 12px;

  }

  .topic-content {
    flex: 1;
    /* Allow topic-content to take up remaining space */

  }

  @media only screen and (max-width: 600px) {
    .right-content {

      display: flex;
      justify-content: flex-end;
    }

    .topic-box .author-info {
      display: flex;
      align-items: center;

    }




    .percentage-display {
      text-align: center;
      margin-top: 10px;
    }

    .balken {
      min-width: 10em;
      display: flex;
    }



  }

  // ende von mediascreen
}

button:active,
button:focus {
  background-color: transparent;
  outline: none;
  /* Entfernt den fokussierten Rahmen um den Button */
}

.author-info {
  gap: 10px;
  width: 100%;
}

button.share-button {
  max-width: 30%;
  min-width: 30%;
  text-align: right;
  justify-content: space-evenly;
  background-color: transparent;
}

/* Füge folgende Regel hinzu, um die Elemente auf einer Zeile anzuzeigen */
.topic-box .author-info,
.topic-box .share-button {
  flex: 1;
  display: flex;
  align-items: center;

}

.profilbild {
  height: 100%;
}

.left-content {
  text-align: LEFT;
  display: flex;
  align-items: center;
  width: 70%;
}
</style>