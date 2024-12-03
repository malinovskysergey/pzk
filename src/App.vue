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
        <img src="/custom-marker-hover.png" alt="Custom Marker" />
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
    <!-- Combined container for all controls -->
    <div class="date-slider-container">
      <!-- Control buttons row - now inside date-slider-container -->
      <div class="control-buttons-row">
        <!-- Group 1: Graph Toggle -->
        <button class="control-button" :class="{ active: isGraphMode }" @click="toggleGraphMode">
          Графики
        </button>

        <!-- Group 2: Only show when graph mode is active -->
        <template v-if="isGraphMode">
          <div class="controls-group">
            <!-- Checkbox -->
            <div class="filter-checkbox">
              <input type="checkbox" checked disabled />
              <span>Есть данные</span>
            </div>

            <!-- Range Controls -->
            <div class="range-controls">
              <button 
                v-for="range in ranges" 
                :key="range.value"
                :class="['control-button', { active: selectedRange === range.value }]"
                @click="setRange(range.value)"
              >
                {{ range.label }}
              </button>
            </div>

            <!-- Playback Controls -->
            <div class="playback-controls" v-if="selectedRange !== 'all'">
              <button 
                :class="['control-button', { active: isPlayingBackwards }]"
                @click="toggleBackwardsPlay"
                :disabled="!canPlay"
              >
                <span v-if="isPlayingBackwards">П</span>
                <span v-else :style="{ transform: 'rotate(180deg)', display: 'inline-block' }">▶</span>
              </button>
              <button 
                :class="['control-button', { active: isPlaying }]"
                @click="togglePlayback"
                :disabled="!canPlay"
              >
                {{ isPlaying ? 'П' : '▶' }}
              </button>
            </div>

          </div>

          <!-- Group 3: Stats -->
          <div class="stats-group">
            <div class="date-stats">
              {{ formattedStartDate }} - {{ formattedEndDate }}
              <span class="">  -  {{ displayedFeatureCount }}</span>
            </div>
          </div>
        </template>
      </div>

      <!-- Histogram section -->
      <div v-if="isGraphMode" class="histogram-wrapper">
        <div class="histogram-scroll-container">
          <div class="histogram-wrapper">
        <div class="histogram-scroll-container">
          <div class="histogram-container"  @click="handleHistogramClick">
            <div 
              v-if="showRangeOverlay && selectedRange !== 'all'"
              class="range-selection-overlay"
              :style="rangeOverlayStyle"
              @mousedown="startRangeDrag"
            ></div>

            <div 
              v-for="month in monthlyData" 
              :key="month.date"
              class="histogram-bar"
              :class="{ 
                'active': isMonthActive(month),
                'in-range': isInCurrentRange(month)
              }"
              :style="{ 
                height: `${(month.count / maxCount) * 100}%`,
                opacity: getBarOpacity(month)
              }"
              @click="handleBarClick(month)"
              @mouseenter="showMonthTooltip($event, month)"
              @mouseleave="hideTooltip"
            ></div>
          </div>

          <div class="year-labels">
            <div 
              v-for="yearLabel in yearLabels" 
              :key="yearLabel.year"
              :class="['year-label', 'active', { 
                'highlighted': isYearInRange(yearLabel.year)
              }]"
              @click="handleYearClick(yearLabel.year)"
              @mouseenter="showYearTooltip($event, yearLabel)"
              @mouseleave="hideTooltip"
            >
              {{ yearLabel.year }}
            </div>
          </div>
          


        </div>
      </div>

      <div 
    v-if="tooltipData"
    class="histogram-tooltip"
    :style="tooltipStyle"
  >
    <div>{{ tooltipData.label }}</div>
    <div>Количество: {{ tooltipData.count }}</div>
  </div>

        </div>
      </div>

 




    </div>
  </div>


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
      dateSliderEnabled: false,
      currentDate:new Date(2024, 11, 31),
      startDate: null,
      endDate: null,
      minDate:  new Date(2000, 0, 1),
      maxDate: new Date(2024, 11, 31),
      monthlyData: [],
      maxCount: 0,
      selectedYear: null,
      selectedMonth: null,
      yearLabels: [], 
      isGraphMode: false,
      ranges: [
        { value: 'all', label: '2000 - 2024', months: -1 },
        { value: 'month', label: '1 месяц', months: 1 },
        { value: '3months', label: '3 месяца', months: 3 },
        { value: '6months', label: '6 месяцев', months: 6 },
        { value: 'year', label: '1 год', months: 12 }
      ],
      selectedRange: 'all',
      speeds: [
        { value: 0.5, label: '0.5x' },
        { value: 1, label: '1x' },
        { value: 2, label: '2x' },
        { value: 4, label: '4x' }
      ],
      playbackSpeed: 1,
      isPlaying: false,
      isPlayingBackwards: false,
      playDirection: 1,
      playbackInterval: null,
      showRangeOverlay: false,
      isDragging: false,
      dragStartX: null,
      tooltipData: null,
      tooltipStyle: null
    };
  }, 
  computed: {
    formattedCurrentDate() {
    if (this.selectedYear) {
      return `${this.selectedYear} г.`;
    }
    return this.currentDate?.toLocaleDateString('ru-RU', { 
      year: 'numeric', 
      month: 'long' 
    });
  },
  formattedStartDate() {
    return this.startDate ? this.startDate.toLocaleDateString('ru-RU', { 
      year: 'numeric', 
      month: 'long' 
    }) : '';
  },
  formattedEndDate() {
    return this.endDate ? this.endDate.toLocaleDateString('ru-RU', { 
      year: 'numeric', 
      month: 'long' 
    }) : '';
  },
  histogramBarWidth() {
    const totalMonths = this.monthlyData.length;
    return totalMonths > 0 ? 100 / totalMonths : 0;
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
    },
    baseFilteredFeatures() {
    return this.geojsonData?.features.filter((feature) => {
      // Apply only non-date filters
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

      return true;
    }) || [];
  },
  hasAnyNonDateFilters() {
    return this.hasActiveClauseFilters || 
           this.hasActiveTagFilters || 
           this.hasActiveNameFilters;
  },
  showDateMode() {
    return this.displayedFeatureCount > 1;
  },
  
  canUseDateMode() {
    return this.showDateMode && this.baseFilteredFeatures.length > 1;
  },
  canPlay() {
      return this.isGraphMode && this.monthlyData.length > 0;
    },
    rangeOverlayStyle() {
    if (!this.showRangeOverlay || this.selectedRange === 'all' || !this.startDate || !this.endDate) {
      return { display: 'none' };
    }

    // Constants based on CSS
    const BAR_WIDTH = 3;
    const BAR_MARGIN = 4; // 2px left + 2px right
    const TOTAL_BAR_WIDTH = BAR_WIDTH + BAR_MARGIN; // 7px total
    const YEAR_WIDTH = TOTAL_BAR_WIDTH * 12; // 84px per year

    // Calculate position
    const startYear = this.startDate.getFullYear();
    const startMonth = this.startDate.getMonth();
    const yearsSince2000 = startYear - 2000;

    // Calculate left position
    const leftPosition = (yearsSince2000 * YEAR_WIDTH) + (startMonth * TOTAL_BAR_WIDTH);

    // Calculate width based on selected range
    let width;
    if (this.selectedRange === 'month') {
      width = TOTAL_BAR_WIDTH;
    } else {
      const endYear = this.endDate.getFullYear();
      const endMonth = this.endDate.getMonth();
      const monthsDifference = ((endYear - startYear) * 12) + (endMonth - startMonth) + 1;
      width = monthsDifference * TOTAL_BAR_WIDTH;
    }

    return {
      left: `${leftPosition}px`,
      width: `${width}px`
    };
  }
  },
  watch: {
    selectedClauses: {
      handler() {
        if (this.dateMode === 'date') {
          this.buildGraph();
        }
        this.applyFilters();
      }
    },
    selectedTags: {
      handler() {
        if (this.dateMode === 'date') {
          this.buildGraph();
        }
        this.applyFilters();
      }
    },
    selectedNames: {
      handler() {
        if (this.dateMode === 'date') {
          this.buildGraph();
        }
        this.applyFilters();
      }
    },
    displayedFeatureCount: {
      handler(newCount) {
        if (newCount <= 1 && this.dateMode === 'date') {
          this.setDateMode('play');
        }
      }
    } 
  },
  mounted() {
  this.initializeMap();
  document.addEventListener('click', this.handleClickOutside);

  this.$nextTick(() => {
        if (this.selectedRange !== 'all') {
          this.showRangeOverlay = true;
        }
      });
},

