<template>
  <!-- 
			<card>
			城市信息 左上角 city country
			背景图 中间
			当前日期 右上角 week uptime 
				<div leftbottome>
				实时温度 左下
				天气 wea wea_img tem 
				最高温 最低温 tem1 2
				空气质量 air 
				</div>
			</card>
			<card>
				七日概况
				使用图表显示
				<tab>
					七日温度 天气状况 wea
				</tab>
				<tab>
					空气质量 air air-tips level
				</tab>
				<tab>
					七日风况 win win-speed meter
				</tab>
			</card>
			杂项
			<card>
				<tab>
					空气质量详细信息 air air-tips level humidity visibility
				</tab>
				<tab>
					气压 pressure
				</tab>
				<tab>
					风速等信息 win win-speed meter
				</tab>
			</card>
		 -->
  <view class="back">
    
    <!-- <uni-card>
      <uni-search-bar @confirm="search" @input="input" placeholder="请输入城市名,快速查询天气"></uni-search-bar>
    </uni-card> -->

    <!-- 实现搜索框 -->
    
       <!-- 内联样式以覆盖uni-card的背景色 -->
    <uni-card class="today " style="background-color:transparent">
      <view class="top">
        <!-- 位置信息 -->
        <view id="local">{{ weather.city }}</view>
        <!-- 天气状况 -->
        <view>
          <img :src="'../../static/icon/' + today.wea_img + '.png'" />
        </view>
      </view>
      <!-- 天气 -->
      <view class="wea">
        <view id="tem">{{ today.tem }}</view>
        <view id="todayWea">{{ today.wea }}</view>
        <view id="todayTime">刷新时间{{ weather.update_time }}</view>
      </view>
    </uni-card>

    <uni-card class="air">
      <view>
        <view class="title">空气质量</view>
        <!-- 圆弧进度图 -->
        <view class="qiun-charts3">
          <canvas
            canvas-id="canvasArcbar1"
            id="canvasArcbar1"
            class="charts3"
          ></canvas>
        </view>

        <view>tips:{{ today.air_tips }}</view>
      </view>
    </uni-card>

    <uni-card>
      <view class="title ">详情数据</view>
      <!-- 风 -->
      <uni-card>
        <view class="iconfont indexTitle">&#xe6f7;</view>

        <view>风向:{{ today.win }}</view>
        <view class="iconfont indexTitle">&#xe608;</view>
        <view>风力:{{ today.win_speed }}</view>
      </uni-card>

      <!-- 温度 -->
      <uni-card>
        <view class="iconfont indexTitle">&#xe61a;</view>
        <view>最高温度：{{ today.tem1 }}</view>
        <view>最低温度：{{ today.tem2 }}</view>
      </uni-card>
    </uni-card>
    <!-- 生活指数 -->
    <uni-card>
      <view class="title title">生活指数</view>
      <uni-card>
        <view class="indexTitle iconfont">&#xe611;中暑指数</view>
        <view>{{ indexs[0].level }}</view>
        <view>{{ indexs[0].desc }}</view>
      </uni-card>
      <uni-card>
        <view class="indexTitle iconfont">&#xea73;健身指数</view>
        <!-- <view>{{ indexs[1].level }}</view> -->
        <view>{{ indexs[1].desc }}</view>
      </uni-card>
      <uni-card>
        <view class="indexTitle iconfont">&#xe64c;血糖指数</view>
        <view>{{ indexs[2].level }}</view>
        <view>{{ indexs[2].desc }}</view>
      </uni-card>
      <uni-card>
        <view class="indexTitle iconfont">&#xe66b;穿衣指数</view>
        <view>{{ indexs[3].level }}</view>
        <view>{{ indexs[3].desc }}</view>
      </uni-card>
      <uni-card>
        <view class="indexTitle iconfont">&#xe607;洗车指数</view>
        <view>{{ indexs[4].level }}</view>
        <view>{{ indexs[4].desc }}</view>
      </uni-card>
      <uni-card>
        <view class="indexTitle iconfont">&#xe528;紫外线指数</view>
        <view>{{ indexs[5].level }}</view>
        <view>{{ indexs[5].desc }}</view>
      </uni-card>
    </uni-card>
    <uin-card> </uin-card>
  </view>
</template>

<script>
import { uniCard } from "@dcloudio/uni-ui";
import uniSearchBar from "../../components/search-bar/uni-search-bar/uni-search-bar";
import uCharts from "../../components/u-charts/u-charts.js";

// 圆弧进度图变量
var _self;
var canvaArcbar1;

