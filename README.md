# GeoSOT-iWhere-API

The OpenAPI building blocks defined by the specification as well as complete bundled example API definition are available [here](https://github.com/AkexStar/GeoSOT-iWhere-API/tree/main/openapi), and can also be visualized and experimented with an example implementation with [SwaggerUI](https://app.swaggerhub.com/apis/Akex/GeoSOT-iwhere-openapi/1.2.4#/) here.

遵循openAPI规范的json文档在[这里](https://github.com/AkexStar/GeoSOT-iWhere-API/tree/main/openapi)获取，本项目同时使用[SwaggerUI](https://app.swaggerhub.com/apis/Akex/GeoSOT-iwhere-openapi/1.2.5#/)进行可视化，可在上面进行在线接口尝试。

<img width="1911" alt="image" src="https://user-images.githubusercontent.com/55226358/227976408-53dc93ba-eef9-4187-a9bb-f87690072c31.png">

## 什么是GeoSOT-2D

GeoSOT（Geographic Coordinate Subdividing Grid with One Dimension Integral Coding on 2n-Tree）是基于2n一维整型数组全球经纬剖分网格。

GeoSOT-2D全球经纬剖分网格是基于地理坐标系统划分的网格系统，可以无缝覆盖全球，并且网格之间连接较为均匀、稳定，其剖分网格具有可标识性、层次性、聚合性和关联性等特点，可实现层次之间的有机关联，为地图多尺度表达提供基础。该网格通过经纬网进行划分，因此与国家标准图幅很好地聚合和关联，能够方便地与现在主要的多种类型数据进行转换，有利于多源数据的兼容以及跨部门、跨系统之间的空间数据的整合和共享。

GeoSOT 网格通过地球表面经纬度范围空间经过3次空间扩展（将地球地理空间扩展为512°、将1°扩展为64′、将1′扩展为64″），实现了整度、整分的整型四叉树剖分网格，具体网格划分方法如下：

- 第1次扩展：先将地球通过简单投影变换为平面，将180°×360°扩展为512°×512°，作为第0级剖面，以本初子午线与赤道为交点为中心点，递归四叉剖分，直到1°网格单元，如图1，又可称为度级剖分网格，此次剖分包含10级，即0~9级。
  - 第0级网格是以本初子午线与赤道为交点为中心点的512°×512°单元格，第0级网格编码为G，意为全球（Globe）。
  - 第1级网格在第0级剖分网格基础上平均分为四份，每个网格大小为256°×256°；第1级网格编码为Gd，其中d为0、1、2或3。G0就对应空间信息区域位置为东北半球。
  - 第2级网格在第1级剖分网格基础上平均分为四份，每个网格大小为128°×128°。 第2级网格编码为Gdd，其中d为0、1、2或3。G00就对应空间信息区域位置为东北半球大部。第2级剖分网格中，部分网格没有实际地理意义，例如G02与G03，不再进行划分；其它第2级网格作为一个整体进入下一级网格的划分，以下层级网格的划分相同。
  - 部分2级网格没有实际地理意义，不再向下划分，如图c所示。其他2级网格虽然有部分区域落在实际地理区域范围之外，仍然可以作为一个整体进行下一级网格划分，这种原则同样适用于以下网格的划分。
  - 第3级网格是第2级剖分网格基础上平均分为四份，每个网格大小为64°×64°；第3级网格编码为Gddd，其中d为0、1、2或3。G001就对应空间信息区域位置为中国、印度与东南亚。
  - 下层网格剖分原则以此类推。
- 第2次扩展：是将1°网格单元从60′扩展为64′，然后递归四叉剖分，直到1′网格单元。又可称为分级剖分网格，此次剖分包含6级，即10~15级，第10级网格定义为在分级网格根节点基础上平均分为四份，每个网格大小为32′×32′；第10级网格编码为Gddddddddd-m，其中d、m取值0、1、2或3的四进制数。下层网格剖分原则以此类推。
- 第3次扩展：是将1°网格单元从60″扩展为64″，然后递归四叉剖分，直到1″网格单元。1″以下剖分单元直接采用四叉分割，直到32级。秒级剖分网格是从第16级到第21级剖分，其秒级网格根节点与第15级网格（1′网格或60″网格）一一对应，且编码相同，网格大小从60″扩展到64″，如图3所示。GeoSOT 秒级剖分在第16级网格开始，定义为在秒级网格根节点基础上平均分为四份。第16级网格编码形式为：Gddddddddd-mmmmmm-s，其中d、m、s取值0、1、2或3的四进制数。下层网格剖分原则以此类推。
- 秒以下22级-32级网格严格按照四分方法进行剖分和编码。

按照上述的剖分层级定义，GeoSOT网格一共分为32个层级，大到全球、小到厘米，均匀地将地球表面空间划分为多层次的网格，这些网格形成了全球四叉树系统。GeoSOT网格上下级别之间的面积之比大致都为4:1，是均匀变化的。

**2D网格单元剖分编码**

依据GeoSOT网格剖分原理，将GeoSOT网格编码采用64位编码对各级剖分网格进行标识。

最长的编码位为32位四进制数值编码。第1～9位是度级网格编码，第10～15位是分级网格编码，第16～21位是秒级网格编码，第22～32位是秒以下网格编码，编码长度即为网格层级。

GeoSOT 网格的四进制1维编码是以G开头，度、分、秒级编码以“-”隔开，秒以下的编码以“.”隔开，其形式为“Gddddddddd-mmmmmm-ssssss.uuuuuuuuuuu”。其中d、m、s、u取值均为0、1、2、3。具体编码规则是，距赤道和本初子午线的交点最近的剖分网格为0，最远的为3，然后按照先沿纬线方向再沿经线方向对其他两个剖分网格分别为1和2。

通过这种编码方式，实现对每个GeoSOT 网格单元进行编码且该编码全球唯一。同时，由于 GeoSOT 网格中每个网格在地球上具有确定的地理空间范围，因此 GeoSOT 网格单元剖编码具有了准确的地理空间含义，可在某种程度上具有地理空间坐标的意义。

## GeoSOT-3D

2n一维整型数组的全球等经纬度剖分网格系统（Geographical coordinate global Subdivision grid with One-dimension-integer on Two to n-th power，简称GeoSOT）。GeoSOT网格设计的核心思想是：对自地心至地球外围5万km的地球立体空间，进行八叉树剖分，并且将地球经纬度空间进行三次扩展 （将地球及其邻近经纬度空间扩展为512°×512°×512°，将1°扩展为64′，将1′扩展为64″），从而实现整分整度整秒的剖分网格，形成一个大到整个地球空间（0级）、小到厘米级体元（32级）的多尺度八叉树网格树，同时涵盖4˚、2˚、1˚、2′、1′、2″、1″、0.5″八个基本体元或面片。其网格划分方式的核心思路如图所示。

![](http://www.fuxitek.com/upload/other/image/20171124151150178241.jpg)

其中，基于经纬度坐标的地球立体空间如图所示，其经纬度坐标以本初子午线与赤道的交点为原点。纬圈为等间隔、等长的直线，经线为与纬线垂直的、等间隔、等长的直线，经线长度与纬线长度之比为1:2，北极点、南极点成为与纬线平行且等长的直线；纬度范围是-90˚到90˚，经度范围是-180˚到180˚。在空间高度上，以参考椭球中心为0，空间高度最大到为512˚，地球表面大约在高度为180˚/π附近，最大高度离地面大约为5万公里。设定高度单位是度分秒，参考椭球参数，建立空间高度单位与千米、米的换算关系：D˚×πR/180˚=H km。其中，R为地球平均半径，H为地心与该经纬度平面的高程。

![](http://www.fuxitek.com/upload/other/image/20171124151150185046.png)

在GeoSOT八叉树经纬度剖分网格的基础上，我们对网格构建一一对应的层次性编码。第0层为整个地球立体空间，标记为G，自G开始每一级网格都在上一级网格基础上采用Z序编码，Z序编码方向与该网格所在1级网格相关。（如图3所示）GeoSOT网格编码共具有四种形式： **八进制1维编码、二进制1维编码、二进制3维编码、十进制3维编码。** 这四种形式是完全对应、一致的，并且相互之间可以方便转换。

![img](http://www.fuxitek.com/upload/other/image/20171124151150192069.png)

GeoSOT地球空间参考网格编码最长采用32位8进制数值（0、1、2、3、4、5、6、7）,第1-9位是度级网格编码,第10-15位是分级网格编码,第16-21是秒级网格编码,第22-32位是秒以下网格编码；编码长度表示网格层级，编码长度越长，网格越细；书写编码时,前面以G开头，度分秒级编码以“-”隔开，秒以下编码以“.”隔开,形式如下：

**Gddddddddd-mmmmmm-ssssss.uuuuuuuuuuu**

当GeoSOT地球空间参考网格高度为大地高时（或地球平均半径），此时高度维退化，真三维地球空间参考网格框架就变为二维球面上的参考网格框架，我们称之为GeoSOT球面参考网格。二维球面的网格划分方式为：对地球表面空间进行四叉树剖分，并且将地球经纬度空间进行三次扩展(将地球表面经纬度空间扩展为512°×512°，将1°扩展为64′，将1′扩展为64″)。其中，地球表面空间以赤道与本初子午线的交点为原点，纬度范围是-90°到90°，经度范围从-180°到180°。

GeoSOT球面参考网格编码与GeoSOT地球空间参考网格编码形式与结构一致，GeoSOT球面参考网格也采用Z序编码，并具有四种形式：四进制1维编码、二进制1维编码、二进制2维编码、十进制2维编码。

## 参考论文

- Li, S., Cheng, C., Chen, B., Meng, M., 2016, “Integration and management of massive remote-sensing data based on GeoSOT subdivision model”, J. Appl. Remote Sens. 10(3), 034003, DOI:10.1117/1.JRS.10.034003
- Cheng, C., Tong, X., Chen, B., Zhai, W., 2016, “A Subdivision Method to Unify the Existing Latitude and Longitude Grids”, ISPRS International Journal of Geo-Information, 5(9), 161, DOI:10.3390/ijgi5090161

## 文件构成说明（WIP）
