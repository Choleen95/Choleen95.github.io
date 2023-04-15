---
title: Js引入百度地图
date: '2020-04-19 21:01'
categories: JS
abbrlink: b80605eb
tags:
---
工作中有时需要根据项目需求引入地图进行展示数据，现在做一个超简单的引入。推荐百度api，全是中文解释，很方便。[百度API*](http://lbsyun.baidu.com/cms/jsapi/reference/jsapi_reference_3_0.html#a3b11)
<!--more-->
### ***引入百度js***
1.新建一个html页面，引入js
```javascript
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta http-equiv="expires" content="0">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta content="width=device-width, initial-scale=1" name="viewport" />
   
    <script type="text/javascript" src="http://api.map.baidu.com/api?v=3.0&ak=gYR4wlvCmPyZYq8ggttE3fkGuBC5sOUB"></script>
    <script type="text/javascript" src="/static/lib/jquery.liMarquee/jquery.liMarquee.js"></script>
    <script>
        (function(a,h,g,f,e,d,c,b){b=function(){d=h.createElement(g);c=h.getElementsByTagName(g)[0];d.src=e;d.charset="utf-8";d.async=1;c.parentNode.insertBefore(d,c)};a["SeniverseWeatherWidgetObject"]=f;a[f]||(a[f]=function(){(a[f].q=a[f].q||[]).push(arguments)});a[f].l=+new Date();if(a.attachEvent){a.attachEvent("onload",b)}else{a.addEventListener("load",b,false)}}(window,document,"script","SeniverseWeatherWidget","/static/lib/xy.weather/widget2.bundle.js?t="+parseInt((new Date().getTime() / 100000000).toString(),10)));
    </script>
    <script src="js/TrackDisplayByBaidu.js"></script>
    <title>轨迹图</title>
    <style type="text/css">
                    .anchorBL{
                        display:none;
                    }
                </style>
</head>
<body>
     <div id="allmap"></div>
</body>
</html>
```     
这样html准备好了。     
2.创建自定义的js文件        
```javascript
/**
 * 实时作业监控JS
 */
var map = null;
var marker = null;
$(function() {
    // 百度地图API功能
    map = new BMap.Map("allmap", {
        enableMapClick:false
    });
    map.centerAndZoom("重庆", 13);
    map.addControl(new BMap.MapTypeControl({
        mapTypes:[
            BMAP_NORMAL_MAP,
            BMAP_SATELLITE_MAP,
            BMAP_HYBRID_MAP
        ]}
    ));
    map.enableScrollWheelZoom(true);
    // 添加比例尺控件
    map.addControl(new BMap.ScaleControl());
    // 初始化图标
    $('#allmap div.anchorBL').hide()
 
    $("#search_button").linkbutton({
       text:'搜索',
       iconCls:'icon-search',
       onClick:function () {
           //获取参数
           map.clearOverlays();
   
           $.ajax({
               url:'json/comment.json',
               method: "post",
               dataType: "json",
               data: params,
               async: false,
               success: function (data) {
                  $.each(data.rows,function (i,obj) {
                     if(obj.start_longitude){
                         doLocate(obj.start_longitude,obj.start_latitude,obj);
                     }
                     if(obj.end_longitude){
                         doLocate(obj.end_longitude,obj.end_latitude,obj);
                     }
                  });
               }
           });
       } 
    });
});
function doLocate(jd, wd,data) {//根据经纬度定位
   var point = new BMap.Point(jd, wd);
    marker = new BMap.Marker(point);
    map.addOverlay(marker);
    marker.enableDragging(); //启用标注拖动
  if(data){
      var taskId = data.trailer_task_id;
      var distance = data.distance;
       var opts = {
           width: 210, // 信息窗口宽度
           height: 125, // 信息窗口高度
           title: "<a href='javascript:;' onclick=\"generateTrack('"+taskId+"','"+distance+"')\">对应站点生成轨迹</a>", // 信息窗口标题
           enableMessage: true, //设置允许信息窗发送短息
           message: ""
       };
       var content = "起始地:"+ data.start_place+"<br>目的地:"+data.end_place+ "<br>经度:" + jd + "<br>纬度:" + wd;
       var infoWindow = new BMap.InfoWindow(content, opts); // 创建信息窗口对象

       marker.addEventListener("click", function() {
           map.openInfoWindow(infoWindow, point); //开启信息窗口
       });
   }

}
function generateTrack(taskId,distance){
    console.info(taskId);
    // 查询人员行进轨迹--画线
    $.ajax({
        url : "json/location.json",
        method : "post",
        dataType : "json",
        async : false,
        success : function (data) {
            // 添加个人轨迹
            var points = [];
            var rows = data.rows;
            $.each(rows, function (i, obj) {
                var currentTaksId = rows[i].taskId;
               if(currentTaksId == taskId){
                   $.each(rows[i].data,function (j,data) {
                       var point = new BMap.Point(data.longitude,data.latitude);
                       points.push(point);
                       if(j == 0){
                           map.setCenter(point);
                       }
                   });
                   $("#driver_naem").html(rows[i].driverName);
               }
            });
            var sy = new BMap.Symbol(BMap_Symbol_SHAPE_BACKWARD_OPEN_ARROW, {
                scale: 0.6,
                strokeColor:'#fff',
                strokeWeight: '2',
            });
            var icons = new BMap.IconSequence(sy, '10', '10');
            var polyline =new BMap.Polyline(points, {
                enableEditing: false,
                enableClicking: true,
                icons:[icons],
                strokeWeight:'8',
                strokeOpacity: 0.8,
                strokeColor:"#ff5c34"
            });
            map.addOverlay(polyline);
            var juli = 0;
            for (let i = 0; i < points.length; i++) {
               if(i == points.length -1){
                   break;
               }
                juli += getDistance(points[i],points[i+1]);
            }
            $("#current_mileage").html(distance+"/km");
            $("#latest_mileage").html((Math.round(juli/100)/10).toFixed(1)+"/km");
        }
    });
}

function getDistance(startPoint, endPoint) {
     return map.getDistance(startPoint ,endPoint);
}
```         

