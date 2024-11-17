<!-- App.vue -->
<template>
  <div id="app">
    <!-- Map Container -->
    <div ref="mapContainer" class="map-container"></div>


    <!-- Filters Toggle Button -->
    <button class="filters-toggle" @click="toggleFilters">
      {{ filtersVisible ? 'Закрыть фильтры' : 'Фильтры' }}
    </button>

    <!-- Filters Panel -->
    <div class="filters" v-show="filtersVisible"> 

      <!-- Feature Count Display -->
      <div class="feature-count">
        <img src="./assets/custom-marker.png" alt="Custom Marker" />
        {{ displayedFeatureCount }}
      </div>   


    <div class="filter-buttons">
      <button @click="applyFilters">Применить</button>
      <button @click="resetFilters">Сбросить</button>
    </div>
    <br>

      <div v-for="clause in clauses" :key="clause"  class="filter-div">
        <input type="checkbox" :value="clause" v-model="selectedClauses"> 
        <span class="filter-label">{{ clause }} </span>
      </div>
 
      <div v-for="tag in tags" :key="tag"   class="filter-div">
        <input type="checkbox" :value="tag" v-model="selectedTags"> 
        <span class="filter-label">{{ tag }}</span>
      </div>
       <!-- Filter by Linkink -->
       <div   class="filter-div">
        <label>
          <input type="checkbox" v-model="filterLinkink">
          <span class="filter-label">группа поддержки</span>
          
        </label>
      </div>
      
      
      <!-- Apply and Reset Filters Buttons -->
       <br>
    <div class="filter-buttons">
      <button @click="applyFilters">Применить</button>
      <button @click="resetFilters">Сбросить</button>
    </div>

    </div>
    
    <!-- Modal for Feature List -->
    <!-- Modal for Feature List or Feature Details -->
    <div v-if="showModal" class="modal" @click="closeModal">
      <div class="modal-content" @click.stop>
        <span class="close-button" @click="closeModal">&times;</span>

        <!-- Feature Details View -->
        <div v-if="selectedFeature">
          <button @click="backToList">Обратно к списку</button>
          <h3>{{ selectedFeature.properties.name || 'No Name Available' }}</h3>
          
          <div v-if="selectedFeature.properties.linkink">
             
            <a :href="selectedFeature.properties.linkink" target="_blank">{{ selectedFeature.properties.linkink }}</a>
          </div>


          <div v-if="selectedFeature.properties.main && selectedFeature.properties.main.length">
          
            <p>{{ selectedFeature.properties.main.join(' ') }}</p>
          </div>


          <div v-if="selectedFeature.properties.address && selectedFeature.properties.address.length">
        
            <p>{{ selectedFeature.properties.address.join(', ') }}</p>
          </div>
          <div v-if="selectedFeature.properties.clauses && selectedFeature.properties.clauses.length">
            <ul>
              <li v-for="clause in selectedFeature.properties.clauses" :key="clause">{{ clause }}</li>
            </ul>
          </div>
          <div v-if="selectedFeature.properties.tags && selectedFeature.properties.tags.length">
            <ul>
              <li v-for="tag in selectedFeature.properties.tags" :key="tag">{{ tag }}</li>
            </ul>
          </div> 
          <div v-if="selectedFeature.properties.sourceUrl"> 
            <a :href="selectedFeature.properties.sourceUrl" target="_blank">Мемориал. Поддержка политзаключённых</a>
          </div>
        </div>

        <!-- Feature List View -->
        <div v-else>
          <ul>
            <li
              v-for="feature in currentFeatures"
              :key="feature.properties.name"
              @click="showFeatureDetails(feature)"
              class="feature-list-item"
            >
              {{ feature.properties.name || 'No Name Available' }}
            </li>
          </ul>
        </div>
      </div>
    </div>

  </div>
</template>

<script>
import mapboxgl from 'mapbox-gl';

import MapboxLanguage from '@mapbox/mapbox-gl-language';

