<template>
  <div id="app">
    <!-- Map Container -->
    <div ref="mapContainer" class="map-container"></div>


      <!-- Zoom Controls -->
      <div class="zoom-controls">
      <button class="zoom-button" @click="zoomIn">+</button>
      <div class="zoom-slider-container">
        <div ref="zoomSlider" class="zoom-slider"></div>
      </div>
      <button class="zoom-button" @click="zoomOut">−</button>
    </div>


    <!-- Top-Left Filter Controls -->
    <div class="filters-toggle-container" ref="filtersToggleContainer">
      <!-- Feature Count -->
      <div class="feature-count">
        <img src="/custom-marker.png" alt="Custom Marker" />
        {{ displayedFeatureCount }}
      </div>
      <!-- Search input -->
      <input
        v-model="searchQuery"
        @input="handleSearch"
        @keyup.enter="handleSearchEnter"
        placeholder="Поиск по имени или статье"
        class="search-input"
      />

      <!-- Filter buttons -->
      <button
        class="filters-toggle"
        @click="setActiveFilter('names')"
        :class="{
          'active-panel': activeFilterCategory === 'names',
          'active-filter': hasActiveNameFilters
        }"
      >
        Имена
      </button>

      <button
        class="filters-toggle"
        @click="setActiveFilter('clauses')"
        :class="{
          'active-panel': activeFilterCategory === 'clauses',
          'active-filter': hasActiveClauseFilters
        }"
      >
        Статьи
      </button>

      <button
        class="filters-toggle"
        @click="setActiveFilter('tags')"
        :class="{
          'active-panel': activeFilterCategory === 'tags',
          'active-filter': hasActiveTagFilters
        }"
      >
        Дела
      </button>

      <button 
        class="filters-toggle" 
        @click="resetFilters"
        :class="{
          'active-filter': hasAnyActiveFilters
        }"
      >
        Сбросить
      </button>
    </div>

    <!-- Date Controls Container -->
    <div class="date-controls-container">
      <div class="date-slider-container" :class="{ disabled: !dateSliderEnabled }" @click="handleSliderContainerClick">
        <!-- Mode Selection Buttons -->
        <div class="date-mode-buttons">
          <button 
            @click="setDateMode('date')"
            :class="{ active: dateMode === 'date' }"
            class="mode-button"
          >
            Дата
          </button>
          <button 
            @click="setDateMode('range')"
            :class="{ active: dateMode === 'range' }"
            class="mode-button"
          >
            Отрезок
          </button>
        </div>

        <div class="date-slider" ref="dateSlider"></div>
        
        <!-- Date Display -->
        <div class="selected-dates">
          <template v-if="dateMode === 'date'">
            <span>{{ formattedCurrentDate }}</span>
          </template>
          <template v-else>
            <span>{{ formattedStartDate }}</span>
            <span>{{ formattedEndDate }}</span>
          </template>
        </div>
      </div>
    </div>



    <!-- Names Filter -->
<div v-if="activeFilterCategory === 'names'">
  <div v-if="filteredNames.length === 0" class="no-results">
    ничего не найденно
  </div>
  <div v-else v-for="name in filteredNames" :key="name" class="filter-div">
    <input 
      type="checkbox" 
      :value="name" 
      v-model="selectedNames" 
      :id="'name-' + name"
    />
    <span 
      class="filter-label" 
      @click="toggleFilter('names', name)"
    >
      {{ name }}
    </span>
  </div>
</div>

<!-- Clauses Filter -->
<div v-if="activeFilterCategory === 'clauses'">
  <div v-if="filteredClauses.length === 0" class="no-results">
    ничего не найденно
  </div>
  <div v-else v-for="clause in filteredClauses" :key="clause" class="filter-div">
    <input 
      type="checkbox" 
      :value="clause" 
      v-model="selectedClauses"
      :id="'clause-' + clause"
    />
    <span 
      class="filter-label" 
      @click="toggleFilter('clauses', clause)"
    >
      {{ clause }}
    </span>
  </div>
