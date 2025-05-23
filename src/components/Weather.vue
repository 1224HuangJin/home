<template>
  <div class="weather" v-if="weatherData.city && weatherData.data.type">
    <span>{{ weatherData.city }}&nbsp;</span>
    <span>{{ weatherData.data.type }}&nbsp;</span>
    <span>{{ weatherData.data.low }}</span>
    <span class="sm-hidden">
      &nbsp;{{ weatherData.data.fengxiang }}&nbsp;
    </span>
    <span class="sm-hidden">{{ weatherData.data.fengli }}</span>
  </div>
  <div class="weather" v-else>
    <span>天气数据获取失败</span>
  </div>
</template>

<script setup>
import { h } from "vue";
import { Error } from "@icon-park/vue-next";
import { ElMessage } from "element-plus";
import { getAdcode, getWeather, getOtherWeather } from "@/api";

// 高德开发者 Key
const mainKey = import.meta.env.VITE_WEATHER_KEY;

// 天气数据
const weatherData = reactive({
  city: null, // 城市
  data: {
    type: null, // 天气现象
    low: null, // 最低气温
    high: null, // 最高气温
    fengxiang: null, // 风向描述
    fengli: null, // 风力级别
  },
});

// 获取天气数据
const getWeatherData = async () => {
  try {
    if (mainKey) {
      // 使用高德地图 API
      const adCode = await getAdcode(mainKey);
      if (adCode.infocode !== "10000") {
        throw "地区查询失败";
      }
      weatherData.city = adCode.city;
      const result = await getWeather(mainKey, adCode.adcode);
      weatherData.data = {
        type: result.lives[0].weather,
        low: result.lives[0].temperature,
        high: result.lives[0].temperature,
        fengxiang: result.lives[0].winddirection,
        fengli: result.lives[0].windpower,
      };
    } else {
      try {
        // 使用自定义 API（韩小韩）
        console.log("未配置，使用备用天气接口");
        const result = await getOtherWeather();
        if (result.success) {
          weatherData.city = result.city;
          weatherData.data = {
            type: result.data.type,
            low: result.data.low,
            high: result.data.high,
            fengxiang: result.data.fengxiang,
            fengli: result.data.fengli,
          };
        } else {
          throw "天气数据获取失败";
        }
      } 
      catch (customError) {
        console.error(customError);
        const result = await getOtherWeather();
        const data = result.result;
        weatherData.city = data.city.City || "未知地区";
        weatherData.data = {
          type: data.condition.day_weather,
          low: data.condition.min_degree,
          high: data.condition.max_degree,
          fengxiang: data.condition.day_wind_direction,
          fengli: data.condition.day_wind_power,
        };
      }
    }
  } catch (error) {
    console.error("天气信息获取失败:" + error);
    onError("天气信息获取失败");
  }
};

// 报错信息
const onError = (message) => {
  ElMessage({
    message,
    icon: h(Error, {
      theme: "filled",
      fill: "#efefef",
    }),
  });
  console.error(message);
};

onMounted(() => {
  // 调用获取天气
  getWeatherData();
});
</script>