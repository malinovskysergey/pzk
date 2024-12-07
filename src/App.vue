<template>
  <div id="app">
    <div ref="mapContainer" class="map-container"></div>
    <div class="zoom-controls">
      <button class="zoom-button" @click="zoomIn">+</button>
      <div class="zoom-slider-container">
        <div ref="zoomSlider" class="zoom-slider"></div>
      </div>
      <button class="zoom-button" @click="zoomOut">−</button>
    </div>
    <div class="filters-toggle-container" ref="filtersToggleContainer">
      <div class="feature-count">
        <img src="/custom-marker-hover.png" alt="Custom Marker" />
        {{ displayedFeatureCount }}
      </div>
      <input v-model="searchQuery" @input="handleSearch" @keyup.enter="handleSearchEnter" placeholder="Поиск по имени или статье" class="search-input" />
      <button class="filters-toggle" @click="setActiveFilter('names')" :class="{'active-panel': activeFilterCategory === 'names','active-filter': hasActiveNameFilters}">Имена</button>
      <button class="filters-toggle" @click="setActiveFilter('clauses')" :class="{'active-panel': activeFilterCategory === 'clauses','active-filter': hasActiveClauseFilters}">Статьи</button>
      <button class="filters-toggle" @click="setActiveFilter('tags')" :class="{'active-panel': activeFilterCategory === 'tags','active-filter': hasActiveTagFilters}">Дела</button>
      <button class="filters-toggle" @click="resetFilters" :class="{'active-filter': hasAnyActiveFilters}">Сбросить</button>
    </div>
    <div class="date-controls-container">
      <div class="date-slider-container">
        <div class="control-buttons-row">
          <button class="control-button" :class="{ active: isGraphMode }" @click="toggleGraphMode">Время</button>
          <div v-show="isGraphMode" class="controls-group">
            <div class="filter-checkbox">
              <input type="checkbox" checked disabled />
              <span>Есть данные</span>
            </div>
            <div class="date-range-slider-container">
              <div ref="dateRangeSlider" class="date-range-slider"></div>
            </div>
            <div class="playback-slider-container">
              <div ref="playbackSlider" class="playback-slider"></div>
            </div>
          </div>
        </div>
        <div v-if="isGraphMode" class="stats-group">
          <div class="date-stats">{{ formattedStartDate }} - {{ formattedEndDate }}</div>
        </div>
        <div v-if="isGraphMode" class="histogram-wrapper">
          <div class="histogram-scroll-container">
            <div class="histogram-container" @click="handleHistogramClick">
              <div v-if="selectedRangeMonths !== totalMonths" class="range-selection-overlay" :style="rangeOverlayStyle" @mousedown="startRangeDrag"></div>
              <div v-for="month in monthlyData" :key="month.date" class="histogram-bar" :class="{ 'active': isMonthActive(month),'in-range': isInCurrentRange(month)}" :style="{height: (month.count / maxCount) * 100 + '%',opacity: getBarOpacity(month)}" @click="handleBarClick(month)" @mouseenter="showMonthTooltip($event, month)" @mouseleave="hideTooltip"></div>
            </div>
            <div class="year-labels">
              <div v-for="yearLabel in yearLabels" :key="yearLabel.year" class="year-label active" :class="{ 'highlighted': isYearInRange(yearLabel.year) }" @click="handleYearClick(yearLabel.year)" @mouseenter="showYearTooltip($event, yearLabel)" @mouseleave="hideTooltip">{{ yearLabel.year }}</div>
            </div>
          </div>
          <div v-if="tooltipData" class="histogram-tooltip" :style="tooltipStyle">
            <div>{{ tooltipData.label }}</div>
            <div>Количество: {{ tooltipData.count }}</div>
          </div>
        </div>
      </div>
    </div>
    <div v-if="activeFilterCategory === 'names'">
      <div v-if="filteredNames.length === 0" class="no-results">Ничего не найдено</div>
      <div v-else v-for="name in filteredNames" :key="name" class="filter-div">
        <input type="checkbox" :value="name" v-model="selectedNames" :id="'name-' + name" />
        <span class="filter-label" @click="toggleFilter('names', name)">{{ name }}</span>
      </div>
    </div>
    <div v-if="activeFilterCategory === 'clauses'">
      <div v-if="filteredClauses.length === 0" class="no-results">Ничего не найдено</div>
      <div v-else v-for="clause in filteredClauses" :key="clause" class="filter-div">
        <input type="checkbox" :value="clause" v-model="selectedClauses" :id="'clause-' + clause" />
        <span class="filter-label" @click="toggleFilter('clauses', clause)">{{ clause }}</span>
      </div>
    </div>
    <div v-if="activeFilterCategory === 'tags'">
      <div v-if="filteredTags.length === 0" class="no-results">Ничего не найдено</div>
      <div v-else v-for="tag in filteredTags" :key="tag" class="filter-div">
        <input type="checkbox" :value="tag" v-model="selectedTags" :id="'tag-' + tag" />
        <span class="filter-label" @click="toggleFilter('tags', tag)">{{ tag }}</span>
      </div>
    </div>
    <div class="filters" v-show="filtersVisible" ref="filtersPanel">
      <div v-if="activeFilterCategory === 'names'">
        <div v-if="filteredNames.length === 0" class="no-results">Ничего не найдено</div>
        <div v-else v-for="name in filteredNames" :key="name" class="filter-div">
          <input type="checkbox" :value="name" v-model="selectedNames" :id="'name-' + name" />
          <span class="filter-label" @click="toggleFilter('names', name)">{{ name }}</span>
        </div>
      </div>
      <div v-if="activeFilterCategory === 'clauses'">
        <div v-if="filteredClauses.length === 0" class="no-results">Ничего не найдено</div>
        <div v-else v-for="clause in filteredClauses" :key="clause" class="filter-div">
          <input type="checkbox" :value="clause" v-model="selectedClauses" :id="'clause-' + clause" />
          <span class="filter-label" @click="toggleFilter('clauses', clause)">{{ clause }}</span>
        </div>
      </div>
      <div v-if="activeFilterCategory === 'tags'">
        <div v-if="filteredTags.length === 0" class="no-results">Ничего не найдено</div>
        <div v-else v-for="tag in filteredTags" :key="tag" class="filter-div">
          <input type="checkbox" :value="tag" v-model="selectedTags" :id="'tag-' + tag" />
          <span class="filter-label" @click="toggleFilter('tags', tag)">{{ tag }}</span>
        </div>
      </div>
    </div>
    <div v-if="showModal" class="modal" @click="closeModal">
      <div class="modal-content" @click.stop>
        <span class="close-button" @click="closeModal">&times;</span>
        <div v-if="selectedFeature">
          <button @click="backToList">Обратно к списку</button>
          <div class="main-info" v-if="selectedFeature.properties.imageFilename">
            <h3>{{ selectedFeature.properties.name || 'Без названия' }}</h3>
            <img :src="getImageUrl(selectedFeature.properties.imageFilename)" alt="Feature Image" class="feature-image" />
          </div>
          <div v-if="selectedFeature.properties.linkink">
            <a :href="selectedFeature.properties.linkink" target="_blank">{{ selectedFeature.properties.linkink }}</a>
          </div>
          <div v-for="(content, index) in selectedFeature.properties.main" :key="index" v-html="content"></div>
          <div v-if="selectedFeature.properties.address?.length">
            <span class="tag-label">Адрес для писем:</span>
            <div v-for="(addr, index) in selectedFeature.properties.address" :key="index" v-html="addr"></div>
          </div>
          <div v-if="selectedFeature.properties.clauses?.length">
            <span class="tag-label">Статьи:</span>
            <ul>
              <li v-for="clause in selectedFeature.properties.clauses" :key="clause">{{ clause }}</li>
            </ul>
          </div>
          <div v-if="selectedFeature.properties.sourceUrl">
            <a :href="selectedFeature.properties.sourceUrl" target="_blank">Поддержка политзаключённых. Мемориал</a>
          </div>
        </div>
        <div v-else>
          <ul>
            <li v-for="feature in currentFeatures" :key="feature.properties.name" @click="showFeatureDetails(feature)" class="feature-list-item">{{ feature.properties.name || 'Без названия' }}</li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import mapboxgl from 'mapbox-gl'