</div>

<!-- Tags Filter -->
<div v-if="activeFilterCategory === 'tags'">
  <div v-if="filteredTags.length === 0" class="no-results">
    ничего не найденно
  </div>
  <div v-else v-for="tag in filteredTags" :key="tag" class="filter-div">
    <input 
      type="checkbox" 
      :value="tag" 
      v-model="selectedTags"
      :id="'tag-' + tag"
    />
    <span 
      class="filter-label" 
      @click="toggleFilter('tags', tag)"
    >
      {{ tag }}
    </span>
  </div>
</div>

    <!-- Filters Panel -->
    <div class="filters" v-show="filtersVisible" ref="filtersPanel">
     
      <!-- Names Filter -->
        <div v-if="activeFilterCategory === 'names'">
          <div v-if="filteredNames.length === 0" class="no-results">
            Ничего не найдено
          </div>
          <div v-else v-for="name in filteredNames" :key="name" class="filter-div">
            <input 
              type="checkbox" 
              :value="name" 
              v-model="selectedNames" 
              :id="'name-' + name"
            />
            <span 
              class="filter-label" 
              @click="toggleFilter('names', name)"
            >
              {{ name }}
            </span>
          </div>
        </div>

        <!-- Clauses Filter -->
        <div v-if="activeFilterCategory === 'clauses'">
          <div v-if="filteredClauses.length === 0" class="no-results">
            Ничего не найдено
          </div>
          <div v-else v-for="clause in filteredClauses" :key="clause" class="filter-div">
            <input 
              type="checkbox" 
              :value="clause" 
              v-model="selectedClauses"
              :id="'clause-' + clause"
            />
            <span 
              class="filter-label" 
              @click="toggleFilter('clauses', clause)"
            >
              {{ clause }}
            </span>
          </div>
        </div>

        <!-- Tags Filter -->
        <div v-if="activeFilterCategory === 'tags'">
          <div v-if="filteredTags.length === 0" class="no-results">
            Ничего не найдено
          </div>
          <div v-else v-for="tag in filteredTags" :key="tag" class="filter-div">
            <input 
              type="checkbox" 
              :value="tag" 
              v-model="selectedTags"
              :id="'tag-' + tag"
            />
            <span 
              class="filter-label" 
              @click="toggleFilter('tags', tag)"
            >
              {{ tag }}
            </span>
          </div>
        </div>
      
    </div>

    <!-- Modal -->
    <div v-if="showModal" class="modal" @click="closeModal">
      <div class="modal-content" @click.stop>
        <span class="close-button" @click="closeModal">&times;</span>

        <!-- Feature Details View -->
        <div v-if="selectedFeature">
          <button @click="backToList">Обратно к списку</button>

          <div class="main-info" v-if="selectedFeature.properties.imageFilename">
            
            <h3>{{ selectedFeature.properties.name || 'Без названия' }}</h3>
            <img
              :src="getImageUrl(selectedFeature.properties.imageFilename)"
              alt="Feature Image"
              class="feature-image"
            />
          </div>

          <div v-if="selectedFeature.properties.linkink">
            <a :href="selectedFeature.properties.linkink" target="_blank">
              {{ selectedFeature.properties.linkink }}
            </a>
          </div>

          <div v-if="selectedFeature.properties.main?.length">
            <div
              v-for="(content, index) in selectedFeature.properties.main"
              :key="index"
              v-html="content"
            ></div>
          </div>

          <div v-if="selectedFeature.properties.address?.length">
            <span class="tag-label">Адрес для писем:</span>
            <div
              v-for="(addr, index) in selectedFeature.properties.address"
              :key="index"
              v-html="addr"
            ></div>
          </div>

          <div v-if="selectedFeature.properties.clauses?.length">
            <span class="tag-label">Статьи:</span>
            <ul>
              <li 
                v-for="clause in selectedFeature.properties.clauses" 
                :key="clause"
              >
                {{ clause }}
              </li>
            </ul>
          </div>
