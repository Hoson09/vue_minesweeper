<template>
  <div id="app">
    <h1>HuYi's 扫雷游戏</h1>
    <div class="container">
      <div class="main">
        <table class="mine-table">
          <tr v-for="(rItem, rIndex) of rows" :key="'row_' + rIndex">
            <!-- <td
              :style="{ width: cellWith + 'px', height: cellHeight + 'px' }"
              v-for="(cItem, cIndex) of cols"
              :key="'cols_' + cIndex"
            ></td> -->
            <cell
              @clearboom="clearboom"
              :cellData="cellArray[rIndex * cols + cIndex]"
              :size="{ w: cellWith, h: cellHeight }"
              v-for="(cItem, cIndex) of cols"
              :key="'cols_' + cIndex"
            ></cell>
          </tr>
        </table>
      </div>
      <div class="aside">
        <div class="top-select">
          <el-radio-group @change="changeLevel" v-model="level" size="mini">
            <el-radio-button label="简单"></el-radio-button>
            <el-radio-button label="中级"></el-radio-button>
            <el-radio-button label="高级"></el-radio-button>
            <el-radio-button label="S级"></el-radio-button>
          </el-radio-group>
        </div>
        <div class="block">
          <span class="demonstration">列数{{ cols }}</span>
          <el-slider v-model="cols" :min="10" :max="25"></el-slider>
        </div>
        <div class="block">
          <span class="demonstration">行数{{ rows }}</span>
          <el-slider v-model="rows" :min="10" :max="30"></el-slider>
        </div>
        <div class="block">
          <el-switch
            class="switch"
            style="display: block"
            v-model="isCellWHSync"
            active-color="#13ce66"
            inactive-color="#ff4949"
            active-text="是否同步宽高"
          >
          </el-switch>
          <span class="demonstration">行高:{{ cellHeight }}</span>
          <el-slider v-model="cellHeight" :min="20"></el-slider>
          <span class="demonstration">列高:{{ cellWith }}</span>
          <el-slider v-model="cellWith" :min="20"></el-slider>
        </div>
        <div class="block">
          <el-button @click="initCellData" type="primary" plain>重置</el-button>
        </div>
        <div class="block">
          <timer></timer>
        </div>
      </div>
    </div>
    <play-sound src-sound="/success.mp3"></play-sound>
    <play-failsound src-sound="/fail.mp3"></play-failsound>
  </div>
</template>

