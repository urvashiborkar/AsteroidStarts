<template>
  <div>
    <h1>Neo Stats</h1>
    <form @submit.prevent="fetchNeoData">
      <label>Start Date:</label>
      <input type="date" v-model="startDate" />
      <label>End Date:</label>
      <input type="date" v-model="endDate" />
      <button type="submit">Submit</button>
    </form>

    <div v-if="stats">
      <h3>Asteroid Stats</h3>
      <p><strong>Fastest Asteroid:</strong> {{ stats.fastestAsteroid.id }} ({{ stats.fastestAsteroid.speed }} km/h)</p>
      <p><strong>Closest Asteroid:</strong> {{ stats.closestAsteroid.id }} ({{ stats.closestAsteroid.distance }} km)</p>
      <p><strong>Average Size:</strong> {{ stats.averageSize }} km</p>
    </div>

    <!-- Chart Container -->
    <canvas id="neoChart" width="400" height="200"></canvas>
  </div>
</template>

<script>
import axios from 'axios';
import Chart from 'chart.js/auto';

export default {
  data() {
    return {
      startDate: '',
      endDate: '',
      stats: null,
      chart: null,
    };
  },
  methods: {
    async fetchNeoData() {
      try {
        const response = await axios.get(
          `https://api.nasa.gov/neo/rest/v1/feed?start_date=${this.startDate}&end_date=${this.endDate}&api_key=DEMO_KEY`
        );
        this.processNeoData(response.data);
        this.plotChart(); 
      } catch (error) {
        console.error('Error fetching Neo data:', error);
      }
    },
    processNeoData(data) {
      let fastestAsteroid = { id: '', speed: 0 };
      let closestAsteroid = { id: '', distance: Infinity };
      let totalSize = 0, asteroidCount = 0;
      let dailyAsteroidCount = []; 
      let labels = []; 

      for (let date in data.near_earth_objects) {
        labels.push(date); 
        let countForDay = data.near_earth_objects[date].length;
        dailyAsteroidCount.push(countForDay); 

        data.near_earth_objects[date].forEach((asteroid) => {
          const speed = asteroid.close_approach_data[0].relative_velocity.kilometers_per_hour;
          const distance = asteroid.close_approach_data[0].miss_distance.kilometers;
          const size = (asteroid.estimated_diameter.kilometers.estimated_diameter_min +
                       asteroid.estimated_diameter.kilometers.estimated_diameter_max) / 2;

          totalSize += size;
          asteroidCount++;

          if (speed > fastestAsteroid.speed) {
            fastestAsteroid = { id: asteroid.id, speed };
          }
          if (distance < closestAsteroid.distance) {
            closestAsteroid = { id: asteroid.id, distance };
          }
        });
      }

      this.stats = {
        fastestAsteroid,
        closestAsteroid,
        averageSize: (totalSize / asteroidCount).toFixed(2)
      };

     
      this.chartData = {
        labels, 
        dailyAsteroidCount 
      };
    },
    plotChart() {
      if (this.chart) {
        this.chart.destroy(); 
      }

      const ctx = document.getElementById('neoChart').getContext('2d');
      this.chart = new Chart(ctx, {
        type: 'line', 
        data: {
          labels: this.chartData.labels, // Dates as labels
          datasets: [{
            label: 'Number of Asteroids',
            data: this.chartData.dailyAsteroidCount,
            backgroundColor: 'rgba(54, 162, 235, 0.2)',
            borderColor: 'rgba(54, 162, 235, 1)',
            borderWidth: 1
          }]
        },
        options: {
          scales: {
            y: {
              beginAtZero: true
            }
          }
        }
      });
    }
  }
};
</script>