beforeUnmount() {
      this.stopAllPlayback();
      document.removeEventListener('mousemove', this.handleRangeDrag);
      document.removeEventListener('mouseup', this.stopRangeDrag);
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
      this.scrollHistogramToEnd();
    },


    processMonthlyData() {
      const startYear = 2000;
      const endYear = new Date().getFullYear();
      const monthlyData = [];
      
      for (let year = startYear; year <= endYear; year++) {
        for (let month = 0; month < 12; month++) {
          const date = `${year}-${String(month + 1).padStart(2, '0')}`;
          const existingData = this.monthlyData.find(m => m.date === date) || { count: 0 };
          
          monthlyData.push({
            date,
            count: existingData.count,
            label: new Date(year, month).toLocaleDateString('ru-RU', {
              year: 'numeric',
              month: 'long'
            })
          });
        }
      }
      
      this.monthlyData = monthlyData;
    },

    // Update processDateData method with proper dates array
    processDateData() {
      if (!this.geojsonData?.features) return;

// Collect features with valid dates
const datedFeatures = this.geojsonData.features.filter(feature => {
  const date = feature.properties.date ? new Date(feature.properties.date) : null;
  return date && !isNaN(date.getTime());
});

if (datedFeatures.length === 0) return;

// Set fixed date range
this.minDate = new Date(2000, 0, 1);
this.maxDate = new Date(2024, 11, 31);

// Initialize startDate and endDate
this.startDate = new Date(this.minDate);
this.endDate = new Date(this.maxDate);
  // Generate monthlyData
  const monthCounts = {};
  datedFeatures.forEach(feature => {
    const date = new Date(feature.properties.date);
    const key = `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}`;
    monthCounts[key] = (monthCounts[key] || 0) + 1;
  });

  // Generate all months between minDate and maxDate
  const monthlyData = [];
  const startYear = this.minDate.getFullYear();
  const startMonth = this.minDate.getMonth();
  const endYear = this.maxDate.getFullYear();
  const endMonth = this.maxDate.getMonth();

  let currentYear = startYear;
  let currentMonth = startMonth;

  while (currentYear < endYear || (currentYear === endYear && currentMonth <= endMonth)) {
    const key = `${currentYear}-${String(currentMonth + 1).padStart(2, '0')}`;
    const count = monthCounts[key] || 0;
    const date = new Date(currentYear, currentMonth);
    const label = date.toLocaleDateString('ru-RU', { year: 'numeric', month: 'long' });

    monthlyData.push({
      date: key,
      count,
      label,
      year: currentYear,
      month: currentMonth
    });

    currentMonth++;
    if (currentMonth > 11) {
      currentMonth = 0;
      currentYear++;
    }
  }

  this.monthlyData = monthlyData;
  this.maxCount = Math.max(...monthlyData.map(m => m.count), 1);
  this.monthlyData = monthlyData;
  this.maxCount = Math.max(...monthlyData.map(m => m.count), 1);
  // Generate yearLabels
  const years = [...new Set(monthlyData.map(m => m.year))];
  this.yearLabels = years.map(year => {
    const monthsInYear = monthlyData.filter(m => m.year === year).length;
    const width = (monthsInYear / monthlyData.length) * 100;
    return { year, width };
  });

  this.scrollHistogramToEnd();
},



