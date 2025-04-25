<template>
  <div class="weather" v-if="weatherData.weather && weatherData.weather.city && weatherData.weather.weather">
    <span>{{ weatherData.weather.city }}&nbsp;</span>
    <span>{{ weatherData.weather.weather }}&nbsp;</span>
    <span>{{ weatherData.weather.temperature }}℃</span>
  </div>
  <div class="weather" v-else>
    <span>天气数据获取失败</span>
  </div>
</template>

<script setup>
import { getAdcode, getWeather, getOtherWeather } from "@/api";
import { Error } from "@icon-park/vue-next";

// 高德开发者 Key
const mainKey = import.meta.env.VITE_WEATHER_KEY;

// 天气数据
const weatherData = reactive({
  weather: {
    city: null,
    weather: null, // 天气现象
    temperature: null, // 实时气温
  },
});

// 取出天气平均值
const getTemperature = (min, max) => {
  try {
    // 计算平均值并四舍五入
    const average = (Number(min) + Number(max)) / 2;
    return Math.round(average);
  } catch (error) {
    console.error("计算温度出现错误：", error);
    return "NaN";
  }
};

// 获取天气数据
const getWeatherData = async () => {
  try {
    // 获取地理位置信息
    if (!mainKey) {
      console.log("未配置，使用备用天气接口");
      const result = await getOtherWeather();
      console.log(result);
      const data = result.data;
      weatherData.weather = {
        city: data.location.city || "未知地区",
        weather: data.now.text || "未知天气",
        temperature: data.now.temperature || "未知温度",
      };
    } else {
      // 获取 Adcode
      const adCode = await getAdcode(mainKey);
      console.log(adCode);
      if (adCode.infocode !== "10000") {
        throw "地区查询失败";
      }
      weatherData.weather = {
        city: adCode.city || "未知地区",
        weather: "未知天气", // 默认值
        temperature: "未知温度", // 默认值
      };
      // 获取天气信息
      const result = await getWeather(mainKey, adCode.adcode);
      // 检查 result.lives 是否存在且不为空
      if (result.lives && result.lives.length > 0) {
        weatherData.weather = {
          city: result.lives[0].city || "未知地区",
          weather: result.lives[0].weather || "未知天气",
          temperature: result.lives[0].temperature || "未知温度",
        };
      } else {
        console.warn("API 返回的 lives 数据为空");
        weatherData.weather = {
          city: "未知地区",
          weather: "未知天气",
          temperature: "未知温度",
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