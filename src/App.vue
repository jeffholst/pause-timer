<template>
  <v-app>
    <p class="text-center display-4" >{{timerDisplay}}</p>
  </v-app>
</template>

<script>

export default {
  name: 'App',
  methods: {
    mouseClicked() {
      if (this.timerStarted) {
        this.timerStarted = false;
        this.stopTimer();
      } else {
        this.timerStarted = true;
        this.startTimer();
      }
    },
    startTimer() {
      this.timer = setInterval(this.updateTimer, 1000);
    },
    stopTimer() {
      clearInterval(this.timer);
    },
    updateTimer() {
      this.totalSeconds += 1;
      this.timerDisplay = `${this.GetHours()}:${this.GetMinutes()}:${this.GetSeconds()}`;
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
  created: function () {
    const self = this;
    document.body.addEventListener('click', () => { self.mouseClicked(); }, true);
  },
  data: () => ({
    timer: 0,
    totalSeconds: 0,
    timerDisplay: '00:00:00',
    timerStarted: false,
  }),
};
</script>
