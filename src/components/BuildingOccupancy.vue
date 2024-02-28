<template>
  <div class="container">
    <div v-if="loading" class="loading">Chargement en cours...</div>
    <div v-else>
      <div class="dashboard">
        <div class="dashboard-content">
          <p v-if="activeFloor === null" class="dashboard-info">
            Total des Pièces : {{ getTotalRoomCount() }}
          </p>
          <p v-else class="dashboard-info">
            Nombre des Pièces : {{ getFloorRoomCount() }}
          </p>
        </div>
      </div>

      <div v-for="(building, index) in buildings" :key="index" class="building">
        <h1 class="page-title">Statistiques d'occupation du bâtiment:</h1>
        <p class="building-name">{{ building.name }}</p>
        <p v-if="activeFloor === null" class="select-floor-message">
          Veuillez sélectionner un étage.
        </p>
        <div
          v-for="(floor, floorIndex) in building.floors"
          :key="floorIndex"
          class="floor"
        >
          <h3
            @mouseover="changeCursor('pointer')"
            @mouseleave="changeCursor('auto')"
            @click="toggleFloor(index, floorIndex)"
            :class="{
              'floor-name': true,
              'selected-floor': activeFloor === floorIndex,
            }"
          >
            {{ floor.name }} ({{ getOccupancyPercentage(floor) }}%)
          </h3>
          <div v-if="activeFloor === floorIndex" class="room-container">
            <div v-for="room in floor.rooms" :key="room.id" class="room">
              <p class="room-name">
                {{ room.name }} :
                <span v-if="room.occupied === true" class="occupied"
                  >Occupé</span
                >
                <span v-else-if="room.occupied === false" class="unoccupied"
                  >Non Occupé</span
                >
                <span v-else class="undefined">Indéfini</span>
              </p>
            </div>
          </div>
        </div>
      </div>

      <img src="@/assets/logo2.png" alt="Logo 2" class="logo" />
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      loading: true,
      buildings: [],
      activeFloor: null,
    };
  },
  async mounted() {
    await this.fetchOccupancyData();
  },
  methods: {
    async fetchOccupancyData() {
      try {
        const response = await axios.get(
          "https://api-developers.spinalcom.com/api/v1/geographicContext/space"
        );
        const buildings = response.data.children.map(async (building) => {
          return {
            name: building.name,
            floors: await Promise.all(
              building.children.map(async (floor) => {
                return {
                  name: floor.name,
                  rooms: await Promise.all(
                    floor.children.map(async (room) => {
                      return {
                        id: room.dynamicId,
                        name: room.name,
                        occupied: await this.getOccupancyStatus(room.dynamicId),
                      };
                    })
                  ),
                };
              })
            ),
          };
        });
        this.buildings = await Promise.all(buildings);
        this.loading = false;
      } catch (error) {
        console.error("Erreur lors de la récupération des données :", error);
        this.loading = false;
      }
    },
    async getOccupancyStatus(dynamicId) {
      try {
        const response = await axios.get(
          `https://api-developers.spinalcom.com/api/v1/room/${dynamicId}/control_endpoint_list`
        );

        if (response.data && response.data.length > 0) {
          const occupancyEndpoint = response.data[0].endpoints.find(
            (endpoint) => endpoint.name === "Occupation"
          );

          if (occupancyEndpoint) {
            return occupancyEndpoint.currentValue;
          } else {
            console.error(
              "Point de terminaison d'occupation non trouvé pour la salle :",
              dynamicId
            );
            return "INDÉFINI";
          }
        } else {
          console.error("Aucune donnée retournée pour la salle :", dynamicId);
          return "INDÉFINI";
        }
      } catch (error) {
        console.error(
          "Erreur lors de la récupération du statut d'occupation :",
          error
        );
        return "INDÉFINI";
      }
    },
    toggleFloor(buildingIndex, floorIndex) {
      this.activeFloor = this.activeFloor === floorIndex ? null : floorIndex;
    },
    changeCursor(cursorType) {
      document.body.style.cursor = cursorType;
    },
    getOccupancyPercentage(floor) {
      if (floor.rooms.length === 0) return 0;
      const occupiedRooms = floor.rooms.filter(
        (room) => room.occupied === true
      ).length;
      return ((occupiedRooms / floor.rooms.length) * 100).toFixed(2);
    },
    getTotalRoomCount() {
      let totalCount = 0;
      this.buildings.forEach((building) => {
        building.floors.forEach((floor) => {
          totalCount += floor.rooms.length;
        });
      });
      return totalCount;
    },
    getFloorRoomCount() {
      if (this.activeFloor === null) return 0;
      return this.buildings
        .map((building) => building.floors[this.activeFloor].rooms.length)
        .reduce((acc, curr) => acc + curr, 0);
    },
  },
};
</script>

<style scoped>
.container {
  padding: 20px;
  background-color: #f5f5f5;
}

.loading {
  font-size: 18px;
  font-weight: bold;
  color: #333;
  text-align: center;
}

.page-title {
  font-size: 28px;
  font-weight: bold;
  color: #333;
  margin-bottom: 20px;
}

.dashboard {
  margin-bottom: 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
  padding: 10px;
}

.dashboard-content {
  margin: auto;
}

.dashboard-info {
  font-size: 16px;
  color: #333;
}

.building-name {
  font-size: 24px;
  font-weight: bold;
  color: #333;
  margin-bottom: 10px;
}

.floor-name {
  font-size: 20px;
  font-weight: bold;
  color: #555;
  cursor: pointer;
  transition: color 0.3s ease;
}

.floor-name:hover {
  color: #007bff;
}

.room-container {
  max-height: 250px;
  width: 250px;
  overflow-y: auto;
  border: 1px solid #ccc;
  padding: 10px;
  margin: 0 auto;
  border-radius: 5px;
  background-color: #fff;
}

.room {
  margin-bottom: 10px;
}

.room-name {
  font-size: 16px;
  color: #333;
}

.occupied {
  color: green;
  font-weight: bold;
}

.unoccupied {
  color: red;
  font-weight: bold;
}

.undefined {
  color: gray;
  font-weight: bold;
}

.select-floor-message {
  font-size: 16px;
  color: gray;
  text-align: center;
}

.logo {
  position: fixed;
  bottom: 20px;
  right: 20px;
  width: 200px;
  height: auto;
  border-radius: 10px;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.2);
}

.building {
  margin-bottom: 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
  padding: 10px;
  background: linear-gradient(45deg, #f5f5f5, #e0e0e0);
}

.floor {
  margin-bottom: 15px;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.selected-floor {
  background-color: #e0e0e0;
}
</style>
