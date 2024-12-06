<template>
  <div id="app">
    <BackgroundLogo />
    <HeaderComponent
        :show-back-button="showProjectDetails"
        @back-to-list="showList"/>
    <ProjectList v-if="isLoggedIn && !showProjectDetails" @view-project-details="showDetails" />
    <ProjectDetails v-if="isLoggedIn && showProjectDetails" :project="selectedProject" />
  </div>
</template>

<script>
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
    };
  },
  mounted() {
    const sessionCookie = this.getCookie('Session');
    if (sessionCookie) {
      try {
        const sessionData = JSON.parse(sessionCookie);
        if (sessionData.id) {
          this.isLoggedIn = true;
        }
      } catch (e) {
        console.error('Failed to parse session cookie:', e);
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
      if (parts.length === 2) return parts.pop().split(';').shift();
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
  //overflow: hidden;
  color: #c2a611;
}
</style>
