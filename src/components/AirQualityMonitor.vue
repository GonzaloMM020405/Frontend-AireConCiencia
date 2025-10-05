<template>
  <div class="app-container">
    <!-- Header con Logo -->
    <header class="app-header">
      <div class="logo-container">
        <img src="../assets/aireconcienciaB.png" alt="AireConCiencia" class="logo">
        <div class="title-section">
          <h1>AireConCiencia</h1>
          <p class="tagline">Respira, aprende y cuida tu entorno</p>
        </div>
      </div>
      <div class="header-buttons">
        <button class="refresh-btn" @click="fetchSensorData" :disabled="loading">
          <span class="btn-icon">🔄</span>
          {{ loading ? 'Actualizando...' : 'Actualizar Datos' }}
        </button>
        <button class="refresh-btn map-btn" @click="fetchLocationData" :disabled="loadingLocation">
          <span class="btn-icon">🗺️</span>
          {{ loadingLocation ? 'Actualizando...' : 'Actualizar Mapa' }}
        </button>
      </div>
    </header>

    <!-- Contenido Principal -->
    <main class="main-content">
      <!-- Sección Superior: Ubicación y Estado Principal -->
      <div class="top-section">
        <!-- Tarjeta de Ubicación con Mapa -->
        <div class="location-card card">
          <div class="card-header">
            <span class="card-icon">📍</span>
            <h3>Ubicación del Sensor</h3>
          </div>
          <div class="card-content">
            <div class="location-info">
              <p class="location-name">Saltillo, Coahuila</p>
              <p class="sensor-info">Sensor: #31843</p>
              <div class="coordinates">
                <span class="coord-item">
                  <strong>Lat:</strong> {{ locationData.latitude?.toFixed(5) || '--' }}
                </span>
                <span class="coord-item">
                  <strong>Lon:</strong> {{ locationData.longitude?.toFixed(5) || '--' }}
                </span>
              </div>
            </div>
            
            <!-- Mapa -->
            <div class="map-container" v-if="hasLocationData">
              <div id="sensor-map" class="sensor-map"></div>
            </div>
            <div class="map-placeholder" v-else>
              <p>📍 Cargando ubicación...</p>
            </div>
          </div>
        </div>
        
        <!-- Resto de tu código permanece igual -->
        <div class="main-status-card card" :class="getStatusCardClass()">
          <div class="card-header">
            <h2>Calidad del Aire Actual</h2>
            <span class="update-time">{{ lastUpdate }}</span>
          </div>
          <div class="status-content">
            <!-- Sección de Imagen Visual -->
            <div class="visual-section">
              <img :src="getStatusImage()" :alt="getPmCategory()" class="status-image">
              <div class="image-caption">{{ getPmCategory() }}</div>
            </div>
            
            <div class="data-section">
              <div class="aqi-section">
                <div class="aqi-circle">
                  <span class="aqi-value">{{ formattedPmAvg }}</span>
                  <span class="aqi-label">PM Promedio</span>
                </div>
              </div>
              <div class="status-info">
                <div class="pm-values">
                  <div class="pm-item">
                    <span class="pm-label">PM1:</span>
                    <span class="pm-number">{{ sensorValues.pm1 || '--' }} µg/m³</span>
                  </div>
                  <div class="pm-item">
                    <span class="pm-label">PM2.5:</span>
                    <span class="pm-number">{{ sensorValues.pm25 || '--' }} µg/m³</span>
                  </div>
                  <div class="pm-item">
                    <span class="pm-label">PM10:</span>
                    <span class="pm-number">{{ sensorValues.pm10 || '--' }} µg/m³</span>
                  </div>
                </div>
                <p class="description">{{ getStatusDescription() }}</p>
              </div>
            </div>
          </div>
        </div>

        <div class="recommendation-card card">
          <div class="card-header">
            <span class="card-icon">💡</span>
            <h3>Recomendación Instantánea</h3>
          </div>
          <div class="card-content">
            <p class="recommendation-text">{{ getInstantRecommendation() }}</p>
          </div>
        </div>
      </div>

      <!-- Sección Inferior: Acciones y Más Información -->
      <div class="bottom-section">
        <div class="action-card card recomendaciones" @click="openTips">
          <div class="card-icon">📋</div>
          <h3>Ver Recomendaciones</h3>
          <p>Guía completa según calidad del aire</p>
        </div>
      
        <div class="action-card card aprender" @click="showLearning">
          <div class="card-icon">📚</div>
          <h3>Aprender sobre el aire</h3>
          <p>Información educativa</p>
        </div>


        <div class="action-card card reporte" @click="openReport">
          <div class="card-icon">⚠️</div>
        <h3>Reportar incidencia</h3>
        <p>Da clic aqui</p>
        </div>   

        <div v-if="showTips" class="modal-overlay" @click.self="closeTips">
        <div class="action-card card recomendaciones">
        <h3 class="titulo-azul">Recomendaciones según la calidad del aire</h3>
        <p class="descripcion-reporte">
          Estas sugerencias se basan en el promedio actual de partículas PM2.5 detectadas:
        </p>

        <ul class="lista-recomendaciones">
          <li>Ideal para actividades al aire libre y deportes.</li>
          <li>Calidad aceptable, ideal para la mayoría de actividades.</li>
          <li>Grupos sensibles deben considerar reducir actividades intensas al aire libre.</li>
          <li>Todas las personas pueden comenzar a experimentar efectos en la salud.</li>
          <li>Advertencia de condiciones sanitarias graves</li>
          <li>Alerta de emergencia sanitaria. Evitar actividades al aire libre.</li>
          
        </ul>

        <button class="close-btn" @click="closeTips">Cerrar</button>
      </div>
    </div>
      

           <!-- Modal para reportar incidenciaMás -->
