<template>
  <header class="header">
    <div class="left-section">
      <button v-if="showBackButton" @click="goBack" class="back-button">Back</button>
    </div>
    <div class="right-section">
      <div v-if="isLoggedIn" class="character-info">
        <span>{{ characterName }}</span>
        <img :src="characterPortraitUrl" alt="Character Portrait" class="character-portrait" />
      </div>
      <SSOLogin v-else />
    </div>
  </header>
</template>


<script>
import SSOLogin from './SSOLogin.vue';

export default {
  name: "HeaderComponent",
  components: {
    SSOLogin,
  },
  props: {
    showBackButton: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      characterName: null,
      characterId: null,
    };
  },
  computed: {
    goBack() {
      return this.$emit("back-to-list");
    },
    isLoggedIn() {
      return this.characterName && this.characterId;
    },
    characterPortraitUrl() {
      return `https://images.evetech.net/characters/${this.characterId}/portrait`;
    }
  },
  mounted() {
    const sessionCookie = this.getCookie('Session');
    if (sessionCookie) {
      try {
        const sessionData = JSON.parse(sessionCookie);
        if (sessionData.character_name && sessionData.id) {
          this.characterName = sessionData.character_name;
          this.characterId = sessionData.id;
        }
      } catch (e) {
        console.error('Failed to parse session cookie:', e);
      }
    }
  },
  methods: {
    getCookie(name) {
      const value = `; ${document.cookie}`;
      const parts = value.split(`; ${name}=`);
      if (parts.length === 2) return parts.pop().split(';').shift();
    },
  },
};
</script>

<style scoped>
.header {
  padding: 20px;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.left-section {
  flex: 0 1 auto;
}

.right-section {
  margin-left: auto;
  display: flex;
  align-items: center;
  gap: 10px;
}

.character-info {
  display: flex;
  align-items: center;
  color: #c2a611;
  font-size: 25px;
  font-weight: bold;
  gap: 10px;
}

.character-portrait {
  height: 40px;
  border-radius: 50%;
  margin-right: 20px;
}

.back-button {
  padding: 5px 10px;
  background-color: #c2a611;
  color: #2c3e50;
  border: none;
  cursor: pointer;
  font-size: 14px;
  margin-left: 30px;
}
</style>
