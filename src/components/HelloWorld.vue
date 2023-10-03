<template>
  <div class="checkbox-container">
    <label v-for="(option, index) in checkboxOptions" :key="index">
      <input class= "ml-6" type="checkbox" v-model="checkedOptions[index]" /> {{ option }}
    </label>
      <select
      class="
        ml-6
        bg-gray-50
        border border-gray-300
        text-gray-900 text-sm
        rounded-lg
        focus:ring-blue-500 focus:border-blue-500
      "
      v-model="selectedItem"
      @change="fetchPresignedUrl"
    >
      <option v-for="item in items" :key="item.value" :value="item.value">
        {{ item.text }}
      </option>
    </select>
    <input type="checkbox" v-model="switchValue">
  </div>
  <div class="videos-wrapper" v-if="selectedItem !== 'c'">
    <div
      class="video-container"
      @click="addPoint"
      @dblclick="handleDoubleClick"
    >
      <video
        ref="videoElement"
        :key="videoOriginal"
        v-if="videoOriginal !== ''"
        margin="20"
        width="640"
        height="360"
        controls
      >
        <source :src="videoOriginal" type="video/mp4" />
        Tu navegador no soporta el elemento de video.
      </video>
      <svg v-if="points.length" :width="videoWidth" :height="videoHeight">
        <polygon
          :points="polygonPoints"
          :fill="polygonColor"
          stroke="red"
          stroke-width="2"
        />
      </svg>
    </div>
    <div class="video-container">
      <video
        :key="videoSource"
        ref="videoElement"
        v-if="videoSource !== ''"
        width="640"
        height="360"
        controls
      >
        <source :src="videoSource" type="video/mp4" />
        Tu navegador no soporta la reproducción de videos en HTML5.
      </video>
    </div>
  </div>
  <div class="coordinates-box">
    <p v-if="points.length > 0">Coordenadas: {{ polygonPoints }}</p>
    <p></p>
  </div>
  <button class="mt-6 mb-6" @click="sendCoordinates">Enviar polígono</button>
  <div class="bar-chart mx-auto w-48" v-if="videoSource !== ''">
    <div
      v-for="(value, label) in barChartData"
      :key="label"
      class="bar flex items-center justify-between mb-2"
    >
      {{ label }}:
      <div
        :style="{ width: value + '%' }"
        class="filled h-5 bg-blue-500 transition-width duration-300 min-w-2.5"
      ></div>
    </div>
  </div>
  <div class="videos-wrapper" v-if="selectedItem == 'c'">
  <div>
    <img :src="imageURL + '?' + timestamp" alt="Cámara en vivo" width="640">
    <!-- <img :src="imageURL"> -->
    <button v-if="imageResult == ''" class="mt-6 mb-6" @click="updateImage">segmentar imagen</button>
    <button v-if="intervalId !== null" class="mt-6 mb-6" @click="stopImageUpdate">Parar streaming</button>
  </div>
  <div v-if="imageResult !== ''">
    <img v-if="selectedItem == 'c'"  :src="imageResult" alt="Resultado de cámara" width="640">
  </div>
</div>
<div v-if="selectedItem == 'c'">
  <textarea v-model="question"></textarea>
  <button @click="queryImage">Send</button>
  <p v-if="viltResponse">{{ viltResponse }}</p>
</div>
</template>

