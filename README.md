# 介绍
## lineChart
	* 是基于vue开发的折线图普,使用H5 canvas实现效果
	* lineChart提供快速生成折现图谱功能,简单好用
	* 大小由外部盒子控制,可完全自定义
	* 自适应屏幕,改变屏幕会自行重新渲染组件

## 安装

```cmd
npm install line-chart-vue --save
```

## 使用
```javascript
import lineChart from 'line-chart-vue'

components:{
	lineChart
}
```
	
```html
<div class='wrapper'>
 <line-chart></line-chart>
</div>
```

## 参数

```html
<line-chart :rowData='rowList'></line-chart>
<!--参数以此逻辑传递-->
```

#### rowData 
	数组 默认空(必填,否则组件为空)
	折线图的中的数据 
	数据格式请自行调整为一维数组[1,2,3,4,...](为了数据调用时更方便)
	组件会自动判断数据中最大值和最小最,然后将Y轴的值等分
#### row 
	数字 默认5(个)
	Y轴坐标的个数
#### colData 
	数据 默认空(非必填)
	X轴需要填写的值(可不填,将以自然数递增)
	此数据存在时,请自行调整为一维数组,长度需和rowData相同
#### colStart 
	数字 默认1
	当colData数据为空时,可自定义X轴的开始值,此后以自然数递增
#### load
	布尔值 默认true(开启)
	是否开启可下载canvas图片功能
#### imgType
	字符串 默认'jpg' (直接写时记得加'')
	图片下载的格式,提供png|jpeg|jpg|bmp|gif格式选择
#### hoverLine
	布尔值 true(开启)
	是否开启数据定位功能,开启后会将鼠标指中数据和X轴垂直连线
#### playing
	布尔值 true(开启)
	是否开启动态画图功能,开启后折线图会是动画画出线路
