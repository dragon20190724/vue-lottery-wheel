<!-- 幸运大转盘 -->
<template>
  <div id="_big-wheel">
    <canvas
      class="_big-wheel-canvas"
      ref="wheelCanvas"
      width="360"
      height="360"
      :style="`transform: rotate(${canvasAngle + targetAngle}deg);transition-duration: ${transitionDuration}s;`"
    ></canvas>
    <img
      :src="goBtnImg"
      class="_big-wheel-go-img"
      @click="_onGo"
    >
  </div>
</template>

<script>
import { BtnImg } from './_base64.js'
export default {
  name: 'vueBigWheel',
  props:{
    //   开始按钮图片地址
      goBtnImg:{
        type:String,
        default:BtnImg
      },
    //   奖品列表
      prizeList:{
          type:Array,
          required:true
      },
    //   奖品区块对应填充色
      colors:{
          type:Array,
          default:["#F47F45", "#FFA468"]
      },
      //旋转动画时间单位S
      transitionDuration:{
          type:Number,
          default:8
      },
    //   奖品字体颜色
      fontColor:{
          type:String,
          default:'#7D2A00'
      },
    //   奖品文字大小
      fontSize:{
          type:String,
          default:'18px'
      },
      // 奖品文字的字体
      fontFamily: { 
          type: String,
          default: () => 'Helvetica Neue,Tahoma,Arial,PingFangSC-Regular,Hiragino Sans GB,Microsoft Yahei,sans-serif'
      },
      // 开始按钮自定义类名
      goClassName: { 
        type: String
    },
    // 奖品名称所对应的key
      strKey:{
          type:String,
          required:true
      },
      // 奖品文字总长度限制
      strMaxLength:{
          type:Number,
          default:0
      },
      // 奖品文字需要多行情况下第一行文字长度
      strLineLength:{
          type:Number,
          default:0
      }
  },
  data() {
    return {
      startAngle: 0, //开始角度
      outsideRadius: 180, //转盘外圆的半径
      insideRadius: 10, //转盘内圆的半径
      textRadius: 140, //转盘奖品位置距离圆心的距离
      isOnRotate: false, // 是否正在旋转，false 否
      stayIndex: 0, // 当前停留在那个奖品的序号
      targetAngle: 0, // 旋转到奖品目标需要的角度
    };
  },
  computed: {
    // 根据奖品列表计算canvas旋转角度，保持让启动按钮指针在奖品分区中间
    canvasAngle: function() {
      let _value = this.prizeList.length
      if(_value % 4 != 0) {
        return 0
      } else {
        let _a = _value / 4
        return _a % 2 == 0 ? 45/_a : 45
      }
    }
  },
  mounted() {
    this._drawWheelCanvas();
  },
  methods: {
    //   点击开始获取到奖品对应的key值
      _onGo(event){
          const _this = this
          if(_this.isOnRotate) return
          _this.isOnRotate = true
          _this.$emit('go-click',event)
      },
    //   计算旋转角度
      rotateFunc(targetIndex){
        let _angles
        //   获取奖品总数量
        let count = this.prizeList.length
        //    每个奖品的区间角度
        let _baseAngle = 360/count
        
        if(this.targetAngle == 0){
            // 第一次旋转角度 = 270度 - (停留的序号-目标序号)*每个奖品区间角度 - 每个奖品区间角度的一半 - canvas自身旋转的度数
            _angles = (360 * 3 / 4 - (targetIndex - this.stayIndex) * _baseAngle - _baseAngle / 2) - this.canvasAngle;
        }else{
            // 后续继续旋转 就只需要计算停留的位置与目标位置的角度
            _angles = -(targetIndex - this.stayIndex) * _baseAngle
        }
        this.stayIndex = targetIndex
        this.targetAngle += _angles + 360 * count // 转八圈 圈数越多，转的越快
        setTimeout(()=>{
            this.isOnRotate = false
            this.$emit('on-over')
        },this.transitionDuration * 1000 + 100)
      },
    // 渲染转盘
    _drawWheelCanvas() {
      // canvasAPI文档在线查看地址https://www.canvasapi.cn/CanvasRenderingContext2D/
      const _this = this;
      const canvas = _this.$refs.wheelCanvas;
      const ctx = canvas.getContext("2d");
      //根据奖品个数计算圆周角度
      let arc = Math.PI / (_this.prizeList.length / 2);
      let canvasW = canvas.width; // 画板的高度
      let canvasH = canvas.height; // 画板的宽度
      //在给定矩形内清空一个矩形
      ctx.clearRect(0, 0, canvasW, canvasH);
      //strokeStyle 绘制颜色
      ctx.strokeStyle = "#FFBE04"; // 设置描边颜色
      //font 画布上文本内容的当前字体属性
      // 整个画板会跟着放大缩小 字和图形都跟着被放大缩小  所以不需要rem单位
      // let _fontSize = _this.fontSize
      // if(_fontSize.includes('rem')) {
      //   // debugger
      //   // 获取html元素字体大小 得知1rem等于多少px
      //   let _rootSize = getComputedStyle(document.querySelector('html'))['font-size']
      //   _rootSize = _rootSize.slice(0, _rootSize.indexOf('px'))
      //   let _size = _fontSize.slice(0, _fontSize.indexOf('rem'))

      //   _fontSize = Number(_size) * Number(_rootSize) + 'px'
      // }
      ctx.font = `${_this.fontSize} ${_this.fontFamily}`;
      // 注意，开始画的位置是从0°角的位置开始画的。也就是水平向右的方向。
      // 画具体内容
      let length = _this.prizeList.length;
      for (let i = 0; i < length; i++) {
        // 当前的角度
        let angle = _this.startAngle + i * arc;
        // 每个奖品区块背景填充颜色
        ctx.fillStyle = _this.colors[i % 2];

        // 开始画内容
        ctx.beginPath();
        /*
         * 画圆弧，和IOS的Quartz2D类似
         * context.arc(x,y,r,sAngle,eAngle,counterclockwise);
         * x :圆的中心点x
         * y :圆的中心点x
         * sAngle,eAngle :起始角度、结束角度
         * counterclockwise : 绘制方向,可选，False = 顺时针，true = 逆时针
         * */
        ctx.arc(
          canvasW * 0.5,
          canvasH * 0.5,
          _this.outsideRadius,
          angle,
          angle + arc,
          false
        );
        ctx.arc(
          canvasW * 0.5,
          canvasH * 0.5,
          _this.insideRadius,
          angle + arc,
          angle,
          true
        );
        ctx.stroke();
        ctx.fill();
        //保存画布的状态，和图形上下文栈类似，后面可以Restore还原状态（坐标还原为当前的0，0），
        ctx.save();

        /*----绘制奖品内容*/
        
        ctx.fillStyle = _this.fontColor;
        let rewardName = _this._limit(_this.prizeList[i][_this.strKey], _this.strMaxLength);
        let line_height = 17;

        // translate方法重新映射画布上的 (0,0) 位置
        let translateX =
          canvasW * 0.5 + Math.cos(angle + arc / 2) * _this.textRadius;
        let translateY =
          canvasH * 0.5 + Math.sin(angle + arc / 2) * _this.textRadius;
        ctx.translate(translateX, translateY);

        // rotate方法旋转当前的绘图，因为文字适合当前扇形中心线垂直的！
        // angle，当前扇形自身旋转的角度 +  arc / 2 中心线多旋转的角度  + 垂直的角度90°
        ctx.rotate(angle + arc / 2 + Math.PI / 2);
        

        if(rewardName.length > _this.strLineLength && _this.strLineLength != 0) {
          rewardName = rewardName.substring(0, _this.strLineLength) + "||" + rewardName.substring(_this.strLineLength);
          let rewardNames = rewardName.split("||");
          for (let j = 0; j < rewardNames.length; j++) {
            ctx.fillText(
              rewardNames[j],
              -ctx.measureText(rewardNames[j]).width / 2,
              j * line_height
            );
          }
        } else {
          ctx.fillText(rewardName, -ctx.measureText(rewardName).width / 2, 0);
        }
        //还原画板的状态到上一个save()状态之前
        ctx.restore();

        /*----绘制奖品结束----*/
      }
    },
    /**
     * 限定字数溢出...
     * value 字符串
     * maxLength 限制长度 包括...
     * 如：'幸运大转盘',限制3个长度，返回就是'幸运...'
     */
    _limit(value,maxLength = 0) {
      if(!value || maxLength == 0) return value
      return value.length > maxLength ? value.slice(0,maxLength-1) + '...' : value
    },
  }
};
</script>
<style  lang='scss' scoped>
#_big-wheel {
  position: relative;
  width: 100%;
  height: 100%;
  ._big-wheel-canvas {
    width: 100%;
    transition: transform cubic-bezier(0.11,0.77,0.2,0.94); // 转盘旋转
  }
  ._big-wheel-go-img {
    z-index: 9999999999999999999;
    width: 100px;
    transform: translateX(-50%);
    position: absolute;
    left: 51%;
    top: 25%;
  }
}
</style>