import MapboxLanguage from '@mapbox/mapbox-gl-language'
import noUiSlider from 'nouislider'
import 'nouislider/dist/nouislider.css'

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
        'оправдание терроризма'
      ],
      selectedClauses: [],
      selectedNames: [],
      selectedTags: [],
      filtersVisible: false,
      activeFilterCategory: null,
      displayedFeatureCount: 0,
      searchQuery: '',
      isGraphMode: false,
      monthlyData: [],
      maxCount: 0,
      yearLabels: [],
      selectedYear: null,
      selectedMonth: null,
      minDate: new Date(2000, 0, 1),
      maxDate: new Date(2024, 11, 31),
      startDate: null,
      endDate: null,
      totalMonths: null,
      selectedRangeMonths: null,
      dateRangeSlider: null,
      tooltipData: null,
      tooltipStyle: null,
      isDragging: false,
      dragStartX: null,
      isPlaying: false,
      isPlayingBackwards: false,
      isPaused: true,
      playbackInterval: null,
      playbackSpeed: 1,
      dateSliderEnabled: false,
      playbackSlider: null,
      allowedPlaybackValues: [-8, -4, -2, -1, 0, 1, 2, 4, 8],
      lastNonZeroIndex: 5 // corresponds to allowedPlaybackValues[5] = 1x forward
    }
  },
  computed: {
    hasActiveClauseFilters() {
      return this.selectedClauses.length > 0
    },
    hasActiveTagFilters() {
      return this.selectedTags.length > 0
    },
    hasActiveNameFilters() {
      return this.selectedNames.length > 0
    },
    hasAnyActiveFilters() {
      return this.hasActiveClauseFilters || this.hasActiveTagFilters || this.hasActiveNameFilters || this.dateSliderEnabled
    },
    filteredNames() {
      if (!this.searchQuery) return this.names
      const q = this.searchQuery.toLowerCase()
      return this.names.filter(n => n.toLowerCase().includes(q))
    },
    filteredClauses() {
      if (!this.searchQuery) return this.clauses
      const q = this.searchQuery.toLowerCase()
      return this.clauses.filter(c => c.toLowerCase().includes(q))
    },
    filteredTags() {
      if (!this.searchQuery) return this.tags
      const q = this.searchQuery.toLowerCase()
      return this.tags.filter(t => t.toLowerCase().includes(q))
    },
    formattedStartDate() {
      return this.startDate ? this.startDate.toLocaleDateString('ru-RU', { year: 'numeric', month: 'long' }) : ''
    },
    formattedEndDate() {
      return this.endDate ? this.endDate.toLocaleDateString('ru-RU', { year: 'numeric', month: 'long' }) : ''
    },
    canPlay() {
      return this.isGraphMode && this.monthlyData.length > 0
    },
    rangeOverlayStyle() {
      if (!this.startDate || !this.endDate) return { display: 'none' }
      const BAR_WIDTH = 3
      const BAR_MARGIN = 4
      const TOTAL_BAR_WIDTH = BAR_WIDTH + BAR_MARGIN
      const sy = this.startDate.getFullYear()
      const sm = this.startDate.getMonth()
      const ey = this.endDate.getFullYear()
      const em = this.endDate.getMonth()
      const left = ((sy - 2000) * (TOTAL_BAR_WIDTH * 12)) + (sm * TOTAL_BAR_WIDTH)
      const monthsDiff = ((ey - sy) * 12) + (em - sm) + 1
      const width = monthsDiff * TOTAL_BAR_WIDTH
      return { left: left + 'px', width: width + 'px' }
    }
  },
  watch: {
    selectedClauses() {
      this.applyFilters()
    },
    selectedTags() {
      this.applyFilters()
    },
    selectedNames() {
      this.applyFilters()
    },
    startDate() {
      this.updateSliderFromDates()
    },
    endDate() {
      this.updateSliderFromDates()
    },
    isGraphMode(val) {
      this.dateSliderEnabled = val
      if (val) {
        this.$nextTick(() => { this.initializePlaybackSlider() })
      } else {
        this.destroyPlaybackInterval()
      }
      this.applyFilters()
    }
  },
  mounted() {
    this.initializeMap()
    document.addEventListener('click', this.handleClickOutside)
    document.addEventListener('keydown', this.handleKeyDown)
  },
  beforeUnmount() {
    this.stopAllPlayback()
    document.removeEventListener('mousemove', this.handleRangeDrag)
    document.removeEventListener('mouseup', this.stopRangeDrag)
    document.removeEventListener('keydown', this.handleKeyDown)
  },
  methods: {
    handleKeyDown(e) {
      if (e.code === 'Space') {
        e.preventDefault()
        this.togglePauseWithMemory()
      }
    },
    togglePauseWithMemory() {
      if (!this.canPlay || !this.playbackSlider) return
      const currentIndex = parseInt(this.playbackSlider.noUiSlider.get(), 10)
      const currentValue = this.allowedPlaybackValues[currentIndex]
      if (this.isPaused) {
        // Resume from lastNonZeroIndex
        this.playbackSlider.noUiSlider.set(this.lastNonZeroIndex)
      } else {
        // Pause and store lastNonZeroIndex if current is not 0
        if (currentValue !== 0) {
          this.lastNonZeroIndex = currentIndex
        }
        // set to pause (0)
        const zeroIndex = this.allowedPlaybackValues.indexOf(0)
        this.playbackSlider.noUiSlider.set(zeroIndex)
      }
    },
    initializeMap() {


mapboxgl.accessToken = 'pk.eyJ1IjoidnVmb3JpYSIsImEiOiJjbTJybXJiaWsxOHVnMmpzYnZtZWk4ZXB1In0.24OWriNL8SBYXoLdBtO9EA'; // Replace with your Mapbox access token
   


      this.map = new mapboxgl.Map({
        container: this.$refs.mapContainer,
        style: 'mapbox://styles/mapbox/navigation-night-v1',
        center: [96.712933, 62.917018],
        zoom: 5
      })
      this.map.addControl(new MapboxLanguage({ defaultLanguage: 'ru' }))
      this.map.on('load', () => {
        this.loadGeoJSONData()
        this.initializeZoomSlider()
      })
    },
    async loadGeoJSONData() {
      const response = await fetch('/main783Date_Geo.geojson')
      this.geojsonData = await response.json()
      await Promise.all([
        new Promise((resolve, reject) => {
          this.map.loadImage('/custom-marker.png', (e, i) => { if (e) reject(e); if (!this.map.hasImage('custom-marker')) this.map.addImage('custom-marker', i); resolve() })
        }),
        new Promise((resolve, reject) => {
          this.map.loadImage('/custom-marker-hover.png', (e, i) => { if (e) reject(e); if (!this.map.hasImage('custom-marker-hover')) this.map.addImage('custom-marker-hover', i); resolve() })
        })
      ])
      this.processFeaturesByCoordinates()
      this.processDateData()
      this.addMarkers()
    },
    processFeaturesByCoordinates() {
      if (!this.geojsonData) return
      const clauseSet = new Set()
      const nameSet = new Set()
      const dates = []
      this.geojsonData.features.forEach(f => {
        if (f.properties.date) {
          const d = new Date(f.properties.date)
          if (!isNaN(d)) dates.push(d)
        }
        if (Array.isArray(f.properties.clauses)) f.properties.clauses.forEach(c => clauseSet.add(c))
        if (f.properties.name) nameSet.add(f.properties.name)
      })
      if (dates.length > 0) {
        this.minDate = new Date(2000, 0, 1)
        this.maxDate = new Date(2024, 11, 31)
        this.startDate = this.minDate
        this.endDate = this.maxDate
        this.selectedRangeMonths = null
      }
      this.clauses = Array.from(clauseSet).sort()
      this.names = Array.from(nameSet).sort()
      this.displayedFeatureCount = this.geojsonData.features.length
    },
    processDateData() {
      if (!this.geojsonData?.features) return
      const datedFeatures = this.geojsonData.features.filter(f => f.properties.date && !isNaN(new Date(f.properties.date)))
      if (datedFeatures.length === 0) return
      this.minDate = new Date(2000, 0, 1)
      this.maxDate = new Date(2024, 11, 31)
      this.startDate = new Date(this.minDate)
      this.endDate = new Date(this.maxDate)
      const monthCounts = {}
      datedFeatures.forEach(ft => {
        const d = new Date(ft.properties.date)
        const k = d.getFullYear() + '-' + String(d.getMonth() + 1).padStart(2, '0')
        monthCounts[k] = (monthCounts[k] || 0) + 1
      })
      const md = []
      let cy = this.minDate.getFullYear()
      let cm = this.minDate.getMonth()
      const ey = this.maxDate.getFullYear()
      const em = this.maxDate.getMonth()
      while (cy < ey || (cy === ey && cm <= em)) {
        const k = cy + '-' + String(cm + 1).padStart(2, '0')
        const c = monthCounts[k] || 0
        const date = new Date(cy, cm)
        md.push({ date: k, count: c, label: date.toLocaleDateString('ru-RU', { year: 'numeric', month: 'long' }), year: cy, month: cm })
        cm++
        if (cm > 11) { cm = 0; cy++ }
      }
      this.monthlyData = md
      this.maxCount = Math.max(...md.map(m => m.count), 1)
      const ys = [...new Set(md.map(m => m.year))]
      this.yearLabels = ys.map(y => ({ year: y, width: (md.filter(m => m.year === y).length / md.length) * 100 }))
      this.totalMonths = ((this.maxDate.getFullYear() - this.minDate.getFullYear()) * 12) + (this.maxDate.getMonth() - this.minDate.getMonth()) + 1
      this.selectedRangeMonths = this.totalMonths
      this.$nextTick(() => { this.initializeDateRangeSlider(); this.scrollHistogramToEnd() })
    },
    updateMonthlyData(filteredFeatures) {
      const datedFiltered = filteredFeatures.filter(f => f.properties.date && !isNaN(new Date(f.properties.date)))
      const monthCounts = {}
      datedFiltered.forEach(ft => {
        const d = new Date(ft.properties.date)
        const k = `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}`
        monthCounts[k] = (monthCounts[k] || 0) + 1
      })
      const md = []
      let cy = this.minDate.getFullYear()
      let cm = this.minDate.getMonth()
      const ey = this.maxDate.getFullYear()
      const em = this.maxDate.getMonth()
      while (cy < ey || (cy === ey && cm <= em)) {
        const k = cy + '-' + String(cm + 1).padStart(2, '0')
        const c = monthCounts[k] || 0
        const date = new Date(cy, cm)
        md.push({ date: k, count: c, label: date.toLocaleDateString('ru-RU', { year: 'numeric', month: 'long' }), year: cy, month: cm })
        cm++
        if (cm > 11) { cm = 0; cy++ }
      }
      this.monthlyData = md
      this.maxCount = Math.max(...md.map(m => m.count), 1)
      const ys = [...new Set(md.map(m => m.year))]
      this.yearLabels = ys.map(y => ({ year: y, width: (md.filter(m => m.year === y).length / md.length) * 100 }))
      this.$nextTick(() => { this.scrollHistogramToEnd() })
    },
    applyFilters() {
      const filtered = this.getFilteredFeatures()
      let dateFiltered = filtered
      if (this.dateSliderEnabled) {
        dateFiltered = filtered.filter(feature => {
          if (!feature.properties.date) return false
          const fd = new Date(feature.properties.date)
          return fd >= this.startDate && fd <= this.endDate
        })
      }
      this.displayedFeatureCount = dateFiltered.length
      this.addMarkers(dateFiltered)
      if (this.isGraphMode) {
        this.updateMonthlyData(filtered)
      } else {
        this.updateMonthlyData(filtered)
      }
    },
    getFilteredFeatures() {
      if (!this.geojsonData?.features) return []
      return this.geojsonData.features.filter(feat => {
        if (this.hasActiveNameFilters && !this.selectedNames.includes(feat.properties.name)) return false
        if (this.hasActiveClauseFilters && !feat.properties.clauses?.some(c => this.selectedClauses.includes(c))) return false
        if (this.hasActiveTagFilters && !feat.properties.tags?.some(t => this.selectedTags.includes(t))) return false
        return true
      })
    },
    zoomIn() {
      this.map.easeTo({ zoom: this.map.getZoom() + 1 })
    },
    zoomOut() {
      this.map.easeTo({ zoom: this.map.getZoom() - 1 })
    },
    handleSearch() {
      if (!this.searchQuery) return
      const isNum = /^\d/.test(this.searchQuery)
      const cat = isNum ? 'clauses' : 'names'
      this.filtersVisible = true
      this.activeFilterCategory = cat
    },
    handleSearchEnter() {
      const isNum = /^\d/.test(this.searchQuery)
      const items = isNum ? this.filteredClauses : this.filteredNames
      if (items.length === 1) {
        const r = items[0]
        if (isNum && !this.selectedClauses.includes(r)) this.selectedClauses.push(r)
        else if (!isNum && !this.selectedNames.includes(r)) this.selectedNames.push(r)
        this.clearSearch()
        this.closeFiltersPanel()
        this.applyFilters()
      }
    },
    clearSearch() {
      this.searchQuery = ''
    },
    closeFiltersPanel() {
      this.filtersVisible = false
      this.activeFilterCategory = null
    },
    setActiveFilter(cat) {
      if (this.activeFilterCategory === cat) this.closeFiltersPanel()
      else {
        this.activeFilterCategory = cat
        this.filtersVisible = true
        this.clearSearch()
      }
    },
    toggleFilter(type, val) {
      if (type === 'names') {
        if (this.selectedNames.includes(val)) this.selectedNames = this.selectedNames.filter(n => n !== val)
        else this.selectedNames = [...this.selectedNames, val]
      }
      if (type === 'clauses') {
        if (this.selectedClauses.includes(val)) this.selectedClauses = this.selectedClauses.filter(c => c !== val)
        else this.selectedClauses = [...this.selectedClauses, val]
      }
      if (type === 'tags') {
        if (this.selectedTags.includes(val)) this.selectedTags = this.selectedTags.filter(t => t !== val)
        else this.selectedTags = [...this.selectedTags, val]
      }
      this.applyFilters()
    },
    resetFilters() {
      this.selectedClauses = []
      this.selectedTags = []
      this.selectedNames = []
      this.selectedMonth = null
      this.selectedYear = null
      this.isGraphMode = false
      this.stopAllPlayback()
      this.dateSliderEnabled = false
      if (this.minDate && this.maxDate) {
        this.startDate = this.minDate
        this.endDate = this.maxDate
      }
      this.filtersVisible = false
      this.activeFilterCategory = null
      this.clearSearch()
      if (this.dateRangeSlider && this.dateRangeSlider.noUiSlider) this.dateRangeSlider.noUiSlider.destroy()
      this.selectedRangeMonths = this.totalMonths
      this.$nextTick(() => { this.initializeDateRangeSlider() })
      this.applyFilters()
    },
    toggleGraphMode() {
      this.stopAllPlayback()
      this.isGraphMode = !this.isGraphMode
      this.dateSliderEnabled = this.isGraphMode
      if (!this.isGraphMode) {
        this.selectedRangeMonths = this.totalMonths
        this.startDate = this.minDate
        this.endDate = this.maxDate
        this.applyFilters()
      } else {
        this.$nextTick(() => {
          this.selectedRangeMonths = this.totalMonths
          if (this.dateRangeSlider && this.dateRangeSlider.noUiSlider) this.dateRangeSlider.noUiSlider.set(this.totalMonths)
          this.scrollHistogramToEnd()
          this.applyFilters()
        })
      }
    },
    isMonthActive(m) {
      if (this.selectedMonth) return this.selectedMonth === m.date
      if (this.selectedYear) return m.year === this.selectedYear
      return true
    },
    isInCurrentRange(m) {
      if (!this.dateSliderEnabled) return true
      if (!this.startDate || !this.endDate) return true
      const md = new Date(m.year, m.month)
      return md >= this.startDate && md <= this.endDate
    },
    isYearInRange(y) {
      if (!this.dateSliderEnabled) return false
      if (!this.startDate || !this.endDate) return false
      const ys = new Date(y, 0, 1)
      const ye = new Date(y, 11, 31)
      return ((this.startDate >= ys && this.startDate <= ye) || (this.endDate >= ys && this.endDate <= ye) || (this.startDate <= ys && this.endDate >= ye))
    },
    handleHistogramClick(e) {
      if (!this.dateSliderEnabled) return
      const c = e.currentTarget
      const r = c.getBoundingClientRect()
      const x = e.clientX - r.left
      const bw = r.width / this.monthlyData.length
      const idx = Math.floor(x / bw)
      if (idx >= 0 && idx < this.monthlyData.length) this.handleBarClick(this.monthlyData[idx])
    },
    handleBarClick(m) {
      if (!this.dateSliderEnabled) return
      const p = m.date.split('-')
      const y = parseInt(p[0], 10)
      const mo = parseInt(p[1], 10) - 1
      this.endDate = new Date(y, mo, new Date(y, mo + 1, 0).getDate())
      this.updateDateRangeFromMonths()
    },
    handleYearClick(y) {
      if (!this.dateSliderEnabled) return
      this.selectedRangeMonths = 12
      this.endDate = new Date(y, 11, 31)
      this.updateDateRangeFromMonths()
      if (this.dateRangeSlider && this.dateRangeSlider.noUiSlider) this.dateRangeSlider.noUiSlider.set(12)
    },
    showMonthTooltip(evt, m) {
      const bar = evt.target
      const rect = bar.getBoundingClientRect()
      this.tooltipStyle = { left: rect.left + (rect.width / 2) + 'px', top: rect.top + 'px' }
      this.tooltipData = m
    },
    showYearTooltip(evt, y) {
      const el = evt.target
      const r = el.getBoundingClientRect()
      const c = this.getYearFeatureCount(y.year)
      if (c > 0) {
        this.tooltipStyle = { left: r.left + (r.width / 2) + 'px', top: r.top + 'px' }
        this.tooltipData = { label: y.year + ' год', count: c }
      }
    },
    hideTooltip() {
      this.tooltipData = null
    },
    getYearFeatureCount(y) {
      return this.monthlyData.filter(m => m.year === y).reduce((sum, m) => sum + m.count, 0)
    },
    getBarOpacity(m) {
      if (this.selectedMonth) return this.selectedMonth === m.date ? 1 : 0.3
      if (this.selectedYear) return m.year === this.selectedYear ? 1 : 0.3
      return 1
    },
    startRangeDrag(e) {
      if (!this.dateSliderEnabled) return
      this.isDragging = true
      this.dragStartX = e.clientX
      document.addEventListener('mousemove', this.handleRangeDrag)
      document.addEventListener('mouseup', this.stopRangeDrag)
    },
    handleRangeDrag(evt) {
      if (!this.isDragging || !this.dateSliderEnabled) return
      const dx = evt.clientX - this.dragStartX
      const cw = this.$el.querySelector('.histogram-container').offsetWidth
      const tm = this.monthlyData.length
      const dm = Math.round((dx / cw) * tm)
      if (dm !== 0) {
        const months = this.selectedRangeMonths
        this.startDate.setMonth(this.startDate.getMonth() + dm)
        this.endDate = new Date(this.startDate)
        this.endDate.setMonth(this.startDate.getMonth() + (months - 1))
        if (this.endDate > this.maxDate) {
          this.startDate = new Date(this.minDate)
          this.endDate = new Date(this.startDate)
          this.endDate.setMonth(this.startDate.getMonth() + (months - 1))
        } else if (this.startDate < this.minDate) {
          this.endDate = new Date(this.maxDate)
          this.startDate = new Date(this.endDate)
          this.startDate.setMonth(this.endDate.getMonth() - (months - 1))
        }
        this.applyFilters()
        this.updateSliderFromDates()
        this.dragStartX = evt.clientX
      }
    },
    stopRangeDrag() {
      this.isDragging = false
      document.removeEventListener('mousemove', this.handleRangeDrag)
      document.removeEventListener('mouseup', this.stopRangeDrag)
    },
    scrollHistogramToEnd() {
      this.$nextTick(() => {
        const w = this.$el.querySelector('.histogram-wrapper')
        if (w) w.scrollLeft = w.scrollWidth
      })
    },
    scrollToCurrentRange() {
      if (!this.dateSliderEnabled) return
      if (!this.startDate || !this.endDate) return
      const c = this.$el.querySelector('.histogram-wrapper')
      const sc = this.$el.querySelector('.histogram-scroll-container')
      if (!c || !sc) return
      const sk = this.startDate.getFullYear() + '-' + String(this.startDate.getMonth() + 1).padStart(2, '0')
      const idx = this.monthlyData.findIndex(m => m.date === sk)
      if (idx === -1) return
      const tw = sc.scrollWidth
      const cw = c.offsetWidth
      const bw = tw / this.monthlyData.length
      const tp = (bw * idx) - (cw / 3)
      c.scrollTo({ left: Math.max(0, tp), behavior: 'smooth' })
    },
    moveRange(step) {
      if (!this.dateSliderEnabled) return
      if (!this.startDate || !this.endDate) return
      const m = this.selectedRangeMonths
      const ns = new Date(this.startDate)
      ns.setMonth(ns.getMonth() + step)
      const ne = new Date(ns)
      ne.setMonth(ns.getMonth() + (m - 1))
      if (ne > this.maxDate) {
        this.startDate = new Date(this.minDate)
        this.endDate = new Date(this.startDate)
        this.endDate.setMonth(this.startDate.getMonth() + (m - 1))
      } else if (ns < this.minDate) {
        this.endDate = new Date(this.maxDate)
        this.startDate = new Date(this.endDate)
        this.startDate.setMonth(this.endDate.getMonth() - (m - 1))
      } else {
        this.startDate = ns
        this.endDate = ne
      }
      this.applyFilters()
      this.updateSliderFromDates()
      this.scrollToCurrentRange()
    },
    stopAllPlayback() {
      this.isPlaying = false
      this.isPlayingBackwards = false
      this.isPaused = true
      if (this.playbackInterval) clearInterval(this.playbackInterval)
      this.playbackInterval = null
    },
    destroyPlaybackInterval() {
      if (this.playbackInterval) clearInterval(this.playbackInterval)
      this.playbackInterval = null
    },
    setPlaybackSpeedAndDirectionIndex(idx) {
      // idx is an integer 0-8
      const value = this.allowedPlaybackValues[idx]
      if (value === 0) {
        this.isPlaying = false
        this.isPlayingBackwards = false
        this.isPaused = true
        this.destroyPlaybackInterval()
      } else {
        this.isPaused = false
        this.playbackSpeed = Math.abs(value)
        if (value > 0) {
          this.isPlaying = true
          this.isPlayingBackwards = false
        } else {
          this.isPlaying = true
          this.isPlayingBackwards = true
        }
        this.updatePlaybackInterval()
      }
    },
    updatePlaybackInterval() {
      this.destroyPlaybackInterval()
      if (!this.isPlaying) return
      const it = 1000 / this.playbackSpeed
      this.playbackInterval = setInterval(() => {
        this.moveRange(this.isPlayingBackwards ? -1 : 1)
      }, it)
    },
    initializeZoomSlider() {
      const e = this.$refs.zoomSlider
      if (!e || !this.map) return
      const mi = this.map.getMinZoom()
      const ma = this.map.getMaxZoom()
      const cz = this.map.getZoom()
      noUiSlider.create(e, { start: [cz], orientation: 'vertical', direction: 'rtl', range: { min: mi, max: ma }, step: 0.5, tooltips: false })
      e.noUiSlider.on('slide', v => { this.map.easeTo({ zoom: parseFloat(v[0]) }) })
      this.map.on('zoom', () => { e.noUiSlider.set([this.map.getZoom()]) })
    },
    initializePlaybackSlider() {
      if (!this.$refs.playbackSlider) return
      const e = this.$refs.playbackSlider
      if (e.noUiSlider) e.noUiSlider.destroy()

      // We'll map our allowed values to indices: 0..8
      // allowedPlaybackValues: [-8,-4,-2,-1,0,1,2,4,8]
      // indices:                0   1   2   3 4 5 6 7 8
      // Default start at index of 0 value => index=4
      const startIndex = this.allowedPlaybackValues.indexOf(0) // 0 speed -> pause
      const format = {
        to: (v) => {
          const val = this.allowedPlaybackValues[v]
          if (val === 0) return 'Pause'
          return val + 'x'
        },
        from: () => {
          // not used since we don't let user type values
          return 4
        }
      }

      noUiSlider.create(e, {
        start: [startIndex],
        range: {
          'min': 0,
          'max': 8
        },
        step: 1,
        tooltips: [format],
        pips: {
          mode: 'values',
          values: [0,1,2,3,4,5,6,7,8],
          format: {
            to: (v) => {
              const val = this.allowedPlaybackValues[v]
              if (val === 0) return 'Pause'
              return val + 'x'
            }
          },
          density: 10
        }
      })

      this.playbackSlider = e
      e.noUiSlider.on('update', (vals) => {
        const idx = parseInt(vals[0], 10)
        this.setPlaybackSpeedAndDirectionIndex(idx)
      })
    },
    initializeDateRangeSlider() {
      const el = this.$refs.dateRangeSlider
      if (!el) return
      if (this.dateRangeSlider && this.dateRangeSlider.noUiSlider) this.dateRangeSlider.noUiSlider.destroy()

      const maxVal = this.totalMonths
      const format = {
        to: (v) => {
          const val = parseInt(v, 10)
          if (val === maxVal) return '2000 - 2024'
          if (val === 120) return '10л'
          if (val === 60) return '5л'
          if (val === 48) return '4г'
          if (val === 36) return '3г'
          if (val === 24) return '2г'
          if (val === 18) return '1.5г'
          if (val === 12) return '1г'
          if (val === 6) return '6м'
          if (val === 3) return '3м'
          if (val === 1) return '1м'
          if (val > 12) {
            const yrs = Math.floor(val / 12)
            const ms = val % 12
            return ms === 0 ? yrs + 'г' : yrs + ' г ' + ms + 'м'
          }
          return val + 'м'
        },
        from: () => {}
      }

      noUiSlider.create(el, {
        start: [this.selectedRangeMonths],
        connect: [true, false],
        direction: 'rtl',
        range: {
          'min': [1],
          '10%': [3],
          '20%': [6],
          '30%': [12],
          '40%': [18],
          '50%': [24],
          '60%': [36],
          '70%': [60],
          '80%': [120],
          'max': [maxVal]
        },
        step: null,
        tooltips: [format],
        pips: {
          mode: 'values',
          values: [maxVal, 120, 60, 36, 24, 18, 12, 6, 3, 1],
          format: format,
          density: 4
        }
      })
      this.dateRangeSlider = el
      el.noUiSlider.on('update', (vals) => {
        const nrm = Math.round(parseFloat(vals[0]))
        if (this.selectedRangeMonths !== nrm) {
          this.selectedRangeMonths = nrm
          this.updateDateRangeFromMonths()
        }
      })
      const pipEls = el.querySelectorAll('.noUi-value')
      pipEls.forEach(p => {
        p.style.cursor = 'pointer'
        p.addEventListener('click', ev => {
          const txt = ev.target.innerText
          let val = maxVal
          if (txt.includes('10л')) val = 120
          else if (txt.includes('5л')) val = 60
          else if (txt.includes('3г')) val = 36
          else if (txt.includes('2г')) val = 24
          else if (txt.includes('1.5г')) val = 18
          else if (txt.includes('1г')) val = 12
          else if (txt.includes('6м')) val = 6
          else if (txt.includes('3м')) val = 3
          else if (txt.includes('1м')) val = 1
          else if (txt.includes('2000 - 2024')) val = maxVal
          el.noUiSlider.set(val)
        })
      })
    },
    updateDateRangeFromMonths() {
      if (!this.dateSliderEnabled) {
        this.startDate = new Date(this.minDate)
        this.endDate = new Date(this.maxDate)
        this.applyFilters()
        return
      }

      const m = this.selectedRangeMonths
      if (m === this.totalMonths) {
        this.startDate = new Date(this.minDate)
        this.endDate = new Date(this.maxDate)
      } else {
        const ad = this.endDate || this.maxDate
        const ay = ad.getFullYear()
        const am = ad.getMonth()
        const sm = ay * 12 + am - (m - 1)
        const sy = Math.floor(sm / 12)
        const sM = sm % 12
        this.startDate = new Date(sy, sM, 1)
        this.endDate = new Date(ay, am + 1, 0)
        if (this.startDate < this.minDate) {
          this.startDate = new Date(this.minDate)
          this.endDate = new Date(this.startDate)
          this.endDate.setMonth(this.startDate.getMonth() + (m - 1))
          if (this.endDate > this.maxDate) this.endDate = new Date(this.maxDate)
        }
        if (this.endDate > this.maxDate) {
          this.endDate = new Date(this.maxDate)
          this.startDate = new Date(this.endDate)
          this.startDate.setMonth(this.endDate.getMonth() - (m - 1))
          if (this.startDate < this.minDate) this.startDate = new Date(this.minDate)
        }
      }
      this.applyFilters()
    },
    updateSliderFromDates() {
      if (!this.dateRangeSlider || !this.dateRangeSlider.noUiSlider) return
      const ms = ((this.endDate.getFullYear() - this.startDate.getFullYear()) * 12) + (this.endDate.getMonth() - this.startDate.getMonth()) + 1
      if (ms !== this.selectedRangeMonths) {
        this.selectedRangeMonths = ms
        this.dateRangeSlider.noUiSlider.set(ms)
      }
    },
    addMarkers(filtered = null) {
      const feats = filtered || this.geojsonData.features
      const gj = this.prepareGeoJSONForMarkers(feats)
      if (!this.map.getSource('features')) {
        this.map.addSource('features', { type: 'geojson', data: gj, generateId: true })
        this.map.addLayer({
          id: 'custom-markers',
          type: 'symbol',
          source: 'features',
          layout: { 'icon-image': 'custom-marker', 'icon-size': ['interpolate', ['linear'], ['get', 'point_count'], 1, 0.5, 3, 0.8, 7, 1], 'icon-allow-overlap': true },
          paint: {
            'icon-opacity': [
              'case',
              ['boolean', ['feature-state', 'hover'], false], 1,
              0.7
            ]
          }
        })
        this.map.addLayer({
          id: 'custom-markers-hover',
          type: 'symbol',
          source: 'features',
          layout: { 'icon-image': 'custom-marker-hover', 'icon-size': 1.2, 'icon-allow-overlap': true },
          paint: {
            'icon-opacity': [
              'case',
              ['boolean', ['feature-state', 'hover'], false],
              1, 0
            ]
          }
        })
        let hId = null
        this.map.on('mousemove', 'custom-markers', e => {
          if (e.features.length > 0) {
            if (hId !== null) this.map.setFeatureState({ source: 'features', id: hId }, { hover: false })
            hId = e.features[0].id
            this.map.setFeatureState({ source: 'features', id: hId }, { hover: true })
            this.map.getCanvas().style.cursor = 'pointer'
          }
        })
        this.map.on('mouseleave', 'custom-markers', () => {
          if (hId !== null) this.map.setFeatureState({ source: 'features', id: hId }, { hover: false })
          hId = null
          this.map.getCanvas().style.cursor = ''
        })
        ;['custom-markers', 'custom-markers-hover'].forEach(l => {
          this.map.on('click', l, e => {
            if (e.features.length > 0) {
              const f = JSON.parse(e.features[0].properties.features)
              this.showFeatureList(f)
            }
          })
        })
      } else {
        const s = this.map.getSource('features')
        if (s) {
          s.setData(gj)
        }
      }
    },
    prepareGeoJSONForMarkers(feats) {
      const byCoord = {}
      feats.forEach(f => {
        const c = f.geometry.coordinates.join(',')
        if (!byCoord[c]) byCoord[c] = []
        byCoord[c].push(f)
      })
      return {
        type: 'FeatureCollection',
        features: Object.entries(byCoord).map(([c, gf]) => {
          const [lng, lat] = c.split(',').map(Number)
          return {
            type: 'Feature',
            geometry: { type: 'Point', coordinates: [lng, lat] },
            properties: { point_count: gf.length, features: JSON.stringify(gf) }
          }
        })
      }
    },
    showFeatureList(fs) {
      this.currentFeatures = fs
      this.selectedFeature = null
      this.showModal = true
    },
    showFeatureDetails(f) {
      this.selectedFeature = f
    },
    backToList() {
      this.selectedFeature = null
    },
    closeModal() {
      this.showModal = false
      this.selectedFeature = null
    },
    getImageUrl(fn) {
      return '/images/' + fn
    },
    handleClickOutside(e) {
      const fp = this.$refs.filtersPanel
      const ftc = this.$refs.filtersToggleContainer
      if (fp && !fp.contains(e.target) && ftc && !ftc.contains(e.target)) this.closeFiltersPanel()
    }
  }
}
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
  max-height: calc(100vh - 390px);
  overflow-y: auto;
  border-radius: 4px;
  min-width: 500px;
}
.playback-slider-container{
  min-width: 250px;
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
.Linkink{
  display: none;
}
.noUi-horizontal .noUi-tooltip{
  font-size: 8px;
}
.noUi-marker-horizontal.noUi-marker{
  display: none;
}
.noUi-horizontal{
  height: 10px;
}
.noUi-value-horizontal{
  font-size: 10px !important;
}
.noUi-horizontal .noUi-tooltip{
  background-color: black;
}
.noUi-horizontal .noUi-handle{
  background-color: black;
  height: 20px;
  box-shadow: none;
}
.noUi-connect{
  background-color: white;
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
  display: none;
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
  background: rgba(255, 255, 255, 0.2);
  pointer-events: none;
  z-index: 1;
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
  width: calc(100vw - 25px);
  height: 100%;
  display: inline-block;
}

/* Control buttons row */
.control-buttons-row {
  display: flex;
  gap: 15px;
  align-items: center;
  margin-bottom: 15px;
  width: calc(100vw - 40px);
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
  justify-content: center;
  padding-top: 10px;
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
.playback-controls {
  display: flex;
  gap: 8px;
  align-items: center;
}
.speed-controls {
  display: flex;
  gap: 8px;
  align-items: center;
  margin-left: 15px;
}
.control-button {
  padding: 8px 12px;
  background-color: rgba(0, 0, 0, 0.6);
  border: 1px solid white;
  border-radius: 4px;
  cursor: pointer;
  color: white;
  min-width: 40px;
  text-align: center;
}
.control-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
.control-button.active {
  background-color: rgba(255, 255, 255, 0.9) !important;
  color: black !important;
}

/* Ensure consistent spacing in the controls row */
.control-buttons-row {
  display: flex;
  gap: 15px;
  align-items: center;
  flex-wrap: nowrap;
  padding: 10px;
}

/* Make controls group more compact */
.controls-group {
  display: flex;
  gap: 15px;
  align-items: center;
  flex-wrap: nowrap;
}


 

/* Basic styling for histogram wrapper and scrollbar */
.histogram-wrapper {
  position: relative;
  overflow: auto;
  white-space: nowrap;
}
.histogram-scroll-container {
  position: relative;
  display: inline-block;
}
.range-selection-overlay:active {
  cursor: grabbing;
}

/* Year labels */
.year-labels {
  position: relative;
  display: flex;
  justify-content: space-between;
  font-size: 12px;
  margin-top: 4px;
}
.year-label {
  cursor: pointer;
  -webkit-user-select: none;
     -moz-user-select: none;
          user-select: none;
}
.year-label.highlighted {
  font-weight: bold;
}

/* Tooltip */
.histogram-tooltip {
  position: fixed;
  background: #333;
  color: #fff;
  padding: 6px 8px;
  border-radius: 3px;
  font-size: 12px;
  pointer-events: none;
  white-space: nowrap;
  transform: translateX(-50%);
}

/* Date sliders and controls */
.date-range-slider-container {
  width: 200px;
  margin: 0 10px;
}
.speed-slider-container {
  width: 150px;
  margin-left: 10px;
}

/* Playback and control buttons */
.control-buttons-row {
  display: flex;
  align-items: center;
}
.control-button {
  cursor: pointer;
  margin: 0 4px;
  background: #f0f0f0;
  border: none;
  padding: 6px 8px;
  transition: background 0.2s;
}
.control-button.active {
  background: #ddd;
}

/* Filters, inputs and other UI elements */
.filters-toggle-container, .filters, .date-stats, .feature-count, .search-input {
  /* Add your preferred styling */
}

/* Modal and feature list */
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.5);
}
.modal-content {
  position: absolute;
  background: #fff;
  padding: 20px;
  max-width: 500px;
  width: 100%;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
.close-button {
  float: right;
  cursor: pointer;
}

/* Adjust as needed */
.feature-list-item:hover {
  background: #f0f0f0;
  cursor: pointer;
}

/* Adjust zoom slider container */
.zoom-slider-container {
  height: 100px;
  margin: 0 6px;
  display: flex;
  align-items: center;
}

/* Additional refinements as needed */









/* Responsive styles */
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
.filters{
    height:calc(100vh - 333px);
}
.modal-content{  
  max-height: calc(100vh - 280px);
}
}
@media (max-width: 980px) {
.playback-slider-container { 
	display: block;
	width: 80%; 
	margin-left: auto;
	margin-right: auto;
}
.histogram-wrapper{
    display: block;
	width: 80%; 
	margin-left: auto;
	margin-right: auto;
}
.control-buttons-row > button{
    position: fixed;
    left: 20px;
    bottom: 20px;
    z-index: 999;
  
    padding: 8px 12px;
  background-color: black;
  border: 1px solid white;
  border-radius: 4px;
  cursor: pointer;
  text-align: left;
}
.date-slider-container > .histogram-wrapper {
	margin-bottom: 185px;
	padding-top: 20px;
}
.date-range-slider-container{
    display: block;
    width: 80%;
    margin-bottom: 60px;
    margin-left: auto;
    margin-right: auto;
}
.filter-checkbox{
    position: fixed;
    bottom: 20px;
    left: 100px
}
.controls-group{
    display: block;
    width: 100%;
    margin-bottom: 50px;
}
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