<template>
  <div class="playfailSound">
    <audio ref="sound" :src="srcSound"></audio>
  </div>
</template>
<script>
import EventBus from "../eventbus";
export default {
  name: "playSound",
  data() {
    return {};
  },
  props: {
    srcSound: {
      type: String,
      required: true
    }
  },
  created() {
    //$on 代表是监听事件。 对应于$emit()这个发送事件的方法
    // EventBus.$on("click-cell", () => {
    //   this.$refs.sound.play();
    // });
    EventBus.$on("boom-end", () => {
      this.$refs.sound.play();
    });
  }
};
</script>
<style scoped lang="scss">
.playSound {
  height: 0;
  width: 0;
  visibility: hidden;
  position: absolute;
}
</style>
