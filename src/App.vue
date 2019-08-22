<template>
  <v-app>
    <p class="text-center display-4"
      v-bind:class="{'red--text': !timerStarted}"
    >
      {{timerDisplay}}
    </p>
  </v-app>
</template>

<script>
import NoSleep from 'nosleep.js';

const noSleep = new NoSleep();

const audioElement = document.createElement('audio');
audioElement.setAttribute('src', 'assets/');

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
        this.startTimer();
      }
    },
    updateTimer() {
      this.timerDisplay = `${this.GetHours()}:${this.GetMinutes()}:${this.GetSeconds()}`;
      this.totalSeconds += 1;
    },
    GetSeconds() {
      /*
        return seconds portion 00:00:ss
      */
      const sec = this.totalSeconds % 60;
      return sec >= 10 ? sec : `0${sec}`;
    },
    GetMinutes() {
      /*
        return minutes portion 00:mm:00
      */
      const min = Math.floor((this.totalSeconds / 60) % 60);
      return min >= 10 ? min : `0${min}`;
    },
    GetHours() {
      /*
        return hours portion hh:00:00
      */
      const hrs = Math.floor(this.totalSeconds / 60 / 60);
      return hrs >= 10 ? hrs : `0${hrs}`;
    },
  },
  mounted() {
    document.body.addEventListener('click', () => { this.mouseClicked(); }, true);
  },
  data: () => ({
    timer: 0,
    totalSeconds: 0,
    timerDisplay: '00:00:00',
    timerStarted: false,
    countdown: 3,
    countdownSeconds: 0,
    coutdownStarted: false,
  }),
};
</script>
