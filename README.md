# uni-simple-address
根据uniapp插件作者[SimpleJalon](https://ext.dcloud.net.cn/publisher?id=44551)的[三级地址联动，支持（app）nvue、小程序、H5](https://ext.dcloud.net.cn/plugin?id=1084)插件替换了地区数据源，使用了[基于 puppeteer 的中国行政区域抓取爬虫](https://github.com/dwqs/area-puppeteer)的格式化数据文件pcaa.js替换了原来的province.js/city.js/area.js三个文件。会比原插件体积上小一点点。

替换原因主要为需要和后台系统中已经使用的VUE省市区组件数据同步，所以引用了同一个地区数据源

## 使用方式

直接复制代码中simple-address文件夹，作为component放在components下使用

## 安装
```
yarn install
```

### 运行DEMO
```
yarn serve
```
