<template>
  <div id="app">
    <!-- Map Container -->
    <div ref="mapContainer" class="map-container"></div>

    <!-- Top-Left Filter Controls -->
    <div class="filters-toggle-container" ref="filtersToggleContainer">
      <!-- Feature Count -->
      <div class="feature-count">
        <img src="./assets/custom-marker.png" alt="Custom Marker" />
        {{ displayedFeatureCount }}
      </div>

      <!-- Clauses Button -->
      <button
        class="filters-toggle"
        @click="setActiveFilter('clauses')"
        :class="{ active: activeFilterCategory === 'clauses' }"
      >
        Clauses
      </button>

      <!-- Tags Button -->
      <button
        class="filters-toggle"
        @click="setActiveFilter('tags')"
        :class="{ active: activeFilterCategory === 'tags' }"
      >
        Tags
      </button>

      <!-- Apply Button -->
      <button class="filters-toggle" @click="applyFilters">
        Применить
      </button>

      <!-- Reset Button -->
      <button class="filters-toggle" @click="resetFilters">
        Сбросить
      </button>
    </div>

    <!-- Date Slider -->
    <div class="date-slider-container">
      <div id="date-slider"></div>
      <div class="selected-dates">
        <span>{{ formattedStartDate }}</span>
        <span>{{ formattedEndDate }}</span>
      </div>
      <div class="play-pause-controls">
        <button @click="togglePlay">
          {{ isPlaying ? 'Пауза' : 'Воспроизвести' }}
        </button>
      </div>
    </div>

    <!-- Filters Panel -->
    <div class="filters" v-show="filtersVisible" ref="filtersPanel">
      <!-- Clauses Filter -->
      <div v-if="activeFilterCategory === 'clauses'">
        <div v-for="clause in clauses" :key="clause" class="filter-div">
          <input type="checkbox" :value="clause" v-model="selectedClauses" />
          <span class="filter-label">{{ clause }}</span>
        </div>
      </div>

      <!-- Tags Filter -->
      <div v-if="activeFilterCategory === 'tags'">
        <div v-for="tag in tags" :key="tag" class="filter-div">
          <input type="checkbox" :value="tag" v-model="selectedTags" />
          <span class="filter-label">{{ tag }}</span>
        </div>
      </div>

      <!-- Filter by Linkink -->
      <div class="filter-div Linkink">
        <label>
          <input type="checkbox" v-model="filterLinkink" />
          <span class="filter-label">linkink</span>
        </label>
      </div>
    </div>

    <!-- Modal for Feature List or Feature Details -->
    <div v-if="showModal" class="modal" @click="closeModal">
      <div class="modal-content" @click.stop>
        <span class="close-button" @click="closeModal">&times;</span>

        <!-- Feature Details View -->
        <div v-if="selectedFeature">
          <button @click="backToList">Обратно к списку</button>
          <h3>{{ selectedFeature.properties.name || 'Без названия' }}</h3>

          <!-- Display Image -->
          <div v-if="selectedFeature.properties.imageFilename">
            <img
              :src="getImageUrl(selectedFeature.properties.imageFilename)"
              alt="Feature Image"
              class="feature-image"
            />
          </div>

          <!-- Display Linkink -->
          <div v-if="selectedFeature.properties.linkink">
            <a :href="selectedFeature.properties.linkink" target="_blank">
              {{ selectedFeature.properties.linkink }}
            </a>
          </div>

          <!-- Display Main Content as HTML -->
          <div v-if="selectedFeature.properties.main && selectedFeature.properties.main.length">
            <div
              v-for="(content, index) in selectedFeature.properties.main"
              :key="index"
            >
              <div v-html="content"></div>
            </div>
          </div>

          <!-- Display Address as HTML -->
          <div
            v-if="selectedFeature.properties.address && selectedFeature.properties.address.length"
          >
            <div
              v-for="(addr, index) in selectedFeature.properties.address"
              :key="index"
            >
              <div v-html="addr"></div>
            </div>
          </div>

          <!-- Display Clauses -->
          <div
            v-if="selectedFeature.properties.clauses && selectedFeature.properties.clauses.length"
          >
            <ul>
              <li v-for="clause in selectedFeature.properties.clauses" :key="clause">
                {{ clause }}
              </li>
            </ul>
          </div>

          <!-- Display Tags -->
          <div
            v-if="selectedFeature.properties.tags && selectedFeature.properties.tags.length"
          >
            <ul>
              <li v-for="tag in selectedFeature.properties.tags" :key="tag">
                {{ tag }}
              </li>
            </ul>
          </div>

          <!-- Display Source URL -->
          <div v-if="selectedFeature.properties.sourceUrl">
            <a :href="selectedFeature.properties.sourceUrl" target="_blank">link</a>
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
              {{ feature.properties.name || 'Без названия' }}
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
import noUiSlider from 'nouislider';
import 'nouislider/dist/nouislider.css';
import debounce from 'lodash/debounce';