<!-- 
          <div v-if="selectedFeature.properties.tags?.length">
            
            <ul>
              <li 
                v-for="tag in selectedFeature.properties.tags" 
                :key="tag"
              >
                {{ tag }}
              </li>
            </ul>
          </div>

          -->

          <div v-if="selectedFeature.properties.sourceUrl">
            <a :href="selectedFeature.properties.sourceUrl" target="_blank">
              Поддержка политзаключённых. Мемориал
            </a>
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
      selectedFeature: null,
      clauses: [],
      names: [],
      searchQuery: '',
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
      selectedClauses: [],
      selectedNames: [],
      selectedTags: [],
      filtersVisible: false,
      activeFilterCategory: null,
      displayedFeatureCount: 0,
      dateSlider: null,
      previousCoordinates: new Set(),
      dateMode: 'date',
      dateSliderEnabled: false,
      currentDate: new Date(),
      startDate: null,
      endDate: null,
      minDate: null,
      maxDate: null,
    };
  },
  computed: {
    formattedCurrentDate() {
      return this.currentDate?.toLocaleDateString('ru-RU', { 
        year: 'numeric', 
        month: 'long' 
      });
    },
    formattedStartDate() {
      return this.startDate?.toLocaleDateString('ru-RU', { 
        year: 'numeric', 
        month: 'long' 
      });
    },
    formattedEndDate() {
      return this.endDate?.toLocaleDateString('ru-RU', { 
        year: 'numeric', 
        month: 'long' 
      });
    },
    hasActiveClauseFilters() {
      return this.selectedClauses.length > 0;
    },
    hasActiveTagFilters() {
      return this.selectedTags.length > 0;
    },
    hasActiveNameFilters() {
      return this.selectedNames.length > 0;
    },
    hasAnyActiveFilters() {
      return this.hasActiveClauseFilters || 
             this.hasActiveTagFilters || 
             this.hasActiveNameFilters || 
             this.dateSliderEnabled;
    },
    filteredNames() {
      if (!this.searchQuery) return this.names;
      const query = this.searchQuery.toLowerCase();
      return this.names.filter(name => 
        name.toLowerCase().includes(query)
      );
    },
    filteredClauses() {
      if (!this.searchQuery) return this.clauses;
      const query = this.searchQuery.toLowerCase();
      return this.clauses.filter(clause => 
        clause.toLowerCase().includes(query)
      );
    },
    filteredTags() {
      if (!this.searchQuery) return this.tags;
      const query = this.searchQuery.toLowerCase();
      return this.tags.filter(tag => 
        tag.toLowerCase().includes(query)
      );
    }
  },
  watch: {
    selectedClauses() {
      this.applyFilters();
    },
    selectedTags() {
      this.applyFilters();
    },
    selectedNames() {
      this.applyFilters();
    }
  },
  mounted() {
    this.initializeMap();
    document.addEventListener('click', this.handleClickOutside);
  },
  beforeUnmount() {
    document.removeEventListener('click', this.handleClickOutside);
    if (this.dateSlider) {
      this.dateSlider.destroy();
    }
  },
  methods: {
    initializeMap() {

      
mapboxgl.accessToken = 'pk.eyJ1IjoidnVmb3JpYSIsImEiOiJjbTJybXJiaWsxOHVnMmpzYnZtZWk4ZXB1In0.24OWriNL8SBYXoLdBtO9EA'; // Replace with your Mapbox access token
   
      this.map = new mapboxgl.Map({
        container: this.$refs.mapContainer,
        style: 'mapbox://styles/mapbox/navigation-night-v1',
        center: [96.712933,62.917018],
        zoom: 5,
      });

      this.map.addControl(
        new MapboxLanguage({
          defaultLanguage: 'ru',
        })
      );

      this.map.on('load', () => {
        this.loadGeoJSONData();
        this.initializeZoomSlider();
      });
    },

    async loadGeoJSONData() {
      try {
        const response = await fetch('/main783Date_Geo.geojson');
        this.geojsonData = await response.json();
        
        // Load images first
        await Promise.all([
          new Promise((resolve, reject) => {
            this.map.loadImage('/custom-marker.png', (error, image) => {
              if (error) reject(error);
              if (!this.map.hasImage('custom-marker')) {
                this.map.addImage('custom-marker', image);
              }
              resolve();
            });
          }),
          new Promise((resolve, reject) => {
            this.map.loadImage('/custom-marker-hover.png', (error, image) => {
              if (error) reject(error);
              if (!this.map.hasImage('custom-marker-hover')) {
                this.map.addImage('custom-marker-hover', image);
              }
              resolve();
            });
          })
        ]);

        this.processFeaturesByCoordinates();
        this.initializeDateSlider();
        this.addMarkers();
      } catch (error) {
        console.error('Error loading GeoJSON data:', error);
      }
    },

    processFeaturesByCoordinates() {
      if (!this.geojsonData) return;

      const clauseSet = new Set();
      const nameSet = new Set();
      const dates = [];

      this.geojsonData.features.forEach((feature) => {
        if (feature.properties.date) {
          const parsedDate = new Date(feature.properties.date);
          if (!isNaN(parsedDate)) {
            dates.push(parsedDate);
            feature.parsedDate = parsedDate;
          }
        }

        if (Array.isArray(feature.properties.clauses)) {
          feature.properties.clauses.forEach((clause) => clauseSet.add(clause));
        }

        if (feature.properties.name) {
          nameSet.add(feature.properties.name);
        }
      });

      if (dates.length > 0) {
        this.minDate = new Date(Math.min(...dates));
        this.maxDate = new Date(Math.max(...dates));
        
        const today = new Date();
        this.currentDate = today > this.maxDate ? this.maxDate : today;
        
        this.startDate = this.minDate;
        this.endDate = this.maxDate;
      }

      this.clauses = Array.from(clauseSet).sort();
      this.names = Array.from(nameSet).sort();
      this.displayedFeatureCount = this.geojsonData.features.length;
    },
    initializeZoomSlider() {
      const zoomSliderElement = this.$refs.zoomSlider;
      
      if (!zoomSliderElement || !this.map) return;

      if (this.zoomSlider) {
        this.zoomSlider.destroy();
      }

      const minZoom = this.map.getMinZoom();
      const maxZoom = this.map.getMaxZoom();
      const currentZoom = this.map.getZoom();

      this.zoomSlider = noUiSlider.create(zoomSliderElement, {
        start: [currentZoom],
        orientation: 'vertical',
        direction: 'rtl',
        range: {
          'min': minZoom,
          'max': maxZoom
        },
        step: 0.5,
        tooltips: false
      });

      this.zoomSlider.on('slide', (values) => {
        const zoom = parseFloat(values[0]);
        this.map.easeTo({ zoom: zoom });
      });

      // Update slider when map is zoomed by other means
      this.map.on('zoom', () => {
        const currentZoom = this.map.getZoom();
        this.zoomSlider.set([currentZoom]);
      });
    },

    zoomIn() {
      const currentZoom = this.map.getZoom();
      this.map.easeTo({ zoom: currentZoom + 1 });
    },

    zoomOut() {
      const currentZoom = this.map.getZoom();
      this.map.easeTo({ zoom: currentZoom - 1 });
    },
    handleSearch() {
      if (!this.searchQuery) {
        return;
      }

      const isNumeric = /^\d/.test(this.searchQuery);
      const category = isNumeric ? 'clauses' : 'names';
      
      // Always show filters panel and set active category when searching
      this.filtersVisible = true;
      this.activeFilterCategory = category;
    },
    toggleFilter(type, value) {
      switch(type) {
        case 'names':
          // Create a new array to trigger reactivity
          if (this.selectedNames.includes(value)) {
            this.selectedNames = this.selectedNames.filter(name => name !== value);
          } else {
            this.selectedNames = [...this.selectedNames, value];
          }
          break;
        case 'clauses':
          if (this.selectedClauses.includes(value)) {
            this.selectedClauses = this.selectedClauses.filter(clause => clause !== value);
          } else {
            this.selectedClauses = [...this.selectedClauses, value];
          }
          break;
        case 'tags':
          if (this.selectedTags.includes(value)) {
            this.selectedTags = this.selectedTags.filter(tag => tag !== value);
          } else {
            this.selectedTags = [...this.selectedTags, value];
          }
          break;
      }
      // The filters will now automatically apply due to the watchers on selectedNames, selectedClauses, and selectedTags
    },
    handleSearchEnter() {
      const isNumeric = /^\d/.test(this.searchQuery);
      const items = isNumeric ? this.filteredClauses : this.filteredNames;
      
      if (items.length === 1) {
        const result = items[0];
        if (isNumeric) {
          // Add to existing selection instead of replacing
          if (!this.selectedClauses.includes(result)) {
            this.selectedClauses.push(result);
          }
        } else {
          // Add to existing selection instead of replacing
          if (!this.selectedNames.includes(result)) {
            this.selectedNames.push(result);
          }
        }
        
        this.clearSearch();
        this.closeFiltersPanel();
        this.applyFilters();
      }
    },

    clearSearch() {
      this.searchQuery = '';
    },

    closeFiltersPanel() {
      this.filtersVisible = false;
      this.activeFilterCategory = null;
    },

    setActiveFilter(category) {
      if (this.activeFilterCategory === category) {
        this.closeFiltersPanel();
      } else {
        this.activeFilterCategory = category;
        this.filtersVisible = true;
        this.clearSearch();
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
        // Use closeFiltersPanel to ensure both panel is closed and active-panel is removed
        this.closeFiltersPanel();
      }
    },

    handleSliderContainerClick(event) {
      if (event.target.classList.contains('noUi-connects')) {
        this.dateSliderEnabled = true;
        this.applyFilters();
      }
    },

    initializeDateSlider() {
      const sliderElement = this.$refs.dateSlider;
      
      if (!sliderElement || !this.geojsonData) return;

      if (this.dateSlider) {
        this.dateSlider.destroy();
      }

      const initialConfig = {
        start: this.dateMode === 'date' 
          ? [this.currentDate.getTime()]
          : [this.startDate.getTime(), this.endDate.getTime()],
        connect: true,
        range: {
          min: this.minDate.getTime(),
          max: this.maxDate.getTime()
        },
        step: 24 * 60 * 60 * 1000,
        tooltips: false
      };

      this.dateSlider = noUiSlider.create(sliderElement, initialConfig);

      this.dateSlider.on('start', () => {
        if (!this.dateSliderEnabled) {
          this.dateSliderEnabled = true;
        }
      });

      const debouncedUpdate = debounce(this.updateFromSlider, 300);
      this.dateSlider.on('update', debouncedUpdate);
    },

    updateFromSlider(values) {
      if (this.dateMode === 'date') {
        this.currentDate = new Date(Number(values[0]));
      } else {
        this.startDate = new Date(Number(values[0]));
        this.endDate = new Date(Number(values[1]));
      }
      this.applyFilters();
    },

    setDateMode(mode) {
      if (mode === this.dateMode) {
        this.dateSliderEnabled = !this.dateSliderEnabled;
      } else {
        this.dateMode = mode;
        this.dateSliderEnabled = true;
      }
      this.resetDateSlider();
    },

    resetDateSlider() {
      this.initializeDateSlider();
      this.applyFilters();
    },

  getFilteredFeatures() {
      return this.geojsonData.features.filter((feature) => {
        // First check if dates are valid when any filter is active
        if (this.hasAnyActiveFilters && !feature.parsedDate) {
          return false;
        }

        // Then apply all other filters
        if (this.hasActiveNameFilters && 
            !this.selectedNames.includes(feature.properties.name)) {
          return false;
        }

        if (this.hasActiveClauseFilters && 
            !feature.properties.clauses?.some(clause => 
              this.selectedClauses.includes(clause))) {
          return false;
        }

        if (this.hasActiveTagFilters && 
            !feature.properties.tags?.some(tag => 
              this.selectedTags.includes(tag))) {
          return false;
        }

        // Date filtering
        if (this.dateSliderEnabled && feature.parsedDate) {
          if (this.dateMode === 'date') {
            return feature.parsedDate <= this.currentDate;
          } else {
            return feature.parsedDate >= this.startDate && 
                  feature.parsedDate <= this.endDate;
          }
        }

        return true;
      });
    },

    applyFilters() {
      const filteredFeatures = this.getFilteredFeatures();
      this.displayedFeatureCount = filteredFeatures.length;
      this.addMarkers(filteredFeatures);
    },

    resetFilters() {
      this.selectedClauses = [];
      this.selectedTags = [];
      this.selectedNames = [];
      this.filtersVisible = false;
      this.activeFilterCategory = null;
      this.dateSliderEnabled = false;
      this.clearSearch();
      
      if (this.minDate && this.maxDate) {
        this.startDate = this.minDate;
        this.endDate = this.maxDate;
        this.currentDate = this.maxDate;
      }
      
      this.resetDateSlider();
      this.applyFilters();
    },

     async loadMarkerImages() {
      return Promise.all([
        new Promise((resolve, reject) => {
          this.map.loadImage('/custom-marker.png', (error, image) => {
            if (error) reject(error);
            if (!this.map.hasImage('custom-marker')) {
              this.map.addImage('custom-marker', image);
            }
            resolve();
          });
        }),
        new Promise((resolve, reject) => {
          this.map.loadImage('/custom-marker-hover.png', (error, image) => {
            if (error) reject(error);
            if (!this.map.hasImage('custom-marker-hover')) {
              this.map.addImage('custom-marker-hover', image);
            }
            resolve();
          });
        })
      ]);
    },
    addMarkers(filteredFeatures = null) {
    const features = filteredFeatures || this.geojsonData.features;
    const geoJSON = this.prepareGeoJSONForMarkers(features);

    if (!this.map.getSource('features')) {
      // Initialize source and layers for first time
      this.map.addSource('features', {
        type: 'geojson',
        data: geoJSON,
        generateId: true
      });

      // Base marker layer
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
            3, 0.8,
            7, 1
          ],
          'icon-allow-overlap': true
        },
        paint: {
          'icon-opacity': [
            'case',
            ['boolean', ['feature-state', 'hover'], false],
            0,  // When hovered, this layer is invisible
            ['number', ['get', 'opacity'], 0.3]  // Use the feature property for opacity
          ]
        }
      });

      // Hover layer
      this.map.addLayer({
        id: 'custom-markers-hover',
        type: 'symbol',
        source: 'features',
        layout: {
          'icon-image': 'custom-marker-hover',
          'icon-size': 1.2,
          'icon-allow-overlap': true
        },
        paint: {
          'icon-opacity': [
            'case',
            ['boolean', ['feature-state', 'hover'], false],
            1,  // When hovered, this layer is visible
            0   // When not hovered, this layer is invisible
          ]
        }
      });

      // Add hover effects
      let hoveredStateId = null;
      
      this.map.on('mousemove', 'custom-markers', (e) => {
        if (e.features.length > 0) {
          if (hoveredStateId !== null) {
            this.map.setFeatureState(
              { source: 'features', id: hoveredStateId },
              { hover: false }
            );
          }
          hoveredStateId = e.features[0].id;
          this.map.setFeatureState(
            { source: 'features', id: hoveredStateId },
            { hover: true }
          );
          this.map.getCanvas().style.cursor = 'pointer';
        }
      });

      this.map.on('mouseleave', 'custom-markers', () => {
        if (hoveredStateId !== null) {
          this.map.setFeatureState(
            { source: 'features', id: hoveredStateId },
            { hover: false }
          );
          hoveredStateId = null;
          this.map.getCanvas().style.cursor = '';
        }
      });

      // Add click handlers
      ['custom-markers', 'custom-markers-hover'].forEach(layerId => {
        this.map.on('click', layerId, (e) => {
          if (e.features.length > 0) {
            const features = JSON.parse(e.features[0].properties.features);
            this.showFeatureList(features);
          }
        });
      });

      // Animate initial markers
      setTimeout(() => {
        this.animateInitialMarkers(geoJSON.features);
      }, 100);
    } else {
      // Update existing source
      const source = this.map.getSource('features');
      if (source) {
        source.setData(geoJSON);
        this.animateNewMarkers(geoJSON.features);
      }
    }
  },

  prepareGeoJSONForMarkers(features) {
    const featuresByCoordinates = {};
    const currentSource = this.map.getSource('features');
    const existingCoords = currentSource ? 
      new Set(currentSource.serialize().data.features.map(f => f.geometry.coordinates.join(','))) :
      new Set();
    
    features.forEach((feature) => {
      const coords = feature.geometry.coordinates.join(',');
      if (!featuresByCoordinates[coords]) {
        featuresByCoordinates[coords] = [];
      }
      featuresByCoordinates[coords].push(feature);
    });

    return {
      type: 'FeatureCollection',
      features: Object.entries(featuresByCoordinates).map(([coords, groupedFeatures]) => {
        const [lng, lat] = coords.split(',').map(Number);
        const isNew = !existingCoords.has(coords);
        
        return {
          type: 'Feature',
          geometry: {
            type: 'Point',
            coordinates: [lng, lat]
          },
          properties: {
            point_count: groupedFeatures.length,
            features: JSON.stringify(groupedFeatures),
            isNew: isNew,
            opacity: 1  // Start with full opacity
          }
        };
      })
    };
  },

  animateInitialMarkers(features) {
    features.forEach((feature) => {
      setTimeout(() => {
        this.animateMarker(feature);
      }, Math.random() * 3000); // Random delay up to 2 seconds
    });
  },

  animateNewMarkers(features) {
    features
      .filter(f => f.properties.isNew)
      .forEach((feature) => {
        setTimeout(() => {
          this.animateMarker(feature);
        }, Math.random() * 3000); // Random delay up to 1 second for new markers
      });
  },

  animateMarker(feature) {
    const source = this.map.getSource('features');
    if (!source) return;

    const animationSteps = [
      { opacity: 0.2, duration: 0 },    
      { opacity: 0.8, duration: 300 },  
      { opacity: 0.4, duration: 600 },    
      { opacity: 0.5, duration: 900 }
    ];

    const animate = async (step = 0) => {
      if (step >= animationSteps.length) return;

      const currentStep = animationSteps[step];
      
      // Update the feature's opacity property in the source data
      const sourceData = source.serialize().data;
      const featureIndex = sourceData.features.findIndex(f => 
        f.geometry.coordinates[0] === feature.geometry.coordinates[0] &&
        f.geometry.coordinates[1] === feature.geometry.coordinates[1]
      );

      if (featureIndex !== -1) {
        sourceData.features[featureIndex].properties.opacity = currentStep.opacity;
        source.setData(sourceData);
      }

      // Schedule next step
      if (step < animationSteps.length - 1) {
        setTimeout(() => animate(step + 1), currentStep.duration);
      }
    };

    animate();
  },
      

    showFeatureList(features) {
      this.currentFeatures = features;
      this.selectedFeature = null;
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
      return `/images/${filename}`;
    }
  }
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