<div v-if="showReport" class="modal-overlay" @click.self="closeReport">
  <div class="action-card card reporte">
    <h3 class="titulo-azul">Genera tu reporte </h3>
      <ul>
    <li>🏫Si detectas algún problema relacionado con la calidad del aire, el sistema o los datos mostrados, puedes enviarnos un reporte detallando la incidencia.  
  Escríbeno</li>
    <li>
    <li>
      <a>medioambiente@saltillo.gob.mx</a>

    </li>
    </li>
  </ul>
    <button class="close-btn" @click="closeReport">Cerrar</button>
  </div>
        </div>
        
      </div>
       <!-- Modal para Aprender Más -->
<div v-if="showLearningModal" class="modal-overlay" @click.self="closeLearning">
  <div class="action-card card aprender">
    <h3 class="titulo-azul">Aprende de la calidad del Aire en tu Zona</h3>
   
      <ul>
       <li>
 Todos los días, los seres humanos respiramos alrededor de 20,000 veces. Una persona adulta inhala de 13.000 a 15.000 litros de aire por día. </li>
    <li >🌫️ <strong>PM2.5</strong> son partículas diminutas que pueden llegar hasta los pulmones y el torrente sanguíneo.</li>
    <li>🌞 Los días soleados con poco viento pueden aumentar los niveles de contaminación.</li>
    <li>🌿 Plantar árboles y mantener áreas verdes mejora la calidad del aire local.</li>
    <li>🏫 Las escuelas con buena ventilación reducen los casos de alergias y fatiga en los estudiantes.</li>
    <li>
      <a href='https://www.earthdata.nasa.gov/data/instruments/tempo'>Accede a datos oficiales de la NASA</a>

    </li>
  </ul>

    <button class="close-btn" @click="closeLearning">Cerrar</button>
  </div>
</div> 
    </main>
  </div>
</template>

<script>
import Excelente from '../assets/Buena.png';
import Buena from '../assets/Moderada.png';
import Moderada from '../assets/Daninaparalasalud.png';
import PocoSaludable from '../assets/Daninaparagrupossensibles.png';
import Insalubre from '../assets/Peligrosa.png';
import Peligrosa from '../assets/Peligrosa.png';
import MuyDanina from '../assets/MuyDanina.png';

// Importar Leaflet para el mapa
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';

