<template>
  <header class="header">
    <div v-if="isLoggedIn" class="character-info">
      <span>{{ characterName }}</span>
      <img :src="characterPortraitUrl" alt="Character Portrait" class="character-portrait" />
    </div>
    <SSOLogin v-else />
  </header>
</template>

<script>
import SSOLogin from './SSOLogin.vue';

export default {
  name: "HeaderComponent",
  components: {
    SSOLogin,
  },
  data() {
    return {
      characterName: null,
      characterId: null,
    };
  },
  computed: {
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
  text-align: center;
  display: flex;
  justify-content: flex-end;
  align-items: center;
}

.character-info {
  display: flex;
  align-items: center;
  gap: 10px;
  color: #c2a611;
  font-size: 25px;
  font-weight: bold;
}


.character-portrait {  height: 40px;
  border-radius: 50%;
  margin-right: 20px;
}
</style>