/* filter: contrast(140%) grayscale(100%) saturate(0%) brightness(74%); */

.mapboxgl-canvas{ 
  filter: contrast(140%) brightness(74%);
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

  border: 1px solid white;
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
  margin-bottom: 5px;
}

.filters-toggle-container .filters-toggle {
  padding: 8px 12px;
  background-color: rgba(255, 255, 255, 0.9);
  border: 1px solid black;
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
.filters-toggle-container .filters-toggle.active-filter{
  background-color: black !important;
  color: white !important;
  opacity: 0.8;
}

.filters-toggle-container .filters-toggle.active-panel, .filters-toggle-container .filters-toggle:hover{
  background-color: black !important;
  color: white;
  opacity: 1;
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
.modal li.feature-list-item{
  text-decoration: underline;
}
.modal li.feature-list-item:hover{
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
  right: 10px !important;
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


/* Disabled State Styles */
.date-slider-container.disabled {
  opacity: 0.5; /* Faded effect */
  /* Allow interactivity by not using pointer-events: none; */
}

/* Central Date Styles */
.selected-dates {
  margin-top: 10px;
  font-size: 14px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.selected-dates span {
  flex: 1;
  text-align: center;
  pointer-events: none; /* Prevent clicking on central date */
}

/* Styles to Disable the Second Handle */
.noUi-handle.disabled-handle {
  opacity: 0.5;
  pointer-events: auto; /* Allow interaction to enable it */
}

/* Hide the connecting line when single handle is active */
.noUi-connect.disabled-connect {
  background: transparent;
}
.selected-dates .faded {
  opacity: 0.5;
  cursor: pointer;
}

.search-input{
  position: relative;
  float: left;
  margin-left: 15px;
  margin-right: 15px;
  background-color: black;
  border: 1px solid white;
  color: white;
  padding: 8px 12px;
  top: 10px;
  outline: none !important;
}


.zoom-controls {
  position: absolute;
  bottom: 30px;
  right: 30px;
  z-index: 15;
  display: flex;
  flex-direction: column;
  align-items: center;
  background: rgba(0, 0, 0, 0.6);
  padding: 10px;
  border-radius: 4px;
}

.zoom-button {
  width: 30px;
  height: 30px;
  background: transparent;
  border: none;
  color: white;
  font-size: 20px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0;
}

.zoom-button:hover {
  background: rgba(255, 255, 255, 0.1);
}

.zoom-slider-container {
  height: 150px;
  margin: 10px 0;
  display: flex;
  align-items: center;
}

.zoom-slider {
  height: 100%;
  width: 4px;
}

/* Custom noUiSlider styles for zoom control */
.zoom-slider .noUi-connects {
  background-color: rgba(255, 255, 255, 0.3);
}

.zoom-slider .noUi-connect {
  background-color: white;
}

.zoom-slider .noUi-handle {
  width: 18px;
  height: 18px;
  border-radius: 9px;
  background-color: white;
  box-shadow: none;
  border: none;
  right: -7px;
}

.zoom-slider .noUi-handle:before,
.zoom-slider .noUi-handle:after {
  display: none;
}


.date-mode-buttons{
  margin-bottom: 20px;
}
.date-mode-buttons button:last-child{
  float: right;
}
.date-slider-container .date-mode-buttons button{
  background-color: transparent;
  border: 1px solid white;
  padding: 8px 16px;
  cursor: pointer;
}

.date-slider-container.disabled .date-mode-buttons button{
  opacity: 0.8;
}

#app .date-slider-container .date-mode-buttons button.active{
  color: black !important;
  background-color: white;
}
.selected-dates{
  margin-top: 20px;
}
.noUi-target{
  background-color: transparent;
}
.noUi-horizontal .noUi-handle{
  cursor: pointer;
  width: 15px;
  right: -8px;
}
.noUi-horizontal .noUi-handle::after, .noUi-horizontal .noUi-handle::before{
  display: none;
}



.filters-toggle:last-child{
  border-color: white !important;
}


.filter-label {
  cursor: pointer;
  user-select: none;
}

.filter-label:hover {
  opacity: 0.8;
}

.feature-image{
  max-height: 300px;
  margin-bottom: 10px;
}
.tag-label{
  margin-top: 20px;
  display: block;
  font-weight: bold;
}
.tag-label + ul, .tag-label + div{
  margin-top: 5px;
}

@media (max-width: 1100px)  {
  #app .date-slider-container{
    right: auto !important;
    left: 10px !important;
    top: auto !important;
    bottom: 10px !important;
  }
}
</style>