// Solucionar problema de iconos en Leaflet
delete L.Icon.Default.prototype._getIconUrl;
L.Icon.Default.mergeOptions({
  iconRetinaUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.7.1/images/marker-icon-2x.png',
  iconUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.7.1/images/marker-icon.png',
  shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.7.1/images/marker-shadow.png',
});

export default {
  name: 'AirQualityMonitor',
  data() {
    return {
      apiData: {},
      locationData: {},
      loading: false,
      loadingLocation: false,
      showLearningModal: false,
      showReport: false,
      showTips: false,
      error: null,
      lastUpdate: 'Cargando datos...',
      map: null,
      marker: null,
      statusImages: {
        'excelente': Excelente,
        'buena': Buena,
        'moderada': Moderada,
        'poco saludable': PocoSaludable,
        'insalubre': Insalubre,
        'peligrosa': Peligrosa,
        'cargando...': MuyDanina
      }
    }
  },
  computed: {
    combinedData() {
      return this.apiData.combined || {};
    },
    sensorValues() {
      return this.apiData.values || {};
    },
    formattedPmAvg() {
      const pmAvg = this.sensorValues.pm_avg;
      return pmAvg ? pmAvg.toFixed(1) : '--';
    },
    subindices() {
      return this.apiData.subindices || {};
    },
    hasLocationData() {
      return this.locationData.latitude && this.locationData.longitude;
    },
    // URLs base desde variables de entorno
    backendBaseUrl() {
      return import.meta.env.VITE_BACKEND_URL || 'http://127.0.0.1:7777';
    }
  },
  async mounted() {
    console.log('Backend URL:', this.backendBaseUrl);
    await this.fetchInitialData();
  },
  methods: {
    showLearning() {
      this.showLearningModal = true;
    },
    closeLearning() {
      this.showLearningModal = false;
    },
    openReport() {
      this.showReport = true;
    },
    closeReport() {
      this.showReport = false;
    },
    openTips() {
      this.showTips = true;
    },
    closeTips() {
      this.showTips = false;
    },
    async fetchInitialData() {
      try {
        console.log('Iniciando carga automática de datos...');
        console.log('Backend URL:', this.backendBaseUrl);
        
        // Cargar datos del sensor y ubicación en paralelo
        await Promise.all([
          this.fetchSensorData(),
          this.fetchLocationData()
        ]);
        
        console.log('Datos cargados exitosamente');
        
      } catch (error) {
        console.error('Error en carga inicial:', error);
        this.error = 'Error al cargar los datos iniciales';
      }
    },
    async fetchSensorData() {
      this.loading = true;
      this.error = null;
      
      try {
        const backendUrl = `${this.backendBaseUrl}/api/aqi/combined`;
        console.log('Fetching sensor data from:', backendUrl);
        
        const response = await fetch(backendUrl);
        
        if (!response.ok) {
          throw new Error(`Error del servidor: ${response.status}`);
        }
        
        const data = await response.json();
        this.apiData = data;
        this.updateTime();
        
      } catch (err) {
        console.error('Error fetching sensor data:', err);
        this.error = `No se pudieron cargar los datos: ${err.message}`;
        // No mostrar alerta automáticamente en producción
        if (import.meta.env.DEV) {
          alert(this.error);
        }
      } finally {
        this.loading = false;
      }
    },

    async fetchLocationData() {
      this.loadingLocation = true;
      
      try {
        const backendUrl = `${this.backendBaseUrl}/api/location`;
        console.log('Fetching location data from:', backendUrl);
        
        const response = await fetch(backendUrl);
        
        if (!response.ok) {
          throw new Error(`Error del servidor: ${response.status}`);
        }
        
        const data = await response.json();
        this.locationData = data;

        setTimeout(() => {
          this.updateMap();
        }, 100);
        
      } catch (err) {
        console.error('Error fetching location data:', err);
        // No mostrar alerta automáticamente en producción
        if (import.meta.env.DEV) {
          alert(`No se pudieron cargar los datos de ubicación: ${err.message}`);
        }
      } finally {
        this.loadingLocation = false;
      }
    },

    updateMap() {
      if (!this.hasLocationData) return;

      const { latitude, longitude } = this.locationData;
      
      // Si el mapa no existe, crearlo
      if (!this.map) {
        this.map = L.map('sensor-map').setView([latitude, longitude], 15);
        
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
          attribution: '© OpenStreetMap contributors',
          maxZoom: 18
        }).addTo(this.map);
        
        // Crear marcador personalizado
        const sensorIcon = L.divIcon({
          html: '📍',
          iconSize: [30, 30],
          className: 'sensor-marker'
        });
        
        this.marker = L.marker([latitude, longitude], { icon: sensorIcon })
          .addTo(this.map)
          .bindPopup(`
            <div class="map-popup">
              <strong>Sensor de Calidad del Aire</strong><br>
              Lat: ${latitude.toFixed(5)}<br>
              Lon: ${longitude.toFixed(5)}<br>
              Estado: ${this.getPmCategory()}
            </div>
          `);
      } else {
        // Actualizar posición existente
        this.map.setView([latitude, longitude], 15);
        this.marker.setLatLng([latitude, longitude]);
        this.marker.getPopup().setContent(`
          <div class="map-popup">
            <strong>Sensor de Calidad del Aire</strong><br>
            Lat: ${latitude.toFixed(5)}<br>
            Lon: ${longitude.toFixed(5)}<br>
            Estado: ${this.getPmCategory()}
          </div>
        `);
      }
    },

    updateTime() {
      const now = new Date();
      this.lastUpdate = `Actualizado: ${now.getHours()}:${now.getMinutes().toString().padStart(2, '0')}`;
    },

    getPmCategory() {
      const pmAvg = this.sensorValues.pm_avg;
      if (!pmAvg) return 'Cargando...';
      
      if (pmAvg <= 12) return 'Excelente';
      if (pmAvg <= 35) return 'Buena';
      if (pmAvg <= 55) return 'Moderada';
      if (pmAvg <= 150) return 'Poco Saludable';
      if (pmAvg <= 250) return 'Insalubre';
      return 'Peligrosa';
    },

    getStatusImage() {
      const category = this.getPmCategory().toLowerCase();
      return this.statusImages[category] || this.statusImages['cargando...'];
    },

    getStatusCardClass() {
      const pmAvg = this.sensorValues.pm_avg;
      if (!pmAvg) return 'status-unknown';
      
      if (pmAvg <= 12) return 'status-good';
      if (pmAvg <= 35) return 'status-moderate';
      if (pmAvg <= 55) return 'status-unhealthy-sensitive';
      if (pmAvg <= 150) return 'status-unhealthy';
      if (pmAvg <= 250) return 'status-very-unhealthy';
      return 'status-hazardous';
    },

    getStatusDescription() {
      const pmAvg = this.sensorValues.pm_avg;
      if (!pmAvg) return 'Cargando información más reciente...';
      
      if (pmAvg <= 12) return 'La calidad del aire es ideal para actividades al aire libre.';
      if (pmAvg <= 35) return 'Calidad aceptable, ideal para la mayoría de actividades.';
      if (pmAvg <= 55) return 'Grupos sensibles deben considerar reducir actividades intensas al aire libre.';
      if (pmAvg <= 150) return 'Todas las personas pueden comenzar a experimentar efectos en la salud.';
      if (pmAvg <= 250) return 'Advertencia de condiciones sanitarias graves.';
      return 'Alerta de emergencia sanitaria. Evitar actividades al aire libre.';
    },

    getInstantRecommendation() {
      const pmAvg = this.sensorValues.pm_avg;
      if (!pmAvg) return 'Actualiza los datos para ver recomendaciones.';
      
      if (pmAvg <= 12) return 'Ideal para actividades al aire libre y deportes.';
      if (pmAvg <= 35) return 'Puedes realizar actividades normales al aire libre.';
      if (pmAvg <= 55) return 'Personas sensibles deben reducir actividades intensas al aire libre.';
      if (pmAvg <= 150) return 'Considera realizar actividades bajo techo.';
      if (pmAvg <= 250) return 'Realiza actividades bajo techo.';
      return 'Evita actividades al aire libre. Permanece en interiores.';
    }
  }
}
</script>

