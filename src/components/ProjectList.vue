<template>
  <div v-if="isLoggedIn" class="project-list">
    <table v-if="projects.length" class="projects-table">
      <thead>
      <tr>
        <th>Projet</th>
        <th>Ravworks ID</th>
        <th>Date de début</th>
        <th>Participants</th>
        <th></th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="project in projects" :key="project.id">
        <td>{{ project.name }}</td>
        <td>{{ project.ravworksId }}</td>
        <td>{{ project.startDatetime }}</td>
        <td>
          <div class="participants">
            <template v-for="(participant, index) in project.participants">
              <img v-if="index < 9" :key="participant" :src="getParticipantImage(participant)" alt="Participant Portrait" class="participant-image" />
              <span v-if="index === 9" :key="participant" class="ellipsis">...</span>
            </template>
          </div>
        </td>
        <td>
          <button v-if="project.participants.includes(characterId)" @click="viewProjectDetails(project)" class="details-button">Détails</button>
          <button v-else @click="joinProject(project)" class="join-button">Participer</button>
        </td>
      </tr>
      </tbody>
    </table>
    <div v-if="totalItems > 15" class="pagination">
      <button @click="previousPage" :disabled="currentPage === 1" type="button">Previous</button>
      <span>Page {{ currentPage }} / {{ maxPage }}</span>
      <button @click="nextPage" :disabled="currentPage === maxPage" type="button">Next</button>
    </div>
    <div v-else-if="!projects.length" class="loading">Loading projects...</div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: "ProjectList",
  data() {
    return {
      isLoggedIn: false,
      projects: [],
      totalItems: 0,
      currentPage: 1,
      characterId: null,
    };
  },
  computed: {
    maxPage() {
      return Math.ceil(this.totalItems / 10);
    }
  },
  mounted() {
    const sessionCookie = this.getCookie('Session');
    if (sessionCookie) {
      try {
        const sessionData = JSON.parse(sessionCookie);
        if (sessionData.id) {
          this.isLoggedIn = true;
          this.characterId = sessionData.id;
          this.fetchProjects(this.currentPage);
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
    previousPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
        this.fetchProjects(this.currentPage);
      }
    },
    nextPage() {
      if (this.currentPage < this.maxPage) {
        this.currentPage++;
        this.fetchProjects(this.currentPage);
      }
    },
    async fetchProjects(page = 1) {
      try {
        const response = await axios.get(`http://evetools.local/api/projects?page=${page}`);
        this.projects = response.data.member.map((project) => {
          const participants = project.json ? project.json.participants : [];
          return {
            id: project['@id'],
            name: project.name,
            ravworksId: project.ravworksId,
            startDatetime: this.formatDate(project.startDatetime),
            participants: participants,
          };
        });
        this.totalItems = response.data.totalItems;
      } catch (error) {
        console.error("Failed to fetch projects:", error);
      }
    },
    formatDate(dateString) {
      if (!dateString) return "-";
      const options = { year: 'numeric', month: '2-digit', day: '2-digit' };
      const date = new Date(dateString);
      return date.toLocaleDateString('fr-FR', options);
    },
    getParticipantImage(participantId) {
      return `https://images.evetech.net/characters/${participantId}/portrait`;
    },
    viewProjectDetails(project) {
      this.$emit('view-project-details', project);
    },
    joinProject(project) {
      // Logic for joining a project will be implemented later
      console.log('Joining project:', project.name);
    }
  },
};
</script>

<style scoped>
.project-list {
  margin-top: 20px;
  width: 80%;
}

.projects-table {
  width: 100%;
  border-collapse: collapse;
  color: #c2a611;
  background-color: rgba(255, 255, 255, 0.1);
}

.projects-table th, .projects-table td {
  border: 1px solid #2c3e50;
  padding: 10px;
  text-align: center;
}

.projects-table th:nth-child(3), .projects-table td:nth-child(3) {
  width: 150px;
}

.projects-table th:nth-child(4), .projects-table td:nth-child(4) {
  width: 200px;
}

.participants {
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
  max-width: 170px;
}

.participant-image {
  height: 30px;
  border-radius: 50%;
}

.ellipsis {
  align-self: center;
  margin-left: 8px;
}

.loading {
  color: #c2a611;
  font-size: 18px;
  margin-top: 20px;
}

.pagination {
  margin-top: 10px;
  display: flex;
  justify-content: center;
  gap: 10px;
}

.pagination button {
  padding: 5px 10px;
  background-color: #c2a611;
  color: #2c3e50;
  border: none;
  cursor: pointer;
  font-size: 16px;
}

.pagination button:disabled {
  background-color: #a9a9a9;
  cursor: not-allowed;
}

.details-button, .join-button {
  padding: 5px 10px;
  background-color: #c2a611;
  color: #2c3e50;
  border: none;
  cursor: pointer;
  font-size: 14px;
}
</style>