<script>
import "element-ui/lib/theme-chalk/index.css";
import Cell from "./components/Cell";
import PlaySound from "./components/PlaySound";
import Timer from "./components/Timer";
import EventBus from "./eventbus";
import PlayFailSound from "./components/PlayFailSound";
export default {
  name: "app",
  data() {
    return {
      level: "简单", // 简单 ：1 中级 ：2 高级：3 S级：4  难度系数0.15
      cols: 10,
      rows: 10,
      cellWith: 30,
      cellHeight: 30,
      isCellWHSync: true,
      // cell:{isBoom:false,data:random,isMarked:false,isClear:false}//一个单元格的数据形式就是这样了。
      cellArray: [],
      //是否重置
      isReset: true
      // sound: ""
    };
  },
  created() {
    this.initCellData();
    document.oncontextmenu = () => {
      // return false; //禁止鼠标右键弹出菜单的默认操作。
    };
    EventBus.$on("click-cell", () => {
      this.isReset && EventBus.$emit("start-timer");
      this.isReset = false;
      // this.sound = "/success.mp3";
    });
    EventBus.$on("boom-end", () => {
      // this.sound = "/fail.mp3";
      this.boomend();
    });
    // EventBus.$on("gameover", () => {
    //   this.initCellData();
    // });
  },
  methods: {
    boomend() {
      //把所有的图片都显示出来
      this.cellArray.forEach(value => {
        if (value.isBoom) {
          this.$set(value, "isShowBoom", true);
        } else {
          this.$set(value, "isClear", true);
        }
      });
    },
    clearboom(index) {
      const clearBoomInnerFun = cIndex => {
        //构建函数的一个内部函数,进行对扫雷的清理工作。
        if (cIndex >= 0 && cIndex < this.cellArray.length) {
          let cell = this.cellArray[cIndex];
          if (cell.isBoom || cell.ismMark || cell.isClear) {
            return;
          }
          this.$set(this.cellArray[cIndex], "isClear", true);
          //如果自己本身data也是0,需要继续清理,(这里是递归调用。)
          this.clearboom(cIndex); //再次调用函数进行验证自己的data是否是0，如果是0继续这一步操作，直到找data不为0的那个cell，让那个cell自己变颜色即可，递归结束再回调。
        }
      };
      //清除data为0的非雷空格和周边的非雷空格(需要递归)。如果自己非0，只清理自己即可
      let cell = this.cellArray[index];
      if (cell.data !== 0) {
        //如果自己非0，只清理自己即可
        this.$set(cell, "isClear", true); //给对象设置属性也可以使用$set方法。
        return;
      }
      //如果是0 周围八个都要清掉
      if (index % this.cols == 0) {
        for (let i = -1; i <= 1; i++) {
          let startNum = index - i * this.cols;
          clearBoomInnerFun(startNum);
          clearBoomInnerFun(startNum + 1);
        }
      } else if (index % this.cols == this.cols - 1) {
        for (let i = -1; i <= 1; i++) {
          let startNum = index - i * this.cols - 1;
          clearBoomInnerFun(startNum);
          clearBoomInnerFun(startNum + 1);
        }
      } else {
        for (let i = -1; i <= 1; i++) {
          let startNum = index - i * this.cols - 1;
          clearBoomInnerFun(startNum);
          clearBoomInnerFun(startNum + 1);
          clearBoomInnerFun(startNum + 2);
        }
      }
    },
    changeLevel() {
      this.initCellData();
    },
    initCellData() {
      this.isReset = true;
      EventBus.$emit("timeReset");
      //动态初始化扫雷的网格数据。cell数组，随机生成地雷数
      //1.计算需要多少个网格数以及总共要生成多少个地雷
      let sum = this.cols * this.rows;
      let randomBoom = sum * this.getLevelNum() * 0.15;
      let randomIndexSet = new Set(); //集合的元素不能重复
      //2.随机给这些地雷设定在网格中的位置索引，并把这些索引添加到Set集合中。
      while (randomIndexSet.size < randomBoom) {
        randomIndexSet.add(Math.trunc(Math.random() * sum));
      }
      // console.log(randomIndexSet); //验证集合Set是否相同
      //3.在Set集合中的位置索引上放置地雷。
      this.cellArray = []; //每次调用都要进行清空数组，才能保证每次调用都是一个全新的数组。(必须)
      for (let i = 0; i < sum; i++) {
        let isBoom = randomIndexSet.has(i);
        //4.设置一个函数方法getBoomNum()计算每一个cell周围的炸弹数量,和自己这个cell上该显示的数字。
        let data = this.getBoomNum(i, randomIndexSet);
        this.cellArray.push({
          isBoom,
          data,
          isShowBoom: false,
          ismMark: false,
          isClear: false,
          cellIndex: i
        }); //这样设置数组，会被Vue对象监控。这种事简便方式
        // this.cellArray[i] = {}; //这样给数组设置方式是不会被监控的
        // this.$set(this.cellArray, i, {}); //这种是最正规的可以被监控的方式。
      }
    },
    getBoomNum(index, boomsSet) {
      let count = 0; //周围总共的炸弹数。
      if (index % this.cols == 0) {
        for (let i = -1; i <= 1; i++) {
          let startNum = index - i * this.cols;
          count += boomsSet.has(startNum);
          count += boomsSet.has(startNum + 1);
        }
      } else if (index % this.cols == this.cols - 1) {
        for (let i = -1; i <= 1; i++) {
          let startNum = index - i * this.cols - 1;
          count += boomsSet.has(startNum);
          count += boomsSet.has(startNum + 1);
        }
      } else {
        for (let i = -1; i <= 1; i++) {
          let startNum = index - i * this.cols - 1;
          count += boomsSet.has(startNum);
          count += boomsSet.has(startNum + 1);
          count += boomsSet.has(startNum + 2);
        }
      }
      return count;
    },
    getLevelNum() {
      if (this.level === "简单") {
        return 1;
      } else if (this.level === "中级") {
        return 2;
      } else if (this.level === "高级") {
        return 3;
      } else {
        return 4;
      }
    }
  },
  watch: {
    /*方法 watch，它可以用来监测Vue实例上的数据变动并赋值。
    如果watch方法对应一个对象，键是观察表达式，值是对应回调，值也可以是方法名，或者是对象，包含选项。
    如果watch监测的是一个对象的话，直接使用watch是不行的，此时我们可以借助于computed计算属性来完成。*/
    cellWith(val) {
      this.isCellWHSync && (this.cellHeight = val);
    },
    cellHeight(val) {
      this.isCellWHSync && (this.cellWith = val);
    },
    rows() {
      this.initCellData();
    },
    cols() {
      this.initCellData();
    }
  },
  components: {
    cell: Cell,
    "play-sound": PlaySound,
    timer: Timer,
    "play-failsound": PlayFailSound
  }
};
</script>

<style scoped lang="scss">
#app {
  .block {
    border-top: 1px solid #ccc;
    padding: 5px 20px;
    margin-top: 5px;
    .switch {
      margin: 5px auto 10px;
    }
  }
  text-align: center;
  .container {
    display: flex;
    & > div {
      border: 1px solid #ccc;
    }
    .main {
      flex: 5 0 500px;
      display: flex;
      flex-direction: row;
      justify-content: space-around;
      align-items: center;
      border-radius: 10px 0 0 10px;
      .mine-table {
        border-collapse: collapse;
      }
    }
    .aside {
      flex: 2 0 200px;
      padding-top: 5px;
      border-radius: 0 10px 10px 0;
    }
  }
}
</style>