<style scoped>
.app-container {
  min-height: 100vh;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  padding: 10px;
}

/* Header Styles Mejorado para Móviles */
.app-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 20px;
  padding: 0 5px;
  gap: 15px;
  text-align: center;
}

.logo-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  width: 100%;
}

.logo {
  width: 120px;
  padding: 4px;
  height: auto;
  border-radius: 4px;
  background-color: white;
}

.title-section h1 {
  font-size: 1.8em;
  font-weight: 700;
  color: white;
  margin-bottom: 5px;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
  line-height: 1.2;
}

.tagline {
  font-size: 1em;
  color: rgba(255, 255, 255, 0.95);
  font-weight: 400;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.2);
}

.header-buttons {
  display: flex;
  flex-direction: column;
  gap: 10px;
  width: 100%;
  max-width: 400px;
}

.refresh-btn {
  background: rgba(255, 255, 255, 0.25);
  border: 2px solid rgba(255, 255, 255, 0.4);
  color: white;
  padding: 12px 20px;
  border-radius: 25px;
  cursor: pointer;
  font-size: 1em;
  font-weight: 600;
  backdrop-filter: blur(10px);
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
  width: 100%;
}

.refresh-btn:hover:not(:disabled) {
  background: rgba(255, 255, 255, 0.35);
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.25);
}

