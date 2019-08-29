<template>
  <td
    @mousedown="cellClick"
    :class="{
      'mine-isShowboom': cellData.isShowBoom,
      'mine-clear': cellData.isClear,
      'mine-marked': cellData.ismMark
    }"
    :style="{ width: size.w + 'px', height: size.h + 'px' }"
    class="cell"
  >
    <!-- <template v-if="cellData.isBoom">
      <div class="boomDiv"><img src="/boom.png" alt="炸弹" /></div>
    </template>
    <template v-else> -->
    <span
      :style="{ color: inlinestyle(cellData.data) }"
      v-if="cellData.data != 0 && cellData.isClear"
      >{{ cellData.data }}</span
    >
    <!-- </template> -->
  </td>
</template>
<script>
import EventBus from "../eventbus";
import { MessageBox } from "element-ui";
export default {
  name: "cell",
  data() {
    return {};
  },
  methods: {
    inlinestyle(num) {
      switch (num) {
        case 1:
          return "blue";
        case 2:
          return "#e36";
        case 3:
          return "#0a2";
        case 4:
          return "#c4d";
        case 5:
          return "white";
        case 6:
          return "brown";
        case 7:
          return "orange";
        case 8:
          return "orange";
        default:
          break;
      }
    },
    cellClick(e) {
      if (this.cellData.isClear || this.cellData.isShowBoom) {
        return; //误操作
      }

      if (e.button === 2) {
        //设置EventBus 点击了开始计时
        EventBus.$emit("click-cell");
        //右键点击 marked状态
        this.$set(this.cellData, "ismMark", !this.cellData.ismMark);
        return;
      }
      //鼠标左键点击
      if (e.button === 0) {
        if (this.cellData.isBoom) {
          //如果标记了小旗子，则左键无效
          if (this.cellData.ismMark) return;
          //如果是炸弹游戏结束
          EventBus.$emit("boom-end");
          MessageBox({
            title: "你输了",
            message: "你真的太菜了",
            type: "warning"
            // callback() {
            //   EventBus.$emit("gameover");
            // }
          });
        } else {
          //如果标记了小旗子，则左键无效
          if (this.cellData.ismMark) return;
          //设置EventBus 点击了开始计时
          EventBus.$emit("click-cell");
          //让当前的单元格显示数字,并且如果是空白的要把周围的空白都打开，这样就得在父容器中进行操作了。
          // this.$set(this.cellData, "isClear", true);//这只能打开当前的空格。
          this.$emit("clearboom", this.cellData.cellIndex);
        }
      }
    }
  },
  props: {
    cellData: {
      //他为什么可以接收到这个数值？
      type: Object,
      required: true
    },
    //props属性可以接收父级传来的值，因为是传给size的对象，所以要调取w和h，只能使用size.w和size.h来调用
    size: {
      type: Object,
      required: false,
      default: function() {
        return {
          w: 30,
          h: 30
        };
      }
    }
  }
};
</script>
<style scoped lang="scss">
td {
  border: 1px solid #c09;
  background-color: #fff;
  text-align: center;
  vertical-align: middle;
}
.mine-clear {
  background-color: #ccc;
}
.mine-isShowboom {
  background: url(/boom.png) no-repeat;
  background-size: contain;
}
.boomDiv {
  width: 100%;
  height: 100%;
  img {
    width: 100%;
    height: 100%;
  }
}
.mine-marked {
  position: relative;
  &::before {
    content: "";
    display: block;
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: url(/flag.png) no-repeat;
    background-size: contain;
  }
}
</style>
