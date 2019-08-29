<template>
  <div class="timer">
    <p>{{ h }}小时{{ m }}分钟{{ s }}秒</p>
  </div>
</template>
<script>
import EventBus from "../eventbus";
export default {
  name: "timer",
  data() {
    return {
      h: "00",
      m: "00",
      s: "00",
      startTime: 0,
      endTime: 0,
      t: 0
    };
  },
  created() {
    EventBus.$on("timeReset", () => {
      this.showTime(0);
    });
    EventBus.$on("boom-end", () => {
      this.endTime = Date.now();
      this.showTime(this.endTime - this.startTime);
      clearInterval(this.t);
    });
    EventBus.$on("start-timer", () => {
      this.startTime = Date.now();
      this.t = setInterval(() => {
        let times = Date.now() - this.startTime;
        this.showTime(times);
      }, 500);
    });
  },
  methods: {
    showTime(times) {
      this.s = String(Math.trunc(times / 1000) % 60).padStart(2, "0"); //padStart()这个方法注意
      this.m = String(Math.trunc(times / (1000 * 60)) % 60).padStart(2, "0"); //padStart()这个方法注意
      this.h = String(Math.trunc(times / (1000 * 60 * 60)) % 60).padStart(
        2,
        "0"
      ); //padStart()这个方法注意
    }
  }
};
</script>
<style scoped lang="scss"></style>