export default {
  data() {
    return {
      

      weather: {},
      daysWeather: [],
      today: {
        day: "",
        week: "",
        wea: "",
        wea_img: "",
        tem: "",
        tem1: "",
        tem2: "",
        hours: [], //两个数组另行渲染
        humidity: "",
        air_level: "",
        air_tips: "",
        air: {
          series: [
            {
              name: "空气质量",
              data: "0",
              color: "#2fc25b",
            },
          ],
        },

        win: "",
        win_speed: "",
      },
      hours: [],
      indexs: [],

      cWidth3: "", //圆弧进度图
      cHeight3: "", //圆弧进度图
      arcbarWidth: "", //圆弧进度图，进度条宽度,此设置可使各端宽度一致
      pixelRatio: 1,
    };
  },
  components: {
    uniCard,
    uCharts,
    uniSearchBar,
  },
  onLoad() {
    this.getWeather();

    // 圆弧进度图
    _self = this;
    this.cWidth3 = uni.upx2px(250); //这里要与样式的宽高对应
    this.cHeight3 = uni.upx2px(250); //这里要与样式的宽高对应
    this.arcbarWidth = uni.upx2px(24);
  },
  methods: {
    getWeather() {
      uni.request({
        url:
          "/api?version=v1&vue=1&appid=47884695&appsecret=pnG7TJVB&city=遂平", //暂时写死位置信息
        data: {},
        header: {
          "custom-header": "weather", //自定义请求头信息
        },
        success: (res) => {
          this.weather = res.data;
          this.daysWeather = this.weather.data;

          // 每日天气
          this.today.day = this.daysWeather[0].day;
          this.today.week = this.daysWeather[0].week;
          this.today.wea = this.daysWeather[0].wea;
          this.today.wea_img = this.daysWeather[0].wea_img;
          this.today.tem = this.daysWeather[0].tem;
          this.today.tem1 = this.daysWeather[0].tem1;
          this.today.tem2 = this.daysWeather[0].tem2;
          this.hours = this.daysWeather[0].hours;
          this.today.humidity = this.daysWeather[0].humidity;
          this.today.air_level = this.daysWeather[0].air_level;
          this.today.air_tips = this.daysWeather[0].air_tips;
          // 获取的数据是整数,因此要先除以100算出百分比
          this.today.air.series[0].data = this.daysWeather[0].air / 100;
          this.today.air.series[0].name = this.today.air_level;

          this.indexs = this.daysWeather[0].index;
          this.today.win = this.daysWeather[0].win[0];
          this.today.win_speed = this.daysWeather[0].win_speed;

          // 传入圆弧进度图带数据
          _self.showArcbar("canvasArcbar1", this.today.air);

          console.log(this.today.air.series[0].data);
          console.log(res);
        },
      });
    },

    // 显示圆弧进度图的方法,直接复制的官方文档
    showArcbar(canvasId, chartData) {
      canvaArcbar1 = new uCharts({
        $this: _self,
        canvasId: canvasId,
        type: "arcbar",
        fontSize: 11,
        legend: { show: false },
        background: "#FFFFFF",
        pixelRatio: _self.pixelRatio,
        series: chartData.series,
        animation: true,
        width: _self.cWidth3 * _self.pixelRatio,
        height: _self.cHeight3 * _self.pixelRatio,
        dataLabel: true,
        title: {
          name: Math.round(chartData.series[0].data * 100) + "%", //这里我自动计算了，您可以直接给任意字符串
          color: chartData.series[0].color,
          fontSize: 25 * _self.pixelRatio,
        },
        subtitle: {
          name: chartData.series[0].name, //这里您可以直接给任意字符串
          color: "#666666",
          fontSize: 15 * _self.pixelRatio,
        },
        extra: {
          arcbar: {
            type: "default",
            width: _self.arcbarWidth * _self.pixelRatio, //圆弧的宽度
          },
        },
      });
    },

    
  },
};
</script>

<style lang="scss" scoped>
// .uni-card {
//   background-color: transparent;
// }

.today {
  .top {
    display: flex;
    flex-direction: row;
    flex-wrap: nowrap;
    justify-content: space-between;
    view {
      img {
        width: 150rpx;
        height: 80rpx;
      }
    }
  }
  .wea {
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    #tem {
      height: 100rpx;
      width: 100rpx;
      font-size: 200%;
      font-style: blue;
    }
    #todayWea {
      font-size: 70%;
    }
    #todayTime {
      font-size: 50%;
    }
  }
  .uni-input {
    background-color: rgb(34, 27, 27);
  }
}

.title {
  font-size: 75%;
  color: rgb(36, 235, 218);
}
.indexTitle {
  font-size: 80%;
  color: rgb(36, 235, 218);
}

.qiun-charts3 {
  width: 250upx;
  height: 250upx;
  background-color: transparent;
  position: relative;
}

.charts3 {
  position: absolute;
  width: 250upx;
  height: 250upx;
  background-color: transparent;
}
</style>
