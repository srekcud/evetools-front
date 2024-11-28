<template>
  <div class="project-details">
    <h2>Détails du projet: {{ project.name }}</h2>
    <div class="tabs">
      <button v-for="(type, tabName) in tabs" :key="tabName" @click="changeTab(type)">
        {{ tabName }}
      </button>
    </div>
    <table v-if="details.length" class="details-table">
      <thead>
      <tr>
        <th v-for="(header, index) in headers" :key="index">{{ header }}</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="(detail, index) in details" :key="index">
        <td v-for="(value, valueIndex) in detail" :key="valueIndex">{{ value }}</td>
      </tr>
      </tbody>
    </table>
    <div v-else class="loading">Loading details...</div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "ProjectDetails",
  props: {
    project: Object,
  },
  data() {
    return {
      tabs: {
        "Intermediate Composite Reactions": "firstStageReaction",
        "Composite Reactions": "secondStageReaction",
        "Biochem Reactions": "bioReaction",
        "Hybrid Reactions": "hybridReactions",
        "Advanced Components": "advancedComponents",
        "Capital Components": "capitalComponents",
        "Others": "others",
        "End Product Jobs": "endProductJob",
      },
      currentTab: null,
      details: [],
      headers: [],
    };
  },
  methods: {
    changeTab(type) {
      this.currentTab = type;
      this.fetchDetails(type);
    },
    async fetchDetails(type) {
      try {
        const response = await axios.get(
            `http://evetools.local/api/projects/${this.project.ravworksId}/details/${type}`
        );
        this.details = response.data.member;
        if (this.details.length > 0) {
          this.headers = Object.keys(this.details[0]);
        }
      } catch (error) {
        console.error("Failed to fetch project details:", error);
      }
    },
  },
};
</script>

<style scoped>
.project-details {
  padding: 20px;
  color: #c2a611;
}

.tabs {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.details-table {
  width: 100%;
  border-collapse: collapse;
  color: #c2a611;
}

.details-table th, .details-table td {
  border: 1px solid #2c3e50;
  padding: 10px;
  text-align: center;
}

.loading {
  color: #c2a611;
  font-size: 18px;
  margin-top: 20px;
}
</style>