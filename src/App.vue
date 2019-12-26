<!-- demo测试专用的页面 -->
<template>
  <div id="demo">
    <div class="box">
      <vue-big-wheel
        ref="luckyWheel"
        :prizeList="listData"
        :colors="colors"
        fontColor = 'red'
        goClassName = 'btn'
        :strMaxLength = '14'
        :strLineLength = '6'
        strKey='name'
        @go-click="go"
        @on-over="onOver"
      >
      </vue-big-wheel>
    </div>
  </div>
</template>

<script>
import vueBigWheel from "@/components/vue-big-wheel/index";
export default {
  data() {
    return {
      listData: [
        {name: "iphone6", value: 1},
        {name: "小米X", value: 2},
        {name: "华为Mate30", value: 3},
        {name: "红米6A", value: 4},
        {name: "天猫精灵", value: 5},
        {name: "小米扫地机器人", value: 6},
        {name: "100元", value: 7},
        {name: "谢谢参与", value: 8},
      ],
      colors: ["#F47F45", "#FFA468"],
      targetIndex: 0
    };
  },
  mounted() {},
  methods: {
    onOver() {
      alert(this.listData[this.targetIndex].name)
    },
    go(event) {
      console.log("TCL: go -> event", event)
      // 模拟请求 200ms
      setTimeout(() => {
        // 随机
        let random = Math.random() * this.listData.length
        console.log(random,'random随机小数*物品数量')
        let _target = Math.floor(random)
        console.log(_target,'上取整,舍余数')
        this.targetIndex = _target
		console.log(this.targetIndex,this.listData[this.targetIndex].name,"奖品数组下标---奖品")
        this.$refs.luckyWheel.rotateFunc(this.targetIndex)
      }, 200);
    },

  },
  components: {
    vueBigWheel
  }
};
</script>
<style  lang='scss'>
#demo {
  width: 100vw;
  min-height: 100vh;
  background-image: url('./assets/img/wheel_bg.jpg');
  background-repeat: no-repeat;
  background-position: center top;
  background-size: 100%;
  overflow: hidden;
  .box {
    box-sizing: border-box;
    width: 3.42rem;
    height: 3.42rem;
    margin: 1.7rem auto 0;
    padding: .22rem;
    background-image: url('./assets/img/disk_bg.png');
    background-repeat: no-repeat;
    background-size: 100%;
    background-position: center;
    overflow: hidden;
    .btn { // 使用自定义类 不能用scope
      width: 1.12rem;
    }
  }
}
</style>
