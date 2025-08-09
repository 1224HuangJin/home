<strong><h2>MiFengの主页</h2></strong>
**二改于[Imsyy/home](https://github.com/imsyy/home/) 仅自用不建议使用我的**

### 部署
 如果你并不知道怎么部署请看 <b>[这里](https://github.com/imsyy/home#%EF%B8%8F-%E8%87%AA%E5%8A%A8%E9%83%A8%E7%BD%B2)</b>

### 天气

> 天气api换成了[MIFENG API](https://api.mfawa.top/)


### 字体

采用 `HarmonyOS Sans` 开源字体，采用字体拆分，提升加载速度
（用的哔哩哔哩加速CDN！）
>
> `https://s1.hdslb.com/bfs/static/jinkela/long/font/regular.css`


### 网站图标及网站背景

#### 网站背景

可以在 `public/home-image` 中修改网站背景

如果想要添加更多的本地图片作为网站背景，可以将图片重命名 `background+数字` 的形式，并在 [`src/components/Background.vue`](src/components/Background.vue) 中进行修改：

```js
// 壁纸随机数
// 请依据文件夹内的图片个数修改 Math.random() 后面的第一个数字
const bgRandom = Math.floor(Math.random() * 10 + 1);
```
### This site is powered by Netlify
<a href="https://www.netlify.com">
  <img width="114" height="50" alt="Deploys by Netlify" src="https://www.netlify.com/v3/img/components/netlify-dark.svg" />
</a>