.refresh-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none;
}

.btn-icon {
  font-size: 1.2em;
}

.map-btn {
  background: rgba(255, 255, 255, 0.2);
}

/* Main Content */
.main-content {
  max-width: 1400px;
  margin: 0 auto;
  padding: 0 5px;
}

/* Card Base Styles - Optimizado para móviles */
.card {
  background: white;
  border-radius: 16px;
  padding: 0;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.12);
  transition: all 0.3s ease;
  overflow: hidden;
  border: 1px solid rgba(255, 255, 255, 0.1);
  margin-bottom: 15px;
}

.card-header {
  background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
  padding: 18px 20px;
  border-bottom: 2px solid #e9ecef;
  display: flex;
  align-items: center;
  gap: 12px;
}

.card-header h2,
.card-header h3 {
  color: #2c3e50;
  margin: 0;
  font-size: 1.2em;
  font-weight: 600;
}

.card-header h2 {
  font-size: 1.3em;
}

.card-content {
  padding: 20px;
}

.card-icon {
  font-size: 1.4em;
  width: 24px;
  text-align: center;
}

/* Top Section - Reorganizado para móviles */
.top-section {
  display: flex;
  flex-direction: column;
  gap: 15px;
  margin-bottom: 20px;
}

/* Location Card Mejorada para móviles */
.location-card {
  display: flex;
  flex-direction: column;
}

.location-info {
  margin-bottom: 15px;
}

.location-name {
  font-size: 1.4em;
  font-weight: bold;
  color: #667eea;
  margin-bottom: 6px;
  text-align: center;
}

.sensor-info {
  font-size: 0.9em;
  color: #6c757d;
  font-weight: 500;
  margin-bottom: 10px;
  text-align: center;
}

.coordinates {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 15px;
}

.coord-item {
  background: #f8f9fa;
  padding: 8px 12px;
  border-radius: 8px;
  font-size: 0.85em;
  color: #495057;
  text-align: center;
}

/* Mapa optimizado para móviles */
.map-container {
  margin-top: 10px;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 3px 10px rgba(0, 0, 0, 0.1);
}

.sensor-map {
  height: 200px;
  width: 100%;
  border-radius: 12px;
}

.map-placeholder {
  height: 200px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f8f9fa;
  border-radius: 12px;
  color: #6c757d;
  font-size: 1em;
  text-align: center;
  padding: 20px;
}

/* Main Status Card - Reorganizado para móviles */
.main-status-card {
  min-height: auto;
}

.status-content {
  display: flex;
  flex-direction: column;
  gap: 20px;
  padding: 20px;
  align-items: center;
}