export default {
  name: 'App',
  data() {
    return {
      map: null,
      geojsonData: null,
      showModal: false,
      currentFeatures: [],
      clauses: [],
      // Hardcoded Tags as per user's request
      tags: [
        'дела гражданских активистов',
        'дела журналистов',
        'дела мусульман',
        'дела о «шпионаже»',
        'дела «украинских диверсантов»',
        'дела о государственной измене',
        'дело антифашистов',
        'преследование адвокатов',
        'преследование крымских татар',
        'преследование свидетелей Иеговы',
        'преследование сторонников Навального',
        'преследование украинцев',
        'преследования по религиозному признаку',
        'принудительное лечение + карательная психиатрия',
        'репрессии за антивоенную позицию',
        '«экстремистские» высказывания + антиэкстремистское законодательство',
        'ЛГБТ',
        'Хизб ут-Тахрир',
        'антитеррористическое законодательство',
        'оправдание терроризма',
      ],
      names: [],
      selectedClauses: [],
      selectedTags: [],
      filtersVisible: false,
      displayedFeatureCount: 0,
      selectedFeature: null,
      filterLinkink: false,
      allDates: [], // Array to store all valid dates
      minDate: null, // Earliest date
      maxDate: null, // Latest date
      selectedStartDate: null, // Start of the selected date range
      selectedEndDate: null, // End of the selected date range
      dateSlider: null, // Reference to the slider
      isPlaying: false,
      playInterval: null,
      previousCoordinates: new Set(), // Set to track previously displayed coordinates
      activeFilterCategory: null, // 'clauses' or 'tags'
    };
  },
  computed: {
    formattedStartDate() {
      return this.selectedStartDate
        ? this.selectedStartDate.toLocaleDateString('ru-RU', { year: 'numeric', month: 'long' })
        : '';
    },
    formattedEndDate() {
      return this.selectedEndDate
        ? this.selectedEndDate.toLocaleDateString('ru-RU', { year: 'numeric', month: 'long' })
        : '';
    },
  },
  mounted() {
    // Initialize Mapbox
    mapboxgl.accessToken = 'pk.eyJ1IjoidnVmb3JpYSIsImEiOiJjbTJybXJiaWsxOHVnMmpzYnZtZWk4ZXB1In0.24OWriNL8SBYXoLdBtO9EA'; // Replace with your Mapbox access token
    this.map = new mapboxgl.Map({
      container: this.$refs.mapContainer,
      style: 'mapbox://styles/mapbox/navigation-night-v1',
      center: [81.755133, 59.08195], // Adjust center as needed
      zoom: 4,
    });
    this.map.addControl(
      new MapboxLanguage({
        defaultLanguage: 'ru',
      })
    );

    this.map.on('load', () => {
      // Load GeoJSON data
      fetch('/main783Date_Geo.geojson') // Ensure the GeoJSON file is in the public directory
        .then((response) => response.json())
        .then((data) => {
          this.geojsonData = data;

          // Process data and extract unique clauses
          this.processFeaturesByCoordinates();

          // Initialize the date slider after data is processed
          this.initializeDateSlider();

          // Add markers to the map
          this.addMarkers();
        })
        .catch((error) => {
          console.error('Error loading GeoJSON data:', error);
        });
    });

    // Add global click listener to handle clicks outside the Filters Panel
    document.addEventListener('click', this.handleClickOutside);
  },
  beforeUnmount() {
    // Remove the global click listener when component is unmounted
    document.removeEventListener('click', this.handleClickOutside);
  },
  methods: {
    toggleFilters() {
      this.filtersVisible = !this.filtersVisible;
    },
    setActiveFilter(category) {
      if (this.activeFilterCategory === category) {
        this.activeFilterCategory = null; // Toggle off if already active
      } else {
        this.activeFilterCategory = category;
      }
      this.filtersVisible = true; // Show filters panel when a category is selected
    },
    processFeaturesByCoordinates() {
      const clauseSet = new Set();
      // No need to collect tags dynamically as they're hardcoded
      const nameSet = new Set();
      this.allDates = []; // Initialize an array to store all valid dates

      this.geojsonData.features.forEach((feature) => {
        // Pre-parse date
        feature.parsedDate = feature.properties.date ? new Date(feature.properties.date) : null;
        if (feature.parsedDate && !isNaN(feature.parsedDate)) {
          this.allDates.push(feature.parsedDate);
        } else {
          feature.parsedDate = null;
        }

        // Collect names
        if (feature.properties.name) {
          nameSet.add(feature.properties.name);
        }

        // Collect clauses
        if (Array.isArray(feature.properties.clauses)) {
          feature.properties.clauses.forEach((clause) => clauseSet.add(clause));
        }
        // Tags are hardcoded; no need to collect from data
      });

      this.names = Array.from(nameSet).sort();
      this.clauses = Array.from(clauseSet).sort();
      // 'tags' are already defined in the data section

      // Determine the min and max dates
      if (this.allDates.length > 0) {
        this.minDate = new Date(Math.min(...this.allDates));
        this.maxDate = new Date(Math.max(...this.allDates));
      } else {
        // Fallback to current date if no dates are available
        this.minDate = new Date();
        this.maxDate = new Date();
      }

      // Initialize selected date range to the full range
      this.selectedStartDate = new Date(this.minDate);
      this.selectedEndDate = new Date(this.maxDate);

      // Set the initial displayed feature count
      this.displayedFeatureCount = this.geojsonData.features.length;
    },
    initializeDateSlider() {
      // Ensure minDate and maxDate are available
      if (!this.minDate || !this.maxDate) {
        console.warn('No valid dates available for the slider.');
        return;
      }

      const sliderElement = document.getElementById('date-slider');

      // Initialize NoUiSlider
      noUiSlider.create(sliderElement, {
        start: [this.minDate.getTime(), this.maxDate.getTime()],
        connect: true,
        range: {
          min: this.minDate.getTime(),
          max: this.maxDate.getTime(),
        },
        step: 30 * 24 * 60 * 60 * 1000, // Approx. one month in milliseconds
        tooltips: false,
        format: {
          to: function (value) {
            return Number(value);
          },
          from: function (value) {
            return Number(value);
          },
        },
      });

      // Store the slider instance
      this.dateSlider = sliderElement.noUiSlider;

      // Debounce the applyDateFilter method
      const debouncedApplyDateFilter = debounce(this.applyDateFilter, 300);

      // Listen to slider updates
      this.dateSlider.on('update', (values) => {
        const startTimestamp = Number(values[0]);
        const endTimestamp = Number(values[1]);

        this.selectedStartDate = new Date(startTimestamp);
        this.selectedEndDate = new Date(endTimestamp);

        // Apply date range filter with debounce
        debouncedApplyDateFilter();
      });
    },
    applyDateFilter() {
      // Re-add markers to the map
      this.addMarkers();
    },
    applyFilters() {
      // Re-add markers to the map
      this.addMarkers();
      // Hide the Filters Panel
      this.filtersVisible = false;
    },
    resetFilters() {
      // Reset all filter selections
      this.selectedClauses = [];
      this.selectedTags = [];
      this.filterLinkink = false;

      // Reset date slider to full range
      if (this.dateSlider) {
        this.dateSlider.set([this.minDate.getTime(), this.maxDate.getTime()]);
      }
      this.selectedStartDate = new Date(this.minDate);
      this.selectedEndDate = new Date(this.maxDate);

      // Apply filters to refresh the map
      this.applyFilters();
    },
    addMarkers() {
      // Prepare filtered GeoJSON data
      const geoJSONData = this.getFilteredGeoJSON();

      // Add the custom marker image if it hasn't been added yet
      if (!this.map.hasImage('custom-marker')) {
        this.map.loadImage('/custom-marker.png', (error, image) => {
          if (error) throw error;
          this.map.addImage('custom-marker', image);

          // Now add the layer
          this.addSymbolLayer(geoJSONData);
        });
      } else {
        this.addSymbolLayer(geoJSONData);
      }
    },
    addSymbolLayer(geoJSONData) {
      // Remove existing layer and source if they exist
      if (this.map.getLayer('custom-markers')) {
        this.map.removeLayer('custom-markers');
      }
      if (this.map.getSource('features')) {
        this.map.removeSource('features');
      }

      // Add a GeoJSON source
      this.map.addSource('features', {
        type: 'geojson',
        data: geoJSONData,
        generateId: true, // Ensures each feature has a unique ID
      });

      // Add a Symbol layer
      this.map.addLayer({
        id: 'custom-markers',
        type: 'symbol',
        source: 'features',
        layout: {
          'icon-image': 'custom-marker',
          'icon-size': [
            'interpolate',
            ['linear'],
            ['get', 'point_count'],
            1, 0.5,
            5, 1,
            10, 1.5,
          ],
          'icon-allow-overlap': true,
        },
        paint: {
          'icon-opacity': [
            'case',
            ['boolean', ['get', 'isNew'], false],
            ['interpolate', ['linear'], ['get', 'flicker'], 0, 0, 0.5, 1],
            1,
          ],
        },
      });

      // Add click event for the markers
      this.map.on('click', 'custom-markers', (e) => {
        const features = this.map.queryRenderedFeatures(e.point, {
          layers: ['custom-markers'],
        });
        const clickedFeature = features[0];
        if (clickedFeature) {
          const properties = clickedFeature.properties;
          const featureList = JSON.parse(properties.features);
          this.showFeatureList(featureList);
        }
      });

      // Change the cursor to a pointer when over the markers
      this.map.on('mouseenter', 'custom-markers', () => {
        this.map.getCanvas().style.cursor = 'pointer';
      });
      this.map.on('mouseleave', 'custom-markers', () => {
        this.map.getCanvas().style.cursor = '';
      });
    },
    getFilteredGeoJSON() {
      // Filter features based on the selected criteria
      let filteredFeatures = this.geojsonData.features.filter((feature) => {
        // Skip items with no date
        if (!feature.parsedDate) return false;

        // Date Filter
        const inDateRange =
          feature.parsedDate >= this.selectedStartDate &&
          feature.parsedDate <= this.selectedEndDate;

        // Clause Filter
        const matchesClause = this.selectedClauses.length
          ? feature.properties.clauses.some((clause) =>
              this.selectedClauses.includes(clause)
            )
          : true;

        // Tag Filter
        const matchesTag = this.selectedTags.length
          ? feature.properties.tags.some((tag) => this.selectedTags.includes(tag))
          : true;

        // Linkink Filter
        const matchesLinkink = this.filterLinkink
          ? feature.properties.linkink && feature.properties.linkink.trim() !== ''
          : true;

        return inDateRange && matchesClause && matchesTag && matchesLinkink;
      });

      // Group features by coordinates
      const featuresByCoordinates = {};
      filteredFeatures.forEach((feature) => {
        const coords = feature.geometry.coordinates.join(',');
        if (!featuresByCoordinates[coords]) {
          featuresByCoordinates[coords] = [];
        }
        featuresByCoordinates[coords].push(feature);
      });

      // Prepare GeoJSON data
      const geoJSON = {
        type: 'FeatureCollection',
        features: [],
      };

      const currentCoordinates = new Set();

      Object.keys(featuresByCoordinates).forEach((coords) => {
        const [lng, lat] = coords.split(',').map(Number);
        const features = featuresByCoordinates[coords];

        const isNew = !this.previousCoordinates.has(coords);

        const pointFeature = {
          type: 'Feature',
          geometry: {
            type: 'Point',
            coordinates: [lng, lat],
          },
          properties: {
            point_count: features.length,
            features: JSON.stringify(features), // Store features as string for properties
            isNew: isNew,
            flicker: isNew ? Math.random() : 1, // Apply flicker only to new features
          },
        };

        geoJSON.features.push(pointFeature);

        // Add to current coordinates
        currentCoordinates.add(coords);
      });

      // Update previousCoordinates
      this.previousCoordinates = currentCoordinates;

      // Update displayed feature count
      this.displayedFeatureCount = filteredFeatures.length;

      return geoJSON;
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
    getImageUrl(filename) {
      // Assuming images are stored in the 'public/images' folder
      return `/images/${filename}`;
    },
    togglePlay() {
      if (this.isPlaying) {
        this.pauseSlider();
      } else {
        this.playSlider();
      }
    },
    playSlider() {
      this.isPlaying = true;
      const step = 30 * 24 * 60 * 60 * 1000; // One month in milliseconds
      this.playInterval = setInterval(() => {
        let [start, end] = this.dateSlider.get().map(Number);

        if (start + step <= end) {
          start += step;
          this.dateSlider.set([start, end]);
        } else {
          this.pauseSlider(); // Stop playing when the start reaches the end
        }
      }, 1000); // Adjust the interval speed as desired
    },
    pauseSlider() {
      this.isPlaying = false;
      if (this.playInterval) {
        clearInterval(this.playInterval);
        this.playInterval = null;
      }
    },
    handleClickOutside(event) {
      const filtersPanel = this.$refs.filtersPanel;
      const filtersToggleContainer = this.$refs.filtersToggleContainer;

      if (
        filtersPanel &&
        !filtersPanel.contains(event.target) &&
        filtersToggleContainer &&
        !filtersToggleContainer.contains(event.target)
      ) {
        this.filtersVisible = false;
        this.activeFilterCategory = null; // Optional: Reset active filter category
      }
    },
  },
};
</script>

<style>
/* ... existing styles ... */

/* Adjusted styles for date slider container */
.date-slider-container {
  position: absolute;
  top: 10px;
  left: 110px;
  z-index: 15;
  background-color: rgba(255, 255, 255, 0.9);
  padding: 10px;
  border-radius: 4px;
  width: 350px;
}

.selected-dates {
  margin-top: 10px;
  font-size: 14px;
  color: #000;
  display: flex;
  justify-content: space-between;
}

.play-pause-controls {
  margin-top: 10px;
  text-align: center;
}

.play-pause-controls button {
  padding: 6px 12px;
  background-color: #51bbd6;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.play-pause-controls button:hover {
  background-color: #429ab8;
}
 
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
  width: 20px;
  height: 20px;
  cursor: pointer;
  opacity: 0.6 !important;
}