getBarOpacity(month) {
  if (this.selectedMonth) {
    return this.selectedMonth === month.date ? 1 : 0.3;
  } else if (this.selectedYear) {
    return month.year === this.selectedYear ? 1 : 0.3;
  } else {
    return 1;
  }
},


  // Update selectMonth method
  selectMonth(dateKey) {
    const [year, month] = dateKey.split('-').map(Number);
    const selectedStartDate = new Date(year, month - 1, 1);
    const selectedEndDate = new Date(year, month, 0); // Last day of the month

    if (this.selectedMonth === dateKey) {
      // Deselect month filter
      this.selectedMonth = null;
      this.startDate = new Date(this.minDate);
      this.endDate = new Date(this.maxDate);
    } else {
      // Select month filter
      this.selectedMonth = dateKey;
      this.selectedYear = null;
      this.startDate = selectedStartDate;
      this.endDate = selectedEndDate;
    }

    this.applyFilters();
  },

  isMonthActive(month) {
    if (this.selectedMonth) {
      return this.selectedMonth === month.date;
    } else if (this.selectedYear) {
      return month.year === this.selectedYear;
    } else {
      return true;
    }
  },
  isYearActive(year) {
    return this.selectedYear === year;
  },

  // Update toggleYear method
  toggleYear(year) {
    if (this.selectedYear === year) {
      // Deselect year filter
      this.selectedYear = null;
      this.startDate = new Date(this.minDate);
      this.endDate = new Date(this.maxDate);
    } else {
      // Select year filter
      this.selectedYear = year;
      this.selectedMonth = null;
      this.startDate = new Date(year, 0, 1);
      this.endDate = new Date(year, 11, 31);
    }

    this.applyFilters();
  },

  getYearFeatureCount(year) {
      return this.monthlyData
        .filter(month => new Date(month.date).getFullYear() === year)
        .reduce((sum, month) => sum + month.count, 0);
    },

    showMonthTooltip(event, data) {
      const bar = event.target;
      const rect = bar.getBoundingClientRect();
      
      this.tooltipStyle = {
        left: `${rect.left + (rect.width / 2)}px`,
        top: `${rect.top}px`
      };
      this.tooltipData = data;
    },

    showYearTooltip(event, yearLabel) {
      const element = event.target;
      const rect = element.getBoundingClientRect();
      
      // Only show tooltip if there are features in this year
      const count = this.getYearFeatureCount(yearLabel.year);
      if (count > 0) {
        this.tooltipStyle = {
          left: `${rect.left + (rect.width / 2)}px`,
          top: `${rect.top}px`
        };
        
        this.tooltipData = {
          label: `${yearLabel.year} год`,
          count
        };
      }
    },

    hideTooltip() {
      this.tooltipData = null;
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

        // Process features and dates
        this.processFeaturesByCoordinates();
        this.processDateData(); // Make sure this is called
        this.initializeDateSlider();
        this.addMarkers();
      } catch (error) {
        console.error('Error loading GeoJSON data:', error);
      }
    },
    toggleGraphMode() {
      this.isGraphMode = !this.isGraphMode;
      if (this.isGraphMode) {
        this.$nextTick(() => {
          this.selectedRange = 'all';
          this.setRange('all');
          this.scrollHistogramToEnd();
        });
      } else {
        // Stop any ongoing playback
        this.stopAllPlayback();
        // Reset to initial state
        this.selectedRange = 'all';
        this.showRangeOverlay = false;
        this.startDate = this.minDate;
        this.endDate = this.maxDate;
        // Apply filters to show all features
        this.applyFilters();
      }
    },
    resetAllDateControls() {
      this.stopPlayback();
      this.showRangeOverlay = false;
      this.selectedRange = 'month';
      this.playbackSpeed = 1;
      this.startDate = null;
      this.endDate = null;
      this.resetFilters();
    },
    initializeDefaultRange() {
      // Find the latest date in the data
      const dates = this.monthlyData.map(m => new Date(m.date));
      const maxDate = new Date(Math.max(...dates));
      this.setRangeFromDate(maxDate);
    },

    handleHistogramClick(event) {
    if (this.selectedRange === 'all') return;
    
    const container = event.currentTarget;
    const rect = container.getBoundingClientRect();
    const x = event.clientX - rect.left;
    const barWidth = rect.width / this.monthlyData.length;
    const clickedIndex = Math.floor(x / barWidth);
    
    if (clickedIndex >= 0 && clickedIndex < this.monthlyData.length) {
      const clickedMonth = this.monthlyData[clickedIndex];
      this.handleBarClick(clickedMonth);
    }
  },

    setRange(range) {
      const previousRange = this.selectedRange;
      this.selectedRange = range;
      this.stopAllPlayback();
      
      if (range === 'all') {
        this.showRangeOverlay = false;
        this.startDate = this.minDate;
        this.endDate = this.maxDate;
      } else {
        // If switching from 'all' to a specific range, use the latest date
        const endDate = previousRange === 'all' ? this.maxDate : (this.endDate || this.maxDate);
        
        // Calculate the new range immediately
        const rangeMonths = this.ranges.find(r => r.value === range).months;
        this.endDate = new Date(endDate.getFullYear(), endDate.getMonth() + 1, 0); // End of month
        this.startDate = new Date(endDate.getFullYear(), endDate.getMonth() - (rangeMonths - 1), 1); // Start of month
        
        this.showRangeOverlay = true;
      }
      
      // Force a re-render of the overlay
      this.$nextTick(() => {
        this.applyDateFilter();
      });
    },
 


    stopAllPlayback() {
      this.isPlaying = false;
      this.isPlayingBackwards = false;
      if (this.playbackInterval) {
        clearInterval(this.playbackInterval);
        this.playbackInterval = null;
      }
    },
    handleYearClick(year) {
      this.selectedRange = 'year';
      const yearEnd = new Date(year, 11, 31);
      this.setRangeFromDate(yearEnd);
    },

    isInCurrentRange(month) {
      if (!this.startDate || !this.endDate || this.selectedRange === 'all') return true;
      
      const monthDate = new Date(month.date);
      const monthStart = new Date(monthDate.getFullYear(), monthDate.getMonth(), 1);
      
      // For month range, check exact month match
      if (this.selectedRange === 'month') {
        return monthStart.getFullYear() === this.startDate.getFullYear() && 
               monthStart.getMonth() === this.startDate.getMonth();
      }
      
      // For other ranges, check if within range
      return monthStart >= this.startDate && monthStart <= this.endDate;
    },

    setRangeFromDate(date) {
      if (this.selectedRange === 'all') return;
      
      const rangeMonths = this.ranges.find(r => r.value === this.selectedRange).months;
      
      if (this.selectedRange === 'month') {
        this.startDate = new Date(date.getFullYear(), date.getMonth(), 1);
        this.endDate = new Date(date.getFullYear(), date.getMonth() + 1, 0);
      } else {
        this.endDate = new Date(date.getFullYear(), date.getMonth() + 1, 0);
        this.startDate = new Date(date.getFullYear(), date.getMonth() - (rangeMonths - 1), 1);
      }
      
      this.showRangeOverlay = true;
      this.applyDateFilter();
    },

    scrollToCurrentRange() {
    if (!this.startDate || !this.selectedRange === 'all') return;

    const container = this.$el.querySelector('.histogram-wrapper');
    const scrollContainer = this.$el.querySelector('.histogram-scroll-container');
    if (!container || !scrollContainer) return;

    // Find the index of the current range start
    const currentIndex = this.monthlyData.findIndex(month => {
      const monthDate = new Date(month.date);
      return monthDate >= this.startDate;
    });

    if (currentIndex === -1) return;

    // Calculate scroll position
    const totalWidth = scrollContainer.scrollWidth;
    const containerWidth = container.offsetWidth;
    const barWidth = totalWidth / this.monthlyData.length;
    const targetPosition = (barWidth * currentIndex) - (containerWidth / 3); // Show some context before

    container.scrollTo({
      left: Math.max(0, targetPosition),
      behavior: 'smooth'
    });
  },

  moveRange(step) {
    if (!this.startDate || !this.endDate || this.selectedRange === 'all') return;
    
    const rangeMonths = this.ranges.find(r => r.value === this.selectedRange).months;
    const newStartDate = new Date(this.startDate);
    const newEndDate = new Date(this.endDate);
    
    if (this.selectedRange === 'month') {
      newStartDate.setMonth(this.startDate.getMonth() + step);
      newEndDate.setMonth(this.endDate.getMonth() + step);
    } else {
      newStartDate.setMonth(this.startDate.getMonth() + (step * rangeMonths));
      newEndDate.setMonth(this.endDate.getMonth() + (step * rangeMonths));
    }

    // Check bounds and update dates
    if (step > 0 && newEndDate > this.maxDate) {
      const firstDate = new Date(Math.min(...this.monthlyData.map(m => new Date(m.date))));
      this.setRangeFromDate(firstDate);
    } else if (step < 0 && newStartDate < this.minDate) {
      const lastDate = new Date(Math.max(...this.monthlyData.map(m => new Date(m.date))));
      this.setRangeFromDate(lastDate);
    } else {
      this.startDate = newStartDate;
      this.endDate = newEndDate;
      this.applyDateFilter();
    }

    // Scroll to keep the current range in view
    this.$nextTick(() => {
      this.scrollToCurrentRange();
    });
  },

    playBackwards() {
      this.playDirection = -1;
      this.startPlayback();
    },

    startPlayback() {
      this.isPlaying = true;
      const intervalTime = 1000 / this.playbackSpeed;
      
      this.playbackInterval = setInterval(() => {
        this.moveRange(this.playDirection);
      }, intervalTime);
    },

    toggleBackwardsPlay() {
      if (this.selectedRange === 'all') {
        this.selectedRange = 'month';
      }
      if (this.isPlayingBackwards) {
        this.stopAllPlayback();
      } else {
        this.playDirection = -1;
        this.isPlayingBackwards = true;
        this.startPlayback();
      }
    },

    togglePlayback() {
      if (this.selectedRange === 'all') {
        this.selectedRange = 'month';
      }
      if (this.isPlaying) {
        this.stopAllPlayback();
      } else {
        this.playDirection = 1;
        this.isPlaying = true;
        this.startPlayback();
      }
    },

    stopPlayback() {
      this.isPlaying = false;
      if (this.playbackInterval) {
        clearInterval(this.playbackInterval);
        this.playbackInterval = null;
      }
    },
    
    isYearInRange(year) {
    if (!this.startDate || !this.endDate || this.selectedRange === 'all') return false;
    
    // Create date objects for the start and end of the year
    const yearStart = new Date(year, 0, 1);
    const yearEnd = new Date(year, 11, 31);
    
    // A year is in range if any part of the selected range overlaps with it
    return (
      // Either the range start date falls within this year
      (this.startDate >= yearStart && this.startDate <= yearEnd) ||
      // Or the range end date falls within this year
      (this.endDate >= yearStart && this.endDate <= yearEnd) ||
      // Or the range completely encompasses this year
      (this.startDate <= yearStart && this.endDate >= yearEnd)
    );
  },

    // Range dragging functionality
    startRangeDrag(event) {
      this.isDragging = true;
      this.dragStartX = event.clientX;
      document.addEventListener('mousemove', this.handleRangeDrag);
      document.addEventListener('mouseup', this.stopRangeDrag);
    },

    handleRangeDrag(event) {
      if (!this.isDragging) return;
      
      const deltaX = event.clientX - this.dragStartX;
      const containerWidth = this.$el.querySelector('.histogram-container').offsetWidth;
      const monthMove = Math.round((deltaX / containerWidth) * this.monthlyData.length);
      
      if (Math.abs(monthMove) >= 1) {
        this.moveRange(monthMove);
        this.dragStartX = event.clientX;
      }
    },

    stopRangeDrag() {
      this.isDragging = false;
      document.removeEventListener('mousemove', this.handleRangeDrag);
      document.removeEventListener('mouseup', this.stopRangeDrag);
    },
    advanceDate() {
      const nextDate = new Date(this.currentDate);
      nextDate.setMonth(nextDate.getMonth() + 1);
      
      // Check if we've reached the end
      if (nextDate > this.maxDate) {
        // Loop back to start
        this.currentDate = new Date(this.minDate);
      } else {
        this.currentDate = nextDate;
      }
      
      this.setRangeFromDate(this.currentDate);
    },
    setPlaybackSpeed(speed) {
      this.playbackSpeed = speed;
      if (this.isPlaying) {
        this.stopPlayback();
        this.startPlayback();
      }
    },
    applyDateFilter() {
      if (!this.startDate || !this.endDate) return;
      
      const filteredFeatures = this.getFilteredFeatures().filter(feature => {
        if (!feature.properties.date) return false;
        const featureDate = new Date(feature.properties.date);
        return featureDate >= this.startDate && featureDate <= this.endDate;
      });

      this.displayedFeatureCount = filteredFeatures.length;
      this.addMarkers(filteredFeatures);
    },


 

    handleBarClick(month) {
    const clickedDate = new Date(month.date);
    
    // If in 'all' range, switch to month range
    if (this.selectedRange === 'all') {
      this.selectedRange = 'month';
    }
    
    if (this.selectedRange === 'month') {
      this.startDate = new Date(clickedDate.getFullYear(), clickedDate.getMonth(), 1);
      this.endDate = new Date(clickedDate.getFullYear(), clickedDate.getMonth() + 1, 0);
    } else {
      this.setRangeFromDate(clickedDate);
    }
    
    this.showRangeOverlay = true;
    this.applyDateFilter();
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
    scrollHistogramToEnd() {
    this.$nextTick(() => {
      const histogramWrapper = this.$el.querySelector('.histogram-wrapper');
      if (histogramWrapper) {
        histogramWrapper.scrollLeft = histogramWrapper.scrollWidth;
      }
    });
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
  // Only enable if clicking on the slider area
  if (event.target.classList.contains('noUi-connects') ||
      event.target.classList.contains('noUi-handle')) {
    this.dateSliderEnabled = true;
    this.applyFilters();
  }
},

processFilteredDates() {
    const monthCounts = {};
    const yearCounts = {};
    const dates = [];

    // Process only filtered features
    this.baseFilteredFeatures.forEach(feature => {
      if (feature.properties.date) {
        const date = new Date(feature.properties.date);
        if (!isNaN(date.getTime())) {
          dates.push(date);
          
          const monthKey = `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}`;
          const yearKey = date.getFullYear();
          
          monthCounts[monthKey] = (monthCounts[monthKey] || 0) + 1;
          yearCounts[yearKey] = (yearCounts[yearKey] || 0) + 1;
        }
      }
    });

    // Update date range based on filtered features
    if (dates.length > 0) {
      this.minDate = new Date(Math.min(...dates));
      this.maxDate = new Date(Math.max(...dates));
      
      // Only update current date if it's outside the new range
      if (this.currentDate > this.maxDate) {
        this.currentDate = this.maxDate;
      } else if (this.currentDate < this.minDate) {
        this.currentDate = this.minDate;
      }

      // Update range dates
      if (this.startDate > this.maxDate || this.startDate < this.minDate) {
        this.startDate = this.minDate;
      }
      if (this.endDate > this.maxDate || this.endDate < this.minDate) {
        this.endDate = this.maxDate;
      }
    }

    // Update histogram data
    this.monthlyData = Object.entries(monthCounts)
      .map(([key, count]) => {
        const [year, month] = key.split('-');
        return {
          date: key,
          count,
          label: new Date(year, parseInt(month) - 1).toLocaleDateString('ru-RU', {
            year: 'numeric',
            month: 'long'
          })
        };
      })
      .sort((a, b) => a.date.localeCompare(b.date));

    this.yearlyData = Object.entries(yearCounts)
      .map(([year, count]) => ({
        year: parseInt(year),
        count
      }))
      .sort((a, b) => a.year - b.year);

    this.maxCount = Math.max(...this.monthlyData.map(m => m.count), 1);

    // Reinitialize slider with new range if in range mode
    if (this.dateMode === 'range' && this.dateSlider) {
      this.$nextTick(() => {
        this.initializeDateSlider();
      });
    }
  },

  // Update watchers to trigger date processing
  watch: {
    selectedClauses: {
      handler() {
        this.processFilteredDates();
        this.applyFilters();
      }
    },
    selectedTags: {
      handler() {
        this.processFilteredDates();
        this.applyFilters();
      }
    },
    selectedNames: {
      handler() {
        this.processFilteredDates();
        this.applyFilters();
      }
    },
    startDate() {
    this.$nextTick(() => {
      this.scrollToCurrentRange();
    });
  },
    monthlyData: {
      handler() {
        if (this.selectedRange !== 'all' && this.showRangeOverlay) {
          this.$nextTick(() => {
            // Force reactive update of overlay
            this.showRangeOverlay = false;
            this.$nextTick(() => {
              this.showRangeOverlay = true;
            });
          });
        }
      },
      deep: true
    },
    selectedRange: {
        immediate: true,
        handler(newRange) {
          if (newRange !== 'all') {
            this.$nextTick(() => {
              this.showRangeOverlay = true;
            });
          }
        }
      }
  },

  // Update initializeDateSlider method
  initializeDateSlider() {
    const sliderElement = this.$refs.dateSlider;
    
    if (!sliderElement || !this.minDate || !this.maxDate) return;

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

    buildGraph() {
    this.processFilteredDates();
    this.graphBuilt = true;
    this.needsRebuild = false;
  },

  setDateMode(mode) {
    // If trying to switch to date mode but not enough features, switch to play
    if (mode === 'date' && !this.canUseDateMode) {
      mode = 'play';
    }

    // If currently in date mode and features drop below threshold, switch to play
    if (this.dateMode === 'date' && !this.canUseDateMode) {
      mode = 'play';
    }

    this.dateMode = mode;
    this.dateSliderEnabled = false;
    this.selectedMonthDate = null;
    this.selectedYear = null;
    
    if (mode === 'date') {
      // Immediately build graph when switching to date mode
      this.$nextTick(() => {
        this.buildGraph();
      });
    } else {
      // Initialize slider for other modes
      this.$nextTick(() => {
        this.initializeDateSlider();
      });
    }
    
    this.applyFilters();
  },

    resetDateSlider() {
      this.initializeDateSlider();
      this.applyFilters();
    },
    getFilteredFeatures() {
  if (!this.geojsonData?.features) return [];

  return this.geojsonData.features.filter((feature) => {
    // Apply name filters
    if (this.hasActiveNameFilters && 
        !this.selectedNames.includes(feature.properties.name)) {
      return false;
    }

    // Apply clause filters
    if (this.hasActiveClauseFilters && 
        !feature.properties.clauses?.some(clause => 
          this.selectedClauses.includes(clause))) {
      return false;
    }

    // Apply tag filters
    if (this.hasActiveTagFilters && 
        !feature.properties.tags?.some(tag => 
          this.selectedTags.includes(tag))) {
      return false;
    }

    // Apply date filtering
    if (this.selectedMonth || this.selectedYear) {
      if (!feature.properties.date) return false; // Exclude items without dates when date filter is active
      const date = new Date(feature.properties.date);
      if (isNaN(date.getTime())) return false;

      if (this.selectedMonth) {
        return feature.properties.date.startsWith(this.selectedMonth);
      } else if (this.selectedYear) {
        return date.getFullYear() === this.selectedYear;
      }
    }

    return true;
  });
},


  
updateMonthlyData(filteredFeatures) {
  const monthCounts = {};

  filteredFeatures.forEach(feature => {
    if (feature.properties.date) {
      const date = new Date(feature.properties.date);
      if (!isNaN(date.getTime())) {
        const key = `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}`;
        monthCounts[key] = (monthCounts[key] || 0) + 1;
      }
    }
  });

  // Update the monthlyData counts
  this.monthlyData = this.monthlyData.map(month => {
    const count = monthCounts[month.date] || 0;
    return {
      ...month,
      count
    };
  });
  this.maxCount = Math.max(...this.monthlyData.map(m => m.count), 1);

// Scroll histogram to end
this.scrollHistogramToEnd();
},

  getNonDateFilteredFeatures() {
  if (!this.geojsonData?.features) return [];

  return this.geojsonData.features.filter((feature) => {
    // Apply name filters
    if (this.hasActiveNameFilters && 
        !this.selectedNames.includes(feature.properties.name)) {
      return false;
    }

    // Apply clause filters
    if (this.hasActiveClauseFilters && 
        !feature.properties.clauses?.some(clause => 
          this.selectedClauses.includes(clause))) {
      return false;
    }

    // Apply tag filters
    if (this.hasActiveTagFilters && 
        !feature.properties.tags?.some(tag => 
          this.selectedTags.includes(tag))) {
      return false;
    }

    // Do not apply date filters
    return true;
  });
},

applyFilters() {
      // Get features filtered by non-date filters
      const filteredFeatures = this.getFilteredFeatures();
      this.displayedFeatureCount = filteredFeatures.length;
      this.addMarkers(filteredFeatures);

      // Update monthly data for histogram if it's visible
      if (this.isGraphMode) {
        const nonDateFilteredFeatures = this.getNonDateFilteredFeatures();
        this.updateMonthlyData(nonDateFilteredFeatures);
      }
    },




    resetFilters() {
  // Reset all filter selections
  this.selectedClauses = [];
  this.selectedTags = [];
  this.selectedNames = [];
  this.selectedMonthDate = null;
  this.selectedYear = null;
    this.isGraphMode = false;
    this.stopAllPlayback();
    this.resetDateFilters();
  
  // Reset date-related states
  this.dateSliderEnabled = false;
  
  if (this.minDate && this.maxDate) {
    this.currentDate = this.maxDate;
    this.startDate = this.minDate;
    this.endDate = this.maxDate;
  }
  
  // Reset UI states
  this.filtersVisible = false;
  this.activeFilterCategory = null;
  this.clearSearch();

  // Force slider to reset if it exists
  if (this.dateSlider) {
    this.dateSlider.destroy();
    this.dateSlider = null;
  }

  // Initialize the slider if in range mode
  this.$nextTick(() => {
    if (this.dateMode === 'range') {
      this.initializeDateSlider();
    }
  });
  
  // Apply the reset filters
  this.applyFilters();
},

resetDateFilters() {
      this.selectedRange = 'all';
      this.showRangeOverlay = false;
      this.startDate = null;
      this.endDate = null;
      this.playbackSpeed = 1;
      // Fix for missing clearDateFilters
      this.startDate = this.minDate;
      this.endDate = this.maxDate;
      this.applyDateFilter();
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
  background-image: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAoCAYAAACM/rhtAAAACXBIWXMAAAsSAAALEgHS3X78AAAFZElEQVRYhcWZz0+cRRjHvzPzbsNW2uwmpm8xttrWrcphlzQW0jSeTEF7ay+mRhN/0dve+geQ3rxVMDHSRK9yaeOpHLSaCLRgpGUPhBatIobsCrweJEJ935mvh87gQIEFd5d+L/vmfed55jPP+84zM88CNUoppQDg2LFj++7fv//V7OzsjUKhsB8AUqmUqtV/TQqCQAFAR0fH4fn5+R9ptbi4ONHZ2XnEb/PE4Lq6uo6QfGCMoTEm0VonWmuSnO3u7n4Z+C/KuyallASAkydPhlEUTZGk1jp2EdRax8YYxnH84PTp08/6Ng2XEEIAEC0tLcHc3Nw36+F8SJJcWFgYyuVyTQAgpRS7AagAYHBw8COSjOP4MTinJEn+IcmhoaGPU6kUAKhH42uQ3LdULBZfJWniODabwTnFcaxJslgsdvo+6i47chGGISqVyvckaYxJqgEaYzRJViqV8TAMU9ZX/cPoRt7T0/OWjUxVOC+KCUn29PS8DzQg9bgRh2GIcrk86kdmO3Jty+XynTAMpe+zLnLR6+7ufo0kkySp+u1tEEVNkpcuXXodqHMU3cwdHh7uJzdOK9XkbIaHh7/wfdYsl7taW1ub4jh+4L+yncjZJEky09ra2uz73rL/bQBKAGhvb88HQXBEa00hxI5XBSGE1FpTKXW4vb294PuuCRCAAIBCofAKAJDUO4VzcraFQqHd910ToDEGAJDP59uA1Xz4v+Rs8/l8wfpmTYBCCBhjTDqdRi6Xe8He24yQWmujtTYANuzY2eZyuefT6TSMMaw24GoRFABMU1OTam5uPrgFII0xUEpBKeWi/hiks21ubm5pamoKABhUec3b+thJpgDs3eyx1lpIKdHX19fd19f3rpSSWmuxEaTVXpJ7ttP3lnIjzmaz+6Ioqth0sSZJJ0miSfLq1asfOLv+/v637TOzLtUYkoyi6I9sNrvf76MhgFrrhCRv3779WRAEkFKmgiBIAcDg4OAnFjJpOGAmk0lHUTSzLkm7/v48c+bMYeDRrtntnE+dOvUMyQXbxvi2URT9lslk0vWMoIyiaHLdihCT5MjIyOcAIKUMPLsAAEZGRj7123qA97LZbLAdwGqThADkysqKWVpaKgMASdpfAQDXrl27YQFXJ4RSivbZ135bZ7u0tFReWVlJbP9b5sItAUlCSimXl5cxPT39k9cJpZQKAMfHx6f8zv1r+8y4tu7+9PT0r8vLy5BSCs9s54AA4JbLUqk04aA9/Q1g0XH5Y7O/kW2zOmDr6671XftS5zorlUo/ADveJq0BcLYTExNjvu+aAI1dFm7dulV6+PDhz0op4b3OpwA8vQGMu87AJniSVEoJrfXs2NhYyfddKyCllGpqamplaGjoW+fYGKMB4MSJEy8Ba2eju87n8y8CkMYY7WBGR0e/m5yc/EsIoWreLHgdAgAGBgYGnJ0QggBw/vz5Lgu9Cuiuz54922XtSVICwPXr178EgLqePl1EDhw4sHpo0lonXqI+ZDuVSikBrE3UbsUpl8t3wzBUvs+6yTt2XiAfHSWrLHW9bqnzjp0fAg2qeLkB24P7kF0ZErdZ6O/vf8+1vXz58gW3WXCH+0qlcicMwz3WV2PqH+tKH7SlD2N3Laa3t/edK1euvElSu3suesVi8Q3fR8O0SfHIaK0NSU0ysbsI4xWPeneleGQBBQC0tLSk5ubmbtoJE1sgbV+5cd/n4uLiyPHjx9PALpXfgDUFzINRFN3zIN0+0S9gHvJtdk1eCfgoyV9cOvFKwL9fvHix1cI92Tp1R0fHc/Pz83dcBBcWFkrnzp07+kThnBxAW1vb/pmZmcGZmZmbbW1tGaA+f0P8C3H0t/gtsh06AAAAAElFTkSuQmCC); /* Replace with your image path */
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
  min-width: 500px;
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
  top: 80px;
  max-height: calc(100vh - 140px);
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
  margin-bottom: 20px;
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
  bottom: 10px;
  right: 10px;
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
  margin-top: 20px;
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
  -webkit-user-select: none;
     -moz-user-select: none;
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
.date-slider-container {
  background-color: rgba(0, 0, 0, 0.6);
  padding: 15px;
  border-radius: 4px;
  width: calc(100% - 30px);
}
.histogram-scroll-container {
  display: flex;
  flex-direction: column;
  width: -moz-max-content;
  width: max-content;
  margin: 0 auto;
} 
.histogram-wrapper::-webkit-scrollbar {
  display: none; /* Chrome, Safari, Opera */
}
.histogram-container {
  position: relative;
  height: 150px;
  width: 100%;
  display: flex;
  align-items: flex-end;
  border-bottom: 1px solid #fff;
  cursor: pointer;
}
.histogram-bar {
  background-color: white;
  cursor: pointer;
  transition: all 0.3s ease;
  width: 3px;
  max-width: 3px;
  flex: 1 0 auto;
  margin-left: 2px;
  margin-right: 2px;
  position: relative;
  z-index: 2;
}
.year-labels {
  display: flex;
}
.year-label {
  flex: 1 0 auto;
  min-width: 84px;
  text-align: center;
  cursor: pointer;
}
.year-label:hover,
.year-label.active {
  opacity: 1;
}
.histogram-wrapper.inactive {
  opacity: 0.3;
  pointer-events: none;
}
.build-graph-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1;
  pointer-events: all;
}
.build-graph-button {
  background: rgba(255, 255, 255, 0.9);
  border: 1px solid white;
  color: black;
  padding: 8px 16px;
  cursor: pointer;
  border-radius: 4px;
  font-size: 14px;
  transition: all 0.3s ease;
}
.build-graph-button:hover {
  background: white;
}
.histogram-bar:hover {
  opacity: 0.8;
}
.year-labels {
  display: flex;
  margin-top: 5px;
}
.year-label button {
  background: transparent;
  border: none;
  color: white;
  cursor: pointer;
  opacity: 0.5;
  transition: opacity 0.2s;
  width: 100%;
}
.year-label.active button,
.year-label button:hover {
  opacity: 1;
}
.date-controls-container {
  position: absolute;
  bottom: 10px;
  left: 10px;
  right: 10px;
  z-index: 15;
}
.control-buttons-row {
  display: flex;
  gap: 15px;
  align-items: center;
  position: relative;
}
.stats-group {
  margin-left: auto;
}
.date-stats {
  color: white;
  display: flex;
  align-items: center;
  gap: 10px;
}
.feature-counter {
  opacity: 0.7;
  font-size: 0.9em;
}
.control-button {
  padding: 4px 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background: white;
  cursor: pointer;
}
.control-button.active {
  background: #007bff;
  color: white;
}
.histogram-tooltip {
  position: fixed; /* Changed from absolute to fixed */
  background: rgba(0, 0, 0, 0.8);
  color: white;
  padding: 4px 8px;
  border-radius: 4px;
  pointer-events: none;
  z-index: 20;
  transform: translate(-50%, -100%);
}
.range-selection-overlay {
  position: absolute;
  height: 100%;
  background: rgba(237, 6, 0, 0.3);
  pointer-events: none;
  z-index: 1;
  transition: left 0.3s ease, width 0.3s ease;
}
.active.histogram-bar:not(.in-range) {
  opacity: 0.5 !important;
}
.range-controls, .playback-controls, .speed-controls {
  display: flex;
  gap: 8px;
}




/* Add to existing styles */
#app .year-label.highlighted { 
  font-weight: bold;
  background-color: white;
  color: black !important;
}
.range-selection-overlay:active {
  cursor: grabbing;
}
.control-buttons-row{
  position: fixed;
  bottom: 10px;
  left: 10px;
}
.date-slider-container{
  bottom: 50px;
}
.control-button{
  padding: 8px 12px;
  background-color: rgba(0, 0, 0, 0.6) !important;
  border: 1px solid black;
  border-radius: 4px;
  cursor: pointer;
  text-align: left;
  color: white;
}
.control-button.active{
  background-color: rgba(255, 255, 255, 0.9) !important;
  color: black !important;
}
.filter-checkbox {
  display: flex;
  align-items: center;
  gap: 5px;
  color: white;
  margin-right: 10px;
  pointer-events: none;
  opacity: 0.8;
}






/* Main container */
.date-controls-container {
  position: absolute;
  bottom: 10px;
  left: 10px;
  right: 10px;
  z-index: 15;
}

/* Slider container */
.date-slider-container {
  background-color: rgba(0, 0, 0, 0.6);
  padding: 0;
  border-radius: 4px;
  width: calc(100vw - 111px);
  height: 100%;
  display: inline-block;
}

/* Control buttons row */
.control-buttons-row {
  display: flex;
  gap: 15px;
  align-items: center;
  margin-bottom: 15px;
  width: calc(100vw - 111px);
  max-width: 100%;
}

/* Controls grouping */
.controls-group {
  display: flex;
  gap: 15px;
  align-items: center;
  flex-wrap: wrap;
}
.stats-group {
  margin-left: auto;
}

/* Filter checkbox styling */
.filter-checkbox {
  display: flex;
  align-items: center;
  gap: 5px;
  color: white;
  margin-right: 10px;
}

/* Range and playback controls */
.range-controls,
.playback-controls,
.speed-controls {
  display: flex;
  gap: 8px;
  align-items: center;
}

/* Control buttons */
.control-button {
  padding: 8px 12px;
  background-color: rgba(0, 0, 0, 0.6);
  border: 1px solid white;
  border-radius: 4px;
  cursor: pointer;
  color: white;
  text-align: center;
}
.control-button.active {
  background-color: rgba(255, 255, 255, 0.9) !important;
  color: black !important;
}

#app .control-buttons-row  .control-button.active{
  color: black !important;
}