<script>
export default {
  data() {
    return {
      checkboxOptions: ['Manzana', 'Persona', 'Auto', 'Camioneta', 'Moto', 'Botella'],
      checkedOptions: [],
      points: [],
      videoOriginal: '',
      imageURL: '',
      imageResult:'',
      timestamp: Date.now(),
      switchValue: false,
      videoSource: '',
      selectedItem: null,
      inputpresigned: null,
      viltResponse: '',
      question: '',
      intervalId: null,
      videoWidth: 640,
      videoHeight: 360,
      polygonColor: 'rgba(255, 0, 0, 0.5)',
      items: [
        { text: 'Línea', value: 'manzanas' },
        { text: 'Pasillos', value: 'people_alm' },
        { text: 'Tráfico', value: 'trafico' },
        { text: 'Reciclaje', value: 'pet'},
        { text: 'Camara', value: 'c'}
      ],
      selectedItem: '',
      barChartData: {
        Persona: 50,
        Caja: 75,
        Casco: 25,
        Chaleco: 90,
      },
    };
  },
  computed: {
    polygonPoints() {
      return this.points.map((p) => p.join(',')).join(' ');
    },
  },
  methods: {
    addPoint(event) {
      const rect = this.$refs.videoElement.getBoundingClientRect();
      const x = event.clientX - rect.left;
      const y = event.clientY - rect.top;

      const roundedX = Math.round(x);
      const roundedY = Math.round(y);

      this.points.push([roundedX, roundedY]);
    },
    stopImageUpdate() {
    if (this.intervalId) {
      clearInterval(this.intervalId);
      this.intervalId = null;
    }
  },
    updateImage() {
        this.timestamp = Date.now();  // Cambiar el timestamp fuerza a la imagen a recargarse
        let apiUrl = 'https://53e4-34-236-18-197.ngrok-free.app/segment'
        if (this.switchValue) {
            this.imageURL = 'http://47.181.86.62:8082/jpg/image.jpg'; // Cambiar por la nueva URL deseada
            //this.imageURL = 'http://47.181.86.62:8082/jpg/image.jpg'; // Cambiar por la nueva URL deseada
            apiUrl = 'https://53e4-34-236-18-197.ngrok-free.app/hpe'
            console.log(this.imageURL, apiUrl)
        } else {
            this.imageURL = 'http://67.43.220.114/jpg/image.jpg'; // Colocar la URL original o una diferente si deseas
            apiUrl = 'https://53e4-34-236-18-197.ngrok-free.app/segment'
        }

        this.intervalId = setInterval(() => {

        fetch(apiUrl, {
        method: 'POST',
        headers: {'Content-Type': 'application/json',},})
        .then((response) => {
          if (!response.ok) {
            throw new Error('Error al enviar los datos a la API');
          }
          return response.json();
        })
        .then((data) => {
          if (data && data.url) {
            this.imageResult = data.url; // Update the videoOriginal property with the new URL
            console.log('URL de imagen asignada:', this.imageResult);
          } else {
            console.error('La respuesta de la API no contiene una URL válida.');
          }
        })
        .catch((error) => {
          console.error('Error al enviar las coordenadas:', error);
        });
      }, 2000);
    },
    fetchPresignedUrl() {
      const apiUrl = `https://53e4-34-236-18-197.ngrok-free.app/sign?object_name=${encodeURIComponent(this.selectedItem)}`;
      if (this.selectedItem !== 'c'){
      fetch(apiUrl, {
        method: 'POST',
        headers: {'Content-Type': 'application/json',},})
        .then((response) => {
          if (!response.ok) {
            throw new Error('Error al enviar los datos a la API');
          }
          return response.json();
        })
        .then((data) => {
          if (data && data.url) {
            this.videoOriginal = data.url; // Update the videoOriginal property with the new URL
            console.log('URL del video asignada:', this.videoOriginal);
          } else {
            console.error('La respuesta de la API no contiene una URL válida.');
          }
        })
        .catch((error) => {
          console.error('Error al enviar las coordenadas:', error);
        });}
        else{
          console.log('camerachoose')
          this.imageURL = 'http://67.43.220.114/jpg/image.jpg'
        }
        
    },
    queryImage() {
      const apiUrl = `https://53e4-34-236-18-197.ngrok-free.app/vilt?question=${encodeURIComponent(this.question)}`;
      console.log(apiUrl)
      fetch(apiUrl, {
        method: 'POST',
        headers: {'Content-Type': 'application/json',},})
        .then((response) => {
          if (!response.ok) {
            throw new Error('Error al enviar los datos a la API');
          }
          return response.json();
        })
        .then((data) => {
          if (data && data.response) {
            this.viltResponse = data.response; // Update the videoOriginal property with the new URL
            console.log('response', this.viltResponse);
          } else {
            console.error('La respuesta de la API no contiene una URL válida.');
          }
        })
        .catch((error) => {
          console.error('Error al enviar las coordenadas:', error);
        });
    },

    handleDoubleClick() {
      this.points = [];
    },
    drawChart() {
      // Aquí simplemente generamos nuevos datos de manera aleatoria para la demostración.
      // En un contexto real, podrías obtener estos datos de la API o de algún cálculo.

      for (const label in this.barChartData) {
        this.barChartData[label] = Math.floor(Math.random() * 100);
      }
    },

    updateBarChartData(newData) {
      this.barChartData = newData;
    },
    sendCoordinates() {
      const polygonData = this.points.map((p) => [p[0], p[1]]);
      const selectedClasses = this.checkboxOptions.filter(
        (_, index) => this.checkedOptions[index]
      );
      const videoProcess = this.selectedItem

      const requestData = {
        polygon: polygonData,
        classes: selectedClasses,
        source: videoProcess,
      };

      const apiUrl = 'https://53e4-34-236-18-197.ngrok-free.app/count';

      if (this.points.length > 0) {
        const pointsData = {
          points: this.points,
        };

        fetch(apiUrl, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(requestData),
        })
          .then((response) => {
            if (!response.ok) {
              throw new Error('Error al enviar los datos a la API');
            }
            return response.json();
          })
          .then((data) => {
            if (data && data.url) {
              this.videoSource = data.url;
              this.barChartData = data.elements
              console.log('URL del video asignada:', this.videoSource);
            } else {
              console.error(
                'La respuesta de la API no contiene una URL válida.'
              );
            }
          })
          .catch((error) => {
            console.error('Error al enviar las coordenadas:', error);
          });
      }
    },
  },
};
</script>

<style scoped>
.bar-chart {
  display: flex;
  flex-direction: column;
  width: 200px;
  margin: 0 auto;
}

.transition-width {
  transition-property: width;
}

.filled {
  background-color: #03142c;
  height: 20px;
  transition: width 0.3s;
}

.videos-wrapper {
  display: flex;
  justify-content: space-between;
}

.video-container {
  position: relative;
  /* width: 640px;
  height: 360px; */
  /* margin-top: 20px; */
  border-radius: 10px;
  overflow: hidden;
}

.checkbox-container {
  margin-top: 10px;
  margin-bottom: 10px;
}

.select-container {
  margin-left: 10px;
}
.coordinates-box {
  position: absolute;
  bottom: 10px;
  left: 10px;
  background-color: rgba(255, 255, 255, 0.8);
  padding: 5px;
  border: 1px solid #ccc;
  border-radius: 5px;
  pointer-events: none; /* Evita que el cuadro de texto capture clics */
}

video {
  display: block;
  width: 640px;
  height: 360px;
}

svg {
  position: absolute;
  top: 0;
  left: 0;
}
</style>