/* Sección de Imagen Visual optimizada */
.visual-section {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  order: -1;
}

.status-image {
  width: 100%;
  max-width: 200px;
  height: auto;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  object-fit: contain;
}

.image-caption {
  font-size: 1.2em;
  font-weight: 700;
  color: #2c3e50;
  text-align: center;
  padding: 6px 14px;
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  border-radius: 20px;
  box-shadow: 0 3px 8px rgba(0, 0, 0, 0.1);
}

/* Sección de Datos optimizada */
.data-section {
  display: flex;
  flex-direction: column;
  gap: 20px;
  width: 100%;
}

.aqi-section {
  display: flex;
  justify-content: center;
}

.aqi-circle {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: white;
  font-weight: bold;
  box-shadow: 0 6px 15px rgba(0, 0, 0, 0.15);
  border: 3px solid rgba(255, 255, 255, 0.8);
}

.aqi-value {
  font-size: 2.2em;
  line-height: 1;
  font-weight: 800;
}

.aqi-label {
  font-size: 0.9em;
  opacity: 0.95;
  margin-top: 4px;
  font-weight: 600;
  text-align: center;
}

.status-info {
  text-align: center;
  width: 100%;
}

.pm-values {
  margin-bottom: 15px;
  background: #f8f9fa;
  padding: 15px;
  border-radius: 10px;
  border-left: 4px solid #667eea;
}

.pm-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 6px;
  padding: 5px 0;
}

.pm-item:last-child {
  margin-bottom: 0;
}

.pm-label {
  color: #495057;
  font-weight: 600;
  font-size: 0.9em;
}

.pm-number {
  color: #2c3e50;
  font-weight: 700;
  font-size: 1em;
  background: rgba(102, 126, 234, 0.1);
  padding: 3px 10px;
  border-radius: 6px;
}

.description {
  color: #495057;
  line-height: 1.5;
  font-size: 0.95em;
  font-weight: 500;
  text-align: center;
}

.update-time {
  font-size: 0.85em;
  color: #6c757d;
  font-weight: 500;
  background: rgba(255, 255, 255, 0.7);
  padding: 3px 10px;
  border-radius: 10px;
  text-align: center;
}

/* Recommendation Card optimizada */
.recommendation-card {
  display: flex;
  flex-direction: column;
}

.recommendation-text {
  color: #495057;
  line-height: 1.5;
  font-size: 0.95em;
  font-weight: 500;
  background: linear-gradient(135deg, #fff3cd, #ffeaa7);
  padding: 15px;
  border-radius: 10px;
  border-left: 4px solid #ffc107;
  text-align: center;
}

/* Bottom Section - Reorganizado para móviles */
.bottom-section {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.action-card {
  text-align: center;
  min-height: 140px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  padding: 15px;
  background-color: #e3f2fd;
  border-radius: 16px;
}

.action-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.15);
}

.action-card .card-icon {
  font-size: 2.5em;
  margin-bottom: 12px;
  width: auto;
  transition: transform 0.3s ease;
}

.action-card:hover .card-icon {
  transform: scale(1.1);
}

.action-card h3 {
  color: #2c3e50;
  margin-bottom: 8px;
  font-size: 1.1em;
  font-weight: 600;
}

.action-card p {
  color: #6c757d;
  font-size: 0.85em;
  line-height: 1.4;
}