/* Stats styling */
.date-stats {
  color: white;
  display: flex;
  align-items: center;
  gap: 10px;
}
.feature-counter {
  opacity: 0.7;
  font-size: 0.9em;
}

/* Histogram container */
.histogram-wrapper {
  overflow-x: auto;
  scrollbar-width: none;
  -ms-overflow-style: none;
}
.date-slider-container > .histogram-wrapper{
  
  margin-bottom: 70px;
  padding-top: 20px;
}
.histogram-wrapper::-webkit-scrollbar {
  display: none;
}
.histogram-container {
  position: relative;
  height: 150px;
  display: flex;
  align-items: flex-end;
  border-bottom: 1px solid #fff;
}

/* Responsive styles */
@media (max-width: 1100px) {
.date-slider-container {
    width: calc(100% - 40px);
}
}
@media (max-width: 980px) {
.control-buttons-row {
    flex-wrap: wrap;
}
.stats-group {
    width: 100%;
    margin-left: 0;
    margin-top: 10px;
}
}
@media (max-width: 1100px)  {
#app .date-slider-container{
    right: auto !important;
    left: 10px !important;
    top: auto !important;
    bottom: 10px !important;
}
.zoom-slider-container{
  display: none;
}
.zoom-controls{
    left: 10px !important;
    right: auto !important;
    bottom: 150px !important;
}
.date-slider-container{
    width: calc(100% - 40px);
}
.filters{
    height:calc(100vh - 333px);
}
.modal-content{  
  max-height: calc(100vh - 280px);
}
}
@media (max-width: 980px) {
.filters-toggle-container  .feature-count{
    right:  10px !important;
    left: auto !important;
    position: absolute;
    top: 10px;
}
.search-input{
    margin-left: 10px;
    float: none;
    display: block;
    margin-bottom: 10px;
    width: 211px;
}
.filters-toggle{
    float: none !important;
}
.filters-toggle-container{
    width: 100%;
    left: 0;
    top: 0;
}
.filters-toggle:last-child{
    display: block;
    margin-top: 10px;
}
.filters{
    height:calc(100vh - 390px);
    top: 145px;
    width: calc(100vw - 50px);
    right: 20px;
}
.modal-content{   
  height: calc(100vh - 440px);
  top: 145px;
  width: calc(100vw - 60px);
  right: auto;
  max-width: 100%;
  left: 0px;
}
}



 
 </style>