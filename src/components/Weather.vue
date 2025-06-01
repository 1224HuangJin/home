<template>
  <div class="weather" v-if="weatherData.city && weatherData.data.type">
    <span>{{ weatherData.city }}&nbsp;</span>
    <span>{{ weatherData.data.type }}&nbsp;</span>
    <span>{{ weatherData.data.low }}°C</span>
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
import { getAdcode, getWeather, getOtherWeather, getVmyWeather } from "@/api";

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
      throw "未配置高德Key";
    }
  } catch (error) {
    try {
      // 使用韩小韩 API 作为第一个兜底
      console.log("尝试使用韩小韩天气接口");
      const result = await getOtherWeather();
      if (result.success) {
        weatherData.city = result.city;
        weatherData.data = {
          type: result.data.type,
          low: result.data.low.replace('℃', ''),
          high: result.data.high.replace('℃', ''),
          fengxiang: result.data.fengxiang,
          fengli: result.data.fengli,
        };
      } else {
        console.warn("韩小韩天气接口返回失败状态");
        throw "韩小韩天气接口失败";
      }
    } catch (hanError) {
      try {
        // 使用52vmy作为最后的兜底
        console.log("尝试使用52vmy天气接口");
        const result = await getVmyWeather().catch(error => {
          console.warn("52vmy请求失败:", error);
          throw error;
        });
        
        if (result.code === 200 && result.data) {
          weatherData.city = result.data.city || (result.data.current && result.data.current.city) || "未知地区";
          weatherData.data = {
            type: result.data.weather || (result.data.current && result.data.current.weather),
            low: (result.data.tempn || (result.data.current && result.data.current.tempn) || '').toString().replace('℃', ''),
            high: (result.data.temp || (result.data.current && result.data.current.temp) || '').toString().replace('℃', ''),
            fengxiang: result.data.wind || (result.data.current && result.data.current.wind),
            fengli: result.data.windSpeed || (result.data.current && result.data.current.windSpeed),
          };
        } else {
          console.warn("52vmy返回数据格式异常:", result);
          throw "52vmy天气接口数据异常";
        }
      } catch (vmyError) {
        console.error("所有天气接口均失败:", vmyError);
        weatherData.city = null;
        weatherData.data = {
          type: null,
          low: null,
          high: null,
          fengxiang: null,
          fengli: null,
        };
        onError("天气信息获取失败");
      }
    }
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