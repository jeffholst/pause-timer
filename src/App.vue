<template>
  <v-app>

  <!-- Sizes your content based upon application components -->
    <v-content id='mainContainer'>

      <!-- Provides the application the proper gutter -->
      <v-container fluid>

        <p class="text-center font-weight-black"
          v-bind:class="[{ 'red--text': !timerStarted},
            {'display-4': displaySizeSelected == '3'},
            {'display-3': displaySizeSelected == '2'},
            {'display-2': displaySizeSelected == '1'},
            ]"
        >
          {{timerDisplay}}
        </p>

        <p class="text-center grey--text lighten-2--text"
        >
          Elapsed Time: {{elapsedDisplay}}
        </p>

        <v-row class="text-center">
          <v-col cols="12">
            <ol class="splits" id="splits">
              <li v-for="split in splits" v-bind:key="split.id">
                {{ split.id}} - {{ split.time }}
              </li>
            </ol>
          </v-col>
        </v-row>

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
                      >
                        <v-tooltip slot="prepend" bottom>
                          <template v-slot:activator="{ on }">
                            <v-icon v-on="on">mdi-alarm-note</v-icon>
                          </template>
                          <span>Sound</span>
                        </v-tooltip>
                      </v-switch>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col cols="12">
                      <v-slider
                        ticks
                        v-model="countdown"
                        thumb-label="always"
                        min=0
                        max=10
                        step="1"
                      >
                         <v-tooltip slot="prepend" bottom>
                          <template v-slot:activator="{ on }">
                            <v-icon v-on="on">mdi-timer-off</v-icon>
                          </template>
                          <span>Countdown</span>
                        </v-tooltip>
                        <v-tooltip slot="append" bottom>
                          <template v-slot:activator="{ on }">
                            <v-icon v-on="on">mdi-timer</v-icon>
                          </template>
                          <span>Countdown</span>
                        </v-tooltip>
                      </v-slider>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col cols="12">
                      <v-slider
                        ticks
                        v-model="displaySizeSelected"
                        min=1
                        max=3
                        step="1"
                      >
                        <v-tooltip slot="prepend" bottom>
                          <template v-slot:activator="{ on }">
                            <v-icon v-on="on">mdi-format-font-size-decrease</v-icon>
                          </template>
                          <span>Display</span>
                        </v-tooltip>
                        <v-tooltip slot="append" bottom>
                          <template v-slot:activator="{ on }">
                            <v-icon v-on="on">mdi-format-font-size-increase</v-icon>
                          </template>
                          <span>Display</span>
                        </v-tooltip>
                      </v-slider>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col cols="12">
                     <v-dialog
                        ref="dialog"
                        v-model="showDurationDialog"
                        :return-value.sync="duration"
                        persistent
                        full-width
                        width="290px"
                      >
                        <template v-slot:activator="{ on }">
                          <v-text-field
                            v-model="duration"
                            label="Duration"
                            prepend-icon="mdi-alarm"
                            readonly
                            v-on="on"
                          ></v-text-field>
                        </template>
                        <v-time-picker
                          v-if="showDurationDialog"
                          v-model="duration"
                          format="24hr"
                          :allowed-hours="[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]"
                          full-width
                          use-seconds
                        >
                          <div class="flex-grow-1"></div>
                          <v-btn text color="primary"
                            @click="showDurationDialog = false">
                            Cancel
                          </v-btn>
                          <v-btn text color="primary"
                            @click="$refs.dialog.save(duration)">
                            OK
                          </v-btn>
                        </v-time-picker>
                      </v-dialog>
                    </v-col>
                  </v-row>
                   <v-row>
                    <v-col cols="12">
                      <v-switch
                        v-model="countDownTimer"
                      >
                        <v-tooltip slot="prepend" bottom>
                          <template v-slot:activator="{ on }">
                            <v-icon v-on="on">mdi-clock-outline</v-icon>
                          </template>
                          <span>Countdown Timer</span>
                        </v-tooltip>
                      </v-switch>
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
    <v-btn @click="subtractSecond();"
      v-show="!timerStarted"
    >
      <v-icon>mdi-minus</v-icon>
    </v-btn>
    <v-btn @click="openSettingsDialog();">
      <v-icon>mdi-settings</v-icon>
    </v-btn>
    <v-btn  @click="addSecond();"
      v-show="!timerStarted"
    >
      <v-icon>mdi-plus</v-icon>
    </v-btn>
  </v-bottom-navigation>
  </v-app>
</template>

<script>
import NoSleep from 'nosleep.js';

const noSleep = new NoSleep();
const beep = new Audio(require('./assets/beep.wav')); // eslint-disable-line global-require
const start = new Audio(require('./assets/start.mp3')); // eslint-disable-line global-require
const alarm = new Audio(require('./assets/alarm.mp3')); // eslint-disable-line global-require