/* Status Card Colors */
.status-good .aqi-circle { background: linear-gradient(135deg, #4CAF50, #45a049); }
.status-moderate .aqi-circle { background: linear-gradient(135deg, #FFC107, #FF8F00); }
.status-unhealthy-sensitive .aqi-circle { background: linear-gradient(135deg, #FF9800, #F57C00); }
.status-unhealthy .aqi-circle { background: linear-gradient(135deg, #F44336, #D32F2F); }
.status-very-unhealthy .aqi-circle { background: linear-gradient(135deg, #9C27B0, #7B1FA2); }
.status-hazardous .aqi-circle { background: linear-gradient(135deg, #795548, #5D4037); }

/* Estilos para listas en modales */
li {
  color: #2563eb;
  font-weight: 500;
  font-size: 0.95em;
  margin-bottom: 8px;
  transition: color 0.3s ease, transform 0.2s ease;
  text-align: left;
  line-height: 1.4;
}

li:hover {
  color: #1e40af;
  transform: translateX(3px);
}

.titulo-azul {
  color: #1e3a8a;
  font-size: 1.3em;
  font-weight: 700;
  margin-bottom: 10px;
  text-align: center;
  letter-spacing: 0.3px;
}

/* Modales optimizados para móviles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: 20px;
  box-sizing: border-box;
  overflow-y: auto;
}

.action-card.card.aprender,
.action-card.card.reporte,
.action-card.card.recomendaciones {
  padding: 20px;
  border-radius: 16px;
  text-align: center;
  max-width: 100%;
  width: 100%;
  margin: 10px;
  max-height: 80vh;
  overflow-y: auto;
}

.close-btn {
  margin-top: 15px;
  padding: 10px 20px;
  border: none;
  background-color: #4a90e2;
  color: white;
  border-radius: 8px;
  cursor: pointer;
  font-weight: bold;
  transition: background 0.3s;
  font-size: 0.9em;
  width: 100%;
  max-width: 200px;
}

.close-btn:hover {
  background-color: #357abd;
}

/* Popup del mapa personalizado */
.map-popup {
  text-align: center;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  font-size: 0.9em;
}

.map-popup strong {
  color: #667eea;
}

/* Marcador personalizado */
.sensor-marker {
  background: transparent !important;
  border: none !important;
  font-size: 20px;
  text-align: center;
}

/* Media Queries para tablets */
@media (min-width: 768px) {
  .app-container {
    padding: 15px;
  }
  
  .app-header {
    flex-direction: row;
    justify-content: space-between;
    text-align: left;
  }
  
  .logo-container {
    flex-direction: row;
    justify-content: flex-start;
  }
  
  .header-buttons {
    flex-direction: row;
    width: auto;
  }
  
  .refresh-btn {
    width: auto;
    min-width: 180px;
  }
  
  .top-section {
    display: grid;
    grid-template-columns: 1fr 2fr 1fr;
    gap: 20px;
  }
  
  .status-content {
    grid-template-columns: 1fr 2fr;
    flex-direction: row;
  }
  
  .visual-section {
    order: 0;
  }
  
  .bottom-section {
    grid-template-columns: repeat(3, 1fr);
    display: grid;
  }
  
  .coordinates {
    flex-direction: row;
  }
}

/* Media Queries para desktop */
@media (min-width: 1024px) {
  .app-container {
    padding: 20px;
  }
  
  .title-section h1 {
    font-size: 2.2em;
  }
  
  .status-image {
    max-width: 280px;
  }
  
  .aqi-circle {
    width: 140px;
    height: 140px;
  }
  
  .aqi-value {
    font-size: 2.8em;
  }
}

/* Mejoras específicas para pantallas muy pequeñas */
@media (max-width: 360px) {
  .app-container {
    padding: 8px;
  }
  
  .card-content {
    padding: 15px;
  }
  
  .status-image {
    max-width: 150px;
  }
  
  .aqi-circle {
    width: 100px;
    height: 100px;
  }
  
  .aqi-value {
    font-size: 1.8em;
  }
  
  .action-card {
    min-height: 120px;
    padding: 12px;
  }
  
  .action-card h3 {
    font-size: 1em;
  }
}

/* Mejora de accesibilidad para touch */
@media (hover: none) {
  .action-card:hover {
    transform: none;
  }
  
  .refresh-btn:hover:not(:disabled) {
    transform: none;
  }
  
  li:hover {
    transform: none;
  }
}

/* Soporte para orientación landscape en móviles */
@media (max-height: 500px) and (orientation: landscape) {
  .app-header {
    margin-bottom: 10px;
  }
  
  .top-section {
    margin-bottom: 10px;
  }
  
  .card {
    margin-bottom: 10px;
  }
  
  .sensor-map {
    height: 150px;
  }
  
  .map-placeholder {
    height: 150px;
  }
}
</style>