/* Marker classes based on the number of linked entries */
.marker-single {
    /* Replace with your single-entry marker icon */
}

.marker-few {
    /* Replace with your 2-5 entries marker icon */
}

.marker-many {
   /* Replace with your >5 entries marker icon */
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
  min-width: 400px
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
/* Top-Left Filter Controls */
.filters-toggle-container {
  position: absolute;
  top: 10px;
  left: 10px;
  z-index: 16; 
  gap: 10px;
}

.filters-toggle-container .feature-count {
  display: flex;
  align-items: center;
  background-color: rgba(0, 0, 0, 0.6);
  padding: 8px 12px;
  border-radius: 4px;
  color: white;
  width: 75px;
  float: left;
  position: relative;
  right: auto;
  top: auto;
}

.filters-toggle-container .feature-count img {
  width: 30px;
  margin-right: 8px;
}

.filters-toggle-container .filters-toggle {
  padding: 8px 12px;
  background-color: rgba(255, 255, 255, 0.9);
  border: none;
  border-radius: 4px;
  cursor: pointer;
  text-align: left;
}

.filters-toggle-container .filters-toggle.active {
  background-color: #51bbd6;
  color: white;
}

.filters-toggle-container .filters-toggle:hover {
  background-color: #429ab8;
  color: white;
}

/* Feature Count Display */
.feature-count {
  position: absolute;
  top: 10px;
  right: 10px;
  z-index: 16;
  background-color: rgba(0, 0, 0, 0.6);
  padding: 8px 12px;
  border-radius: 4px;
  display: flex;
  align-items: center;
  color: white;
}

.feature-count img {
  width: 30px;
  margin-right: 8px;
}

/* Top-level Filter Buttons */
.filter-buttons-top {
  position: absolute;
  top: 10px;
  left: 10px;
  z-index: 16;
  display: flex;
  gap: 10px;
}

.filter-buttons-top button {
  padding: 8px 12px;
  background-color: rgba(255, 255, 255, 0.9);
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.filter-buttons-top button.active {
  background-color: #51bbd6;
  color: white;
}

.filter-buttons-top button:hover {
  background-color: #429ab8;
  color: white;
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
  position: relative;
  top: 10px;
  left: 10px;
  z-index: 15;
  background-color: rgba(255, 255, 255, 0.9);
  border: none;
  padding: 8px 12px;
  cursor: pointer;
  border-radius: 4px;
  float: left;
  margin-right: 20px;
}


/* Filters panel styles */
.filters {
  position: absolute;
  top: 80px; /* Adjusted to be below the toggle button */
  left: 10px;
  z-index: 10;
  padding: 0;
  /* ... other styles ... */
}
.filters > div{
  padding: 10px;
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

#app .date-slider-container{
  right: 0 !important;
  left: auto !important;
  background-color: rgba(0,0,0,0.6) !important;
}
#app .date-slider-container *{
  color: white !important;
}
.noUi-connect, .play-pause-controls button{
  background-color: rgba(0,0,0,0.6) !important;
}

.Linkink{
  display: none;
}
.filters-toggle-container .filters-toggle.active, .filters-toggle-container .filters-toggle:hover{
  background-color: black !important;
}
</style>