<template>
  <v-app>

  <!-- Sizes your content based upon application components -->
    <v-content id='mainContainer'>

      <!-- Provides the application the proper gutter -->
      <v-container fluid>

        <p class="text-center font-weight-black"
          v-bind:class="[{ 'red--text': !timerStarted},
            {'display-4': displaySizeSelected == '4'},
            {'display-3': displaySizeSelected == '3'},
            {'display-2': displaySizeSelected == '2'},
            ]"
        >
          {{timerDisplay}}
        </p>

        <v-row justify="center">
          <v-dialog v-model="settings" persistent max-width="300px">
            <v-card>
              <v-card-title>
                <span class="headline">Settings</span>
              </v-card-title>
              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12">
                      <v-switch
                        v-model="soundOn"
                        :label="`Sound: ${soundOn.toString()}`"
                      ></v-switch>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col cols="12">
                      <v-select
                        v-model="countdownSelected"
                        :items='countdownItems'
                        label='Countdown Seconds'
                        @change="countdown = countdownSelected"
                      ></v-select>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col cols="12">
                      <v-select
                        v-model="displaySizeSelected"
                        :items='displaySizeItems'
                        label='Display Size'
                        @change="countdown = countdownSelected"
                      ></v-select>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>
              <v-card-actions>
                <div class="flex-grow-1"></div>
                <v-btn color="blue darken-1" text @click="saveSettings()">Close</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-row>
      </v-container>
    </v-content>
    <v-bottom-navigation
    color="deep-purple accent-4"
    >
    <v-btn @click="settings = true">
      <span>Settings</span>
      <v-icon>mdi-settings</v-icon>
    </v-btn>

  </v-bottom-navigation>
  </v-app>
</template>

<script>
import NoSleep from 'nosleep.js';

const noSleep = new NoSleep();
const beep = new Audio(require('./assets/beep.wav')); // eslint-disable-line global-require
const start = new Audio(require('./assets/start.mp3')); // eslint-disable-line global-require

export default {
  name: 'App',
  methods: {
    mouseClicked() {
      if (this.timerStarted) {
        this.timerStarted = false;
        this.stopTimer();
        noSleep.disable();
      } else {
        if (this.timer) {
          window.clearInterval(this.timer);
          this.timer = null;
        }

        this.startCountdown();
        noSleep.enable();
      }
    },
    startCountdown() {
      if (this.countdown > 0) {
        this.countdownStarted = true;
        this.countdownSeconds = this.countdown;
        this.updateCountdown();
        this.timer = setInterval(this.updateCountdown, 1000);
      } else {
        this.startTimer();
      }
    },
    startTimer() {
      this.timerStarted = true;
      this.updateTimer();
      this.timer = setInterval(this.updateTimer, 1000);
    },
    stopTimer() {
      window.clearInterval(this.timer);
      this.timer = null;
      this.totalSeconds -= 1;
    },
    updateCountdown() {
      this.timerDisplay = this.countdownSeconds;
      this.countdownSeconds -= 1;

      if (this.countdownSeconds < 0) {
        window.clearInterval(this.timer);
        this.timer = null;
        this.countdownStarted = false;
        this.timerStarted = true;
        if (this.soundOn) { start.play(); }
        this.startTimer();
      } else if (this.soundOn) {
        beep.play();
      }
    },
    updateTimer() {
      this.timerDisplay = `${this.getHours()}:${this.getMinutes()}:${this.getSeconds()}`;
      this.totalSeconds += 1;
    },
    getSeconds() {
      /*
        return seconds portion 00:00:ss
      */
      const sec = this.totalSeconds % 60;
      return sec >= 10 ? sec : `0${sec}`;
    },
    getMinutes() {
      /*
        return minutes portion 00:mm:00
      */
      const min = Math.floor((this.totalSeconds / 60) % 60);
      return min >= 10 ? min : `0${min}`;
    },
    getHours() {
      /*
        return hours portion hh:00:00
      */
      const hrs = Math.floor(this.totalSeconds / 60 / 60);
      return hrs >= 10 ? hrs : `0${hrs}`;
    },
    saveSettings() {
      const obj = { sound: '', countdown: '', displaySize: '' };
      obj.sound = this.soundOn;
      obj.countdown = this.countdown;
      obj.displaySize = this.displaySizeSelected;
      localStorage.pauseTimer = JSON.stringify(obj);
      this.settings = false;
    },
    loadSettings() {
      if (localStorage.pauseTimer) {
        const obj = JSON.parse(localStorage.pauseTimer);
        this.soundOn = obj.sound;
        this.countdownSelected = obj.countdown;
        this.countdown = obj.countdown;
        this.displaySizeSelected = obj.displaySize;
      }
    },
  },
  mounted() {
    const self = this;

    document.getElementById('mainContainer').addEventListener('click', () => {
      self.mouseClicked();
    });
    self.loadSettings();
  },
  data: () => ({
    timer: 0,
    totalSeconds: 0,
    timerDisplay: '00:00:00',
    timerStarted: false,
    countdown: 3,
    countdownSeconds: 0,
    coutdownStarted: false,
    settings: false,
    soundOn: false,
    mini: true,
    countdownSelected: 3,
    countdownItems: ['5', '4', '3', '2', '1', '0'],
    displaySizeSelected: 4,
    displaySizeItems: [
      { text: 'Large', value: 4 },
      { text: 'Medium', value: 3 },
      { text: 'Small', value: 2 },
    ],
  }),
};
</script>