export default {
  name: 'App',
  data() {
    return {
      map: null,
      geojsonData: null,
      featuresByCoordinates: {},
      markers: [],
      showModal: false,
      currentFeatures: [],
      clauses: [],
      tags: [],
      selectedClauses: [],
      selectedTags: [],
      filtersVisible: false, 
      displayedFeatureCount: 0,
      selectedFeature: null,
    };
  },
  mounted() {
    // Initialize Mapbox
    mapboxgl.accessToken = 'pk.eyJ1IjoidnVmb3JpYSIsImEiOiJjbTJybXJiaWsxOHVnMmpzYnZtZWk4ZXB1In0.24OWriNL8SBYXoLdBtO9EA'; // Replace with your Mapbox access token
    this.map = new mapboxgl.Map({
      container: this.$refs.mapContainer,
      style: 'mapbox://styles/mapbox/navigation-night-v1',
      center: [81.755133,59.08195], // Adjust center as needed
      zoom: 3,
    });
    this.map.addControl(
    new MapboxLanguage({
      defaultLanguage: 'ru',  
    })
  );
    // Load GeoJSON data
    fetch('/your-data.geojson') // Ensure the GeoJSON file is in the public directory
      .then((response) => response.json())
      .then((data) => {
        this.geojsonData = data;

        // Process data and extract unique clauses and tags
        this.processFeaturesByCoordinates();

        // Add markers to the map
        this.addMarkers();
      })
      .catch((error) => {
        console.error('Error loading GeoJSON data:', error);
      });
  },
  methods: {

    toggleFilters() {
      this.filtersVisible = !this.filtersVisible;
    },
    processFeaturesByCoordinates() {
      const clauseSet = new Set();
      const tagSet = new Set();

      this.geojsonData.features.forEach((feature) => {
        const coords = feature.geometry.coordinates.join(',');
        if (!this.featuresByCoordinates[coords]) {
          this.featuresByCoordinates[coords] = [];
        }
        this.featuresByCoordinates[coords].push(feature);

        // Extract unique clauses and tags
        feature.properties.clauses.forEach((clause) => clauseSet.add(clause));
        feature.properties.tags.forEach((tag) => tagSet.add(tag));
      });

      this.clauses = Array.from(clauseSet).sort();
      this.tags = Array.from(tagSet).sort();

      // Set the initial displayed feature count
      this.displayedFeatureCount = this.geojsonData.features.length;
    },

    addMarkers() {
      // Remove existing markers
      if (this.markers.length) {
        this.markers.forEach((marker) => marker.remove());
        this.markers = [];
      }

      Object.keys(this.featuresByCoordinates).forEach((coords) => {
        const features = this.featuresByCoordinates[coords];
        const [lng, lat] = coords.split(',').map(Number);

        // Create a DOM element for the marker
        const el = document.createElement('div');
        el.className = 'custom-marker';

        // Handle click event
        el.addEventListener('click', () => {
          this.showFeatureList(features, [lng, lat]);
        });

        // Add marker to the map
        const marker = new mapboxgl.Marker(el)
          .setLngLat([lng, lat])
          .addTo(this.map);

        this.markers.push(marker);
      });
    },
    showFeatureList(features) {
      this.currentFeatures = features;
      this.selectedFeature = null; // Ensure no feature is selected
      this.showModal = true;
    },

    showFeatureDetails(feature) {
      this.selectedFeature = feature;
    },
    backToList() {
      this.selectedFeature = null;
    },
    closeModal() {
      this.showModal = false;
      this.selectedFeature = null;
    },
    applyFilters() {
    // Filter features based on selected clauses, tags, and 'linkink' value
    const filteredFeatures = this.geojsonData.features.filter((feature) => {
      const matchesClause = this.selectedClauses.length
        ? feature.properties.clauses && feature.properties.clauses.some((clause) => this.selectedClauses.includes(clause))
        : true;

      const matchesTag = this.selectedTags.length
        ? feature.properties.tags && feature.properties.tags.some((tag) => this.selectedTags.includes(tag))
        : true;

      const matchesLinkink = this.filterLinkink
        ? feature.properties.linkink && feature.properties.linkink.trim() !== ''
        : true;

      return matchesClause && matchesTag && matchesLinkink;
    });

    // Update the displayed feature count
    this.displayedFeatureCount = filteredFeatures.length;

    // Re-group features by coordinates
    this.featuresByCoordinates = {};
    filteredFeatures.forEach((feature) => {
      const coords = feature.geometry.coordinates.join(',');
      if (!this.featuresByCoordinates[coords]) {
        this.featuresByCoordinates[coords] = [];
      }
      this.featuresByCoordinates[coords].push(feature);
    });

    // Re-add markers to the map
    this.addMarkers();

    // Close the filters panel
    this.filtersVisible = false;
  },

  resetFilters() {
      // Reset all filter selections
      this.selectedClauses = [];
      this.selectedTags = [];
      this.filterLinkink = false;

      // Reset visibility of filter sections
      this.clausesVisible = true;
      this.tagsVisible = true;

      // Apply filters to refresh the map
      this.applyFilters();

      // Close the filters panel
      this.filtersVisible = false;
    }, 
  },
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@400;500;600;700&display=swap');

body {
  font-family: 'Montserrat', sans-serif;
}

/* Ensure the app container fills the viewport */
#app {
  position: relative;
  height: 100vh;
  width: 100vw;
  overflow: hidden;
}

