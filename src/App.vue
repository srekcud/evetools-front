<template>
  <div id="app">
    <div v-if="isWaiting" class="waiting-screen">
      <p>Authenticating, please wait...</p>
    </div>
    <div v-else>
      <BackgroundLogo />
      <HeaderComponent
          :show-back-button="showProjectDetails"
          @back-to-list="showList"/>
      <ProjectList v-if="isLoggedIn && !showProjectDetails" @view-project-details="showDetails" />
      <ProjectDetails v-if="isLoggedIn && showProjectDetails" :project="selectedProject" />
    </div>
  </div>
</template>

<script>
import axios from "axios";
import BackgroundLogo from "./components/BackgroundLogo.vue";
import HeaderComponent from "./components/HeaderComponent.vue";
import ProjectList from "./components/ProjectList.vue";
import ProjectDetails from "./components/ProjectDetails.vue";

export default {
  components: {
    BackgroundLogo,
    HeaderComponent,
    ProjectList,
    ProjectDetails,
  },
  data() {
    return {
      showProjectDetails: false,
      selectedProject: null,
      isLoggedIn: false,
      isWaiting: false, // State for waiting screen
    };
  },
  async mounted() {
    const urlParams = new URLSearchParams(window.location.search);
    const code = urlParams.get("code");

    if (code) {
      try {
        this.isWaiting = true; // Show waiting screen

        // Effectuer l'appel API avec POST et le bon content-type
        await axios.post(
            `https://evetools.local/api/auth/code/${code}`,
            {},
            {
              headers: {
                "Content-Type": "application/ld+json", // Corriger le content-type
              },
              withCredentials: true, // Inclure les cookies
            }
        );

        // Attendre la création du cookie "Session"
        await this.waitForCookie("Session");

        // Rediriger vers le domaine de base
        window.location.href = "/";
      } catch (error) {
        console.error("Error during login process:", error);
      } finally {
        this.isWaiting = false; // Hide waiting screen
      }
    } else {
      // Vérifier si le cookie de session existe
      const sessionCookie = this.getCookie("Session");
      if (sessionCookie) {
        try {
          const sessionData = JSON.parse(sessionCookie);
          if (sessionData.id) {
            this.isLoggedIn = true;
          }
        } catch (e) {
          console.error("Failed to parse session cookie:", e);
        }
      }
    }
  },
  methods: {
    showList() {
      this.showProjectDetails = false;
      this.selectedProject = null;
    },
    getCookie(name) {
      const value = `; ${document.cookie}`;
      const parts = value.split(`; ${name}=`);
      if (parts.length === 2) return parts.pop().split(";").shift();
    },
    async waitForCookie(name, timeout = 5000) {
      const interval = 100; // Intervalle pour vérifier (ms)
      const endTime = Date.now() + timeout;

      while (Date.now() < endTime) {
        const cookie = this.getCookie(name);
        if (cookie) return; // Cookie trouvé
        await new Promise((resolve) => setTimeout(resolve, interval));
      }

      throw new Error(`Cookie ${name} not found within timeout`);
    },
    showDetails(project) {
      this.selectedProject = project;
      this.showProjectDetails = true;
    },
  },
};
</script>

<style scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  background-color: #696969;
  position: relative;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  margin: 0;
  padding: 0;
  overflow-x: hidden;
  overflow-y: hidden;
  box-sizing: border-box;
}

body {
  margin: 0;
  padding: 0;
  color: #c2a611;
}

.waiting-screen {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  justify-content: center;
  align-items: center;
  color: white;
  font-size: 1.5rem;
  z-index: 1000;
}
</style>
