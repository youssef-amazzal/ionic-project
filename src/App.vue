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
          <p v-if="currentState === 'Work' || currentState === 'Short Break'">
            Session: {{ sessionCount + 1 }} / {{ SESSIONS_BEFORE_LONG_BREAK }}
          </p>
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
            Start {{ currentState }} 
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

          
          <ion-grid>
            <ion-row>
              <ion-col size="6"> 
                <ion-button
                  @click="resetTimerToWork"
                  color="danger"
                  fill="outline"
                  expand="block"
                  class="button-lower"
                >
                  <ion-icon slot="start" :icon="refreshOutline"></ion-icon>
                  Reset
                </ion-button>
              </ion-col>
              <ion-col size="6"> 
                
                <ion-button
                  @click="skipTimer"
                  color="medium"
                  fill="outline"
                  expand="block"
                  :disabled="timeRemaining <= 0 && isPaused"
                  class="button-lower"
                >
                  <ion-icon slot="start" :icon="playSkipForwardOutline"></ion-icon>
                  Skip
                </ion-button>
              </ion-col>
            </ion-row>
          </ion-grid>

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
  IonGrid,
  IonRow,
  IonCol,
} from '@ionic/vue';

import {
  playOutline,
  pauseOutline,
  refreshOutline,
  playSkipForwardOutline
} from 'ionicons/icons';

enum PomodoroState {
  Work = 'Work',
  ShortBreak = 'Short Break',
  LongBreak = 'Long Break'
}

const WORK_DURATION = 25 * 60; 
const SHORT_BREAK_DURATION = 5 * 60; 
const LONG_BREAK_DURATION = 15 * 60; 
const SESSIONS_BEFORE_LONG_BREAK = 4;

const timeRemaining = ref<number>(0); 
const displayTime = ref<string>('00:00'); 
const currentState = ref<PomodoroState>(PomodoroState.Work); 
const isPaused = ref<boolean>(true); 
const timerInterval = ref<any>(null); 
const sessionCount = ref<number>(0); 

const formatDisplayTime = () => {
  if (timeRemaining.value < 0) timeRemaining.value = 0;
  const minutes = Math.floor(timeRemaining.value / 60);
  const seconds = timeRemaining.value % 60;
  
  displayTime.value = `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
};


const clearTimerInterval = () => {
  if (timerInterval.value) {
    clearInterval(timerInterval.value);
    timerInterval.value = null; 
  }
};

const startTimer = () => {
  
  if (isPaused.value && timeRemaining.value > 0) {
    isPaused.value = false;
    clearTimerInterval(); 
    
    timerInterval.value = setInterval(() => {
      timeRemaining.value--; 
      formatDisplayTime();   
      
      if (timeRemaining.value <= 0) {
        clearTimerInterval();       
        handleTimerCompletion();    
      }
    }, 1000); 
  }
};

const pauseTimer = () => {
  clearTimerInterval(); 
  isPaused.value = true; 
};


const resetTimerToWork = () => {
  clearTimerInterval(); 
  currentState.value = PomodoroState.Work; 
  timeRemaining.value = WORK_DURATION;     
  sessionCount.value = 0;                  
  isPaused.value = true;                   
  formatDisplayTime();                     
};



const handleTimerCompletion = () => {
  if (currentState.value === PomodoroState.Work) {
    sessionCount.value++; 
    console.log(`Work session ${sessionCount.value} completed.`);
    
    if (sessionCount.value >= SESSIONS_BEFORE_LONG_BREAK) {
      currentState.value = PomodoroState.LongBreak;
      timeRemaining.value = LONG_BREAK_DURATION;
      sessionCount.value = 0; 
      console.log('Starting Long Break');
    } else {
      
      currentState.value = PomodoroState.ShortBreak;
      timeRemaining.value = SHORT_BREAK_DURATION;
      console.log('Starting Short Break');
    }
  } else {
    
    currentState.value = PomodoroState.Work;
    timeRemaining.value = WORK_DURATION;
    console.log('Starting Work session');
  }

  
  isPaused.value = true;
  formatDisplayTime(); 
};


const skipTimer = () => {
  console.log(`Skipping ${currentState.value}`);
  clearTimerInterval(); 
  isPaused.value = true; 
  handleTimerCompletion(); 
};


onMounted(() => {
  
  resetTimerToWork();
  console.log('Pomodoro Timer component mounted.');
});

onUnmounted(() => {
  
  clearTimerInterval();
  console.log('Pomodoro Timer component unmounted, interval cleared.');
});

</script>


<style scoped>
#timer-container {
  margin-top: 25vh; 
  transform: translateY(-50%);
}

#timer-display h1 {
  font-size: 5rem; 
  font-weight: bold;
  margin-top: 10px; 
  margin-bottom: 25px; 
  color: var(--ion-color-primary); 
}

#timer-state h2 {
  font-size: 1.8rem;
  font-weight: 500;
  color: var(--ion-color-dark-tint); 
}

#timer-state p {
  font-size: 0.9rem;
  color: var(--ion-color-medium-shade);
  margin-top: -5px; 
  margin-bottom: 10px; 
}

#timer-controls ion-button {
  margin-top: 10px;
}


ion-grid ion-button.button-lower {
  margin-top: 0;
}

ion-grid {
    margin-top: 15px; 
}

</style>