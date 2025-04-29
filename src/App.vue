<template>
  <ion-page>
    <ion-header :translucent="true">
     <ion-toolbar>
       <ion-title>Pomodoro Timer</ion-title>
     </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true" class="ion-padding">
       <ion-header collapse="condense">
         <ion-toolbar>
           <ion-title size="large">Pomodoro Timer</ion-title>
         </ion-toolbar>
       </ion-header>

       <div id="timer-container" class="ion-text-center">
         <div id="timer-state">
           <h2>{{ currentState }}</h2>
         </div>
         <div id="timer-display">
           <h1>{{ displayTime }}</h1>
         </div>
         <div id="timer-controls" class="ion-margin-top">
           <ion-button
             v-if="isPaused && timeRemaining > 0"
             @click="startTimer"
             color="success"
             expand="block"
           >
             <ion-icon slot="start" :icon="playOutline"></ion-icon>
             Start
           </ion-button>

           <ion-button
             v-if="!isPaused"
             @click="pauseTimer"
             color="warning"
             expand="block"
           >
             <ion-icon slot="start" :icon="pauseOutline"></ion-icon>
             Pause
           </ion-button>

           <ion-button
             @click="resetTimer"
             color="danger"
             fill="outline"
             expand="block"
             class="ion-margin-top"
           >
              <ion-icon slot="start" :icon="refreshOutline"></ion-icon>
             Reset
           </ion-button>
         </div>
       </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'; 
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
  IonButton, 
  IonIcon,   
} from '@ionic/vue';

import { playOutline, pauseOutline, refreshOutline } from 'ionicons/icons';


const WORK_DURATION = 25 * 60;
const SHORT_BREAK_DURATION = 5 * 60;



const timeRemaining = ref<number>(WORK_DURATION);
const displayTime = ref<string>('25:00');
const currentState = ref<string>('Work');
const isPaused = ref<boolean>(true); 

const timerInterval = ref<any>(null);


const formatDisplayTime = () => {
  if (timeRemaining.value < 0) timeRemaining.value = 0;
  const minutes = Math.floor(timeRemaining.value / 60);
  const seconds = timeRemaining.value % 60;
  displayTime.value = `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
};


const startTimer = () => {
  
  if (isPaused.value && timeRemaining.value > 0) {
    isPaused.value = false;
    
    if (timerInterval.value) clearInterval(timerInterval.value);

    timerInterval.value = setInterval(() => {
      timeRemaining.value--;
      formatDisplayTime();

      if (timeRemaining.value <= 0) {
        pauseTimer(); 
        
        console.log('Timer finished!');
      }
    }, 1000); 
  }
};

const pauseTimer = () => {
  if (timerInterval.value) {
    clearInterval(timerInterval.value);
    timerInterval.value = null;
  }
  isPaused.value = true; 
};

const resetTimer = () => {
  pauseTimer(); 
  
  timeRemaining.value = WORK_DURATION;
  currentState.value = 'Work';
  isPaused.value = true;
  formatDisplayTime();
};


onMounted(() => {
  resetTimer(); 
});

onUnmounted(() => {
  
  if (timerInterval.value) {
    clearInterval(timerInterval.value);
  }
});

</script>

<style scoped>
#timer-container {
  margin-top: 30vh;
  transform: translateY(-50%);
}

#timer-display h1 {
  font-size: 5rem;
  font-weight: bold;
  margin: 20px 0;
}

 #timer-state h2 {
    color: var(--ion-color-medium-shade);
 }

#timer-controls ion-button {
    margin-top: 10px;
}
</style>