/* Map container styles */
.map-container {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 100%;
}

/* Custom marker styles */
.custom-marker {
  background-image: url('./assets/custom-marker.png'); /* Replace with your image path */
  background-size: cover;
  width: 30px;
  height: 30px;
  cursor: pointer;
}

/* Filters panel styles */
.filters {
  position: absolute;
  top: 10px;
  left: 10px;
  background-color: rgba(255, 255, 255, 0.9);
  padding: 10px;
  z-index: 1;
  max-height: 80vh;
  overflow-y: auto;
  border-radius: 4px;
}

/* Modal styles */
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 2;
}

.modal-content {
  background-color: white;
  padding: 20px;
  max-height: 80vh;
  overflow-y: auto;
  border-radius: 4px;
  width: 300px;
}

.close-button {
  float: right;
  font-size: 24px;
  cursor: pointer;
}



.mapboxgl-canvas{ 
  filter: contrast(140%) grayscale(100%) saturate(0%) brightness(74%);
}

#app {
  position: relative;
  height: 100vh;
  width: 100vw;
  margin: 0;
  padding: 0;
  overflow: hidden;
}

/* Map container styles */
.map-container {
  position: relative; /* Changed from absolute to relative */
  height: 100%;
  width: 100%;
}



/* Optional: Style popups for better readability */
.mapboxgl-popup-content {
  font-family: Arial, sans-serif;
  font-size: 14px;
  color: #333;
}

.mapboxgl-popup-content h3 {
  margin-top: 0;
  font-size: 16px;
  color: #000;
}

.mapboxgl-popup-content p {
  margin: 5px 0;
}

.mapboxgl-popup-content button {
  margin-top: 10px;
  padding: 5px 10px;
  background-color: #51bbd6;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.mapboxgl-popup-content button:hover {
  background-color: #429ab8;
}

/* Modal Styling */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background: #fff;
  padding: 20px;
  border-radius: 8px;
  width: 80%;
  max-width: 500px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
  position: relative;
}

.modal-content h3 {
  margin-top: 0;
  font-size: 18px;
  color: #333;
}

.modal-content p {
  margin: 10px 0;
  font-size: 14px;
  color: #555;
}

.modal-content button {
  margin-top: 15px;
  padding: 8px 16px;
  background-color: #51bbd6;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.modal-content button:hover {
  background-color: #429ab8;
}
.mapboxgl-ctrl-logo, .mapboxgl-ctrl-attrib{
  display: none !important;
}
.mapboxgl-popup-content{
  background-color: #000009 !important;
  color: #ccc;
  padding-right: 30px;
  min-width: 222px;
}
.mapboxgl-popup-close-button{
background-color: transparent !important;
}
.mapboxgl-popup-content a{
  color: #ccc !important;
  padding-top: 20px !important;
  display: block;
}
.modal-content, .modal-content *{
  background-color: #000009 !important;
  color: #ddd !important; 
}
.modal-content button{
  position: absolute;
  right: 0;
  top: 0;
}
.mapboxgl-popup-anchor-bottom .mapboxgl-popup-tip{
  border-top-color: #000009 !important;
}
body{
  margin: 0 !important;
}


.filters-toggle {
  position: absolute;
  top: 10px;
  left: 10px;
  z-index: 15;
  background-color: rgba(255, 255, 255, 0.9);
  border: none;
  padding: 8px 12px;
  cursor: pointer;
  border-radius: 4px;
}


/* Filters panel styles */
.filters {
  position: absolute;
  top: 50px; /* Adjusted to be below the toggle button */
  left: 10px;
  z-index: 10;
  /* ... other styles ... */
}


.modal li{
  list-style-type: none;
}
.modal li:hover{
  cursor: pointer;
}
.modal ul{
  padding-left: 0;
}
.filters, .filters-toggle{
  filter: invert(1);
}
.filters input{
  filter: saturate(0);
  width: 20px;
  height: 20px;
  position: absolute;
}
.filter-label { 
  display: block; 
  margin-left: 35px; 
    
  margin-top: 3px;
}
.filter-div{
  margin-bottom: 5px;
  position: relative;
  padding-top: 5px;
}

.feature-count img{
  filter: invert(1);
  display: block;
  margin-bottom: 10px;
  width: 30px;
}
.feature-count{
  position: absolute;
  right: 30px;
  text-align: center;
}

.feature-list-item{
  margin-bottom: 5px;
}
.filter-buttons button{
  margin-right: 10px;
}
@media (max-width: 980px) {
  .modal-content button + h3{
    width: 100% !important;
    margin-top: 40px;
  }
}
</style>
