<template>
  <div class="project-details">
    <h2>Détails du projet : {{ project.name }}</h2>
    <div class="tabs">
      <button
          v-for="(type, tabName) in tabs"
          :key="tabName"
          @click="changeTab(type)"
          :disabled="isTabDisabled(type)"
          :class="{ 'active-tab': currentTab === type }"
      >
        {{ tabName }}
      </button>
    </div>
    <div v-if="details.length" class="table-container">
      <table class="details-table">
        <thead class="fixed-header">
        <tr>
          <th v-for="header in headers" :key="header">{{ header }}</th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="(detail, index) in details" :key="index">
          <td v-for="header in headers" :key="header">{{ detail[header] || '' }}</td>
        </tr>
        </tbody>
      </table>
    </div>
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
      headers: ["name", "run", "cost", "character", "startDatetime", "endDatetime", "status"], // Columns to display
      totalItems: {},
    };
  },
  mounted() {
    this.fetchAllTotalItems();
  },
  methods: {
    async fetchAllTotalItems() {
      for (let type of Object.values(this.tabs)) {
        try {
          const response = await axios.get(
              `https://evetools.local/api/projects/${this.project.ravworksId}/details/${type}`
          );
          this.totalItems[type] = response.data.totalItems;
        } catch (error) {
          console.error("Failed to fetch total items for tab:", type, error);
        }
      }
      this.setInitialTab();
    },
    setInitialTab() {
      for (let type of Object.values(this.tabs)) {
        if (!this.isTabDisabled(type)) {
          this.changeTab(type);
          break;
        }
      }
    },
    isTabDisabled(type) {
      return this.totalItems[type] === 0;
    },
    changeTab(type) {
      this.currentTab = type;
      this.fetchDetails(type);
    },
    async fetchDetails(type) {
      try {
        const response = await axios.get(
            `https://evetools.local/api/projects/${this.project.ravworksId}/details/${type}`
        );
        this.details = response.data.member.map((item) => {
          // Ensure only relevant columns are present, and fill missing values with ''
          const filteredItem = {};
          this.headers.forEach((header) => {
            filteredItem[header] = item[header] || ""; // Default to empty string if value is missing
          });
          return filteredItem;
        });
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

.tabs button {
  background-color: #f0f0f0;
  border: 1px solid #c2a611;
  padding: 10px;
  cursor: pointer;
  color: #333;
}

.tabs button:disabled {
  background-color: #a9a9a9;
  color: #6b6b6b;
  cursor: not-allowed;
}

.tabs button.active-tab {
  background-color: #c2a611;
  color: #ffffff;
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

.fixed-header {
  position: sticky;
  top: 0;
  background-color: #f0f0f0;
  z-index: 1;
}

.table-container {
  max-height: 400px;
  overflow-y: auto;
  border: 1px solid #2c3e50;
}

.loading {
  color: #c2a611;
  font-size: 18px;
  margin-top: 20px;
}
</style>