export default {
  name: 'App',
  methods: {
    mouseClicked() {
      if (this.timerStarted) {
        this.timerStarted = false;
        this.pauseTimer();
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
      if (this.timer) {
        window.clearInterval(this.timer);
        this.timer = null;
      }
      this.timer = setInterval(this.updateTimer, 1000);
    },
    pauseTimer() {
      this.totalSeconds -= 1;
      if (this.timer) {
        window.clearInterval(this.timer);
        this.timer = null;
      }
      this.timer = setInterval(this.updateElapsedTimer, 1000);

      const t = `${this.getHours(this.totalSeconds)}:${this.getMinutes(this.totalSeconds)}:${this.getSeconds(this.totalSeconds)}`;

      this.splits.push({ id: this.splits.length + 1, time: t });
    },
    updateCountdown() {
      this.timerDisplay = this.countdownSeconds;
      this.countdownSeconds -= 1;

      if (!this.firstCountdown) { this.updateElapsedTimer(); }

      if (this.countdownSeconds < 0) {
        window.clearInterval(this.timer);
        this.timer = null;
        this.countdownStarted = false;
        this.timerStarted = true;
        if (this.soundOn) { start.play(); }
        this.startTimer();
        this.firstCountdown = false;
      } else if (this.soundOn) {
        beep.play();
      }
    },
    updateElapsedTimer() {
      this.updateElapsedDisplay();
      this.elapsedSeconds += 1;
    },
    updateElapsedDisplay() {
      this.elapsedDisplay = `${this.getHours(this.elapsedSeconds)}:${this.getMinutes(this.elapsedSeconds)}:${this.getSeconds(this.elapsedSeconds)}`;
    },
    updateTimer() {
      this.timerDisplay = `${this.getHours(this.totalSeconds)}:${this.getMinutes(this.totalSeconds)}:${this.getSeconds(this.totalSeconds)}`;

      if (!this.countDownTimer) {
        if (this.duration !== '00:00:00' && this.duration === this.timerDisplay) {
          alarm.play();
          this.pauseTimer();
        } else {
          this.totalSeconds += 1;
        }
      } else {
        if (this.totalSeconds === 0) { // eslint-disable-line no-lonely-if
          alarm.play();
          this.pauseTimer();
        } else {
          this.totalSeconds -= 1;
        }
      }

      this.updateElapsedTimer();
    },
    getSeconds(seconds) {
      /*
        return seconds portion 00:00:ss
      */
      const sec = seconds % 60;
      return sec >= 10 ? sec : `0${sec}`;
    },
    getMinutes(seconds) {
      /*
        return minutes portion 00:mm:00
      */
      const min = Math.floor((seconds / 60) % 60);
      return min >= 10 ? min : `0${min}`;
    },
    getHours(seconds) {
      /*
        return hours portion hh:00:00
      */
      const hrs = Math.floor(seconds / 60 / 60);
      return hrs >= 10 ? hrs : `0${hrs}`;
    },
    openSettingsDialog() {
      this.timerStarted = false;
      this.firstCountdown = true;
      this.elapsedTimer = '00:00:00';
      this.elapsedSeconds = 0;
      this.updateElapsedDisplay();
      this.splits = [];
      if (this.timer) {
        window.clearInterval(this.timer);
        this.timer = null;
      }
      this.settings = true;
    },
    saveSettings() {
      const obj = {
        sound: '',
        countdown: '',
        displaySize: '',
        duration: '',
        countDownTimer: '',
      };
      obj.sound = this.soundOn;
      obj.countdown = this.countdown;
      obj.displaySize = this.displaySizeSelected;
      obj.duration = this.duration;
      obj.countDownTimer = this.countDownTimer;
      localStorage.pauseTimer = JSON.stringify(obj);
      if (this.countDownTimer) {
        this.totalSeconds = 45; // JKH TODO
        this.timerDisplay = this.duration;
      } else {
        this.totalSeconds = 0;
        this.timerDisplay = '00:00:00';
      }
      this.settings = false;
    },
    loadSettings() {
      if (localStorage.pauseTimer) {
        const obj = JSON.parse(localStorage.pauseTimer);
        this.soundOn = obj.sound;
        this.countdown = obj.countdown;
        this.displaySizeSelected = obj.displaySize;
        this.duration = obj.duration;
        this.countDownTimer = obj.countDownTimer;
      }
    },
    adjustLastSplit(tDisplay) {
      this.splits[this.splits.length - 1].time = tDisplay;
    },
    subtractSecond() {
      if (this.totalSeconds > 0) {
        this.totalSeconds += -1;
        this.timerDisplay = `${this.getHours(this.totalSeconds)}:${this.getMinutes(this.totalSeconds)}:${this.getSeconds(this.totalSeconds)}`;
        this.adjustLastSplit(this.timerDisplay);
      }
    },
    addSecond() {
      this.totalSeconds += 1;
      this.timerDisplay = `${this.getHours(this.totalSeconds)}:${this.getMinutes(this.totalSeconds)}:${this.getSeconds(this.totalSeconds)}`;
      this.adjustLastSplit(this.timerDisplay);
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
    splits: [],
    timer: 0,
    totalSeconds: 0,
    elapsedSeconds: 0,
    firstCountdown: true,
    timerDisplay: '00:00:00',
    elapsedDisplay: '00:00:00',
    timerStarted: false,
    countdown: 3,
    countdownSeconds: 0,
    coutdownStarted: false,
    settings: false,
    soundOn: false,
    mini: true,
    displaySizeSelected: 3,
    duration: '00:00:00',
    showDurationDialog: false,
    countDownTimer: false,
  }),
};
</script>

<style scoped>
.splits {
  list-style-type: none
}
</style>
