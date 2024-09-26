---
title: Processing satellite imagery
#标题：处理卫星图像

description: Learn how to find satellite raster data, process it using the Rasterio command-line tool, and publish it on a webpage.
#说明：学习如何查找卫星光栅数据，使用Rasterio命令行工具处理数据，并将其发布到网页上。

thumbnail: processingSatelliteImagery

level: 3
topics:
- analysis
- satellite
language:
- JavaScript
prereq: Familiarity with front-end development concepts. Some advanced JavaScript required.
#前提：熟悉前端开发理念，需要一些高级的JavaScript能力

prependJs:
  - "import * as constants from '../../constants';"
  - "import Note from '@mapbox/dr-ui/note';"
  - "import BookImage from '@mapbox/dr-ui/book-image';"
  - "import Icon from '@mapbox/mr-ui/icon';"
  - "import UserAccessToken from '../../components/user-access-token';"
  - "import DemoIframe from '@mapbox/dr-ui/demo-iframe';"
  - "import Button from '@mapbox/mr-ui/button';"
  - "import AppropriateImage from '../../components/appropriate-image';"
contentType: tutorial
---
#内容类型：教程


在此教程中你将学习如何找到卫星栅格图像，然后用命令行工具进行图像处理。最后你会用Mapbox GL JS来创建一个地图，以展现迪拜景观从21世纪初到如今是如何变化的。

你将使用来自USGS（美国地质勘探局）陆地卫星的图像来完成这些，并用红、绿、蓝光波段来创建基于地理信息的合成图像，从而突出陆地和水的自然外貌。陆地卫星图像可以免费使用，也是你在其他资源中经常看到的经典图像。


{{
  <DemoIframe src="/help/demos/processing-satellite-imagery/index.html" />
}}


## 准备开始
如果你是第一次使用卫星图像或者对波段和栅格数据类型不熟悉，请在开始前阅读[How satellite imagery works guide（卫星图像操作指南）](/help/how-mapbox-works/satellite-imagery)。

以下是你完成这个教程需要用到的工具：

- **GDAL.** [GDAL](http://www.gdal.org/)是Rasterio所需要的低阶GIS工具包。按照你的操作系统推荐的步骤来安装GDAL。
- **Rasterio.** [Rasterio](https://github.com/mapbox/rasterio) 是用来读写地理空间栅格数据的工具。安装Rasterio时，在命令行输入命令`pip install rasterio`
- **美国宇航局地球数据账号**  拥有一个免费的[NASA Earthdata account（美国宇航局地球数据账号）](https://urs.earthdata.nasa.gov/users/new)能让你使用EarthExplorer（地球探索者），这是我们本次教程的数据来源。
- **你的Mapbox访问令牌**  你可以在Mapbox [Account page](https://www.mapbox.com/account/)账号页面中找到你的访问令牌。
- **文本编辑器**  使用你自己选择的文本编辑器来编写HTML, CSS和JavaScript

你还需要熟悉如何使用命令行来完成本次教程。


## 从EarthExplorer下载场景

为便于分布，陆地卫星图像数据被分割成 **场景**，它们大体上是正方形图像。你可以把某个场景看作是相机的取景框。一个陆地卫星场景大约有170 &times; 185公里（105 &times; 115英里）范围。

{{
    <Note
        title="陆地卫星成像过程"
        imageComponent={<BookImage />}
    >
        <p>如需更详细地了解陆地卫星的成像过程，请参阅 <a href='https://directory.eoportal.org/web/eoportal/satellite-missions/l/landsat-8-ldcm'>陆地卫星数据连续性任务文档</a>。</p>
    </Note>
 }}

在本次教程中，你将会比较阿拉伯联合酋长国迪拜的历史和现代景观，这些景观展现了迪拜在21世纪初经济增长时期前后的情况。为了做出比较，这个教程中的场景来自陆地卫星系列的两个不同的卫星。前一个图像来自陆地卫星5，它于2013年停止使用。后一个图像来自最新发射的陆地卫星8。这些卫星中的某些特征改变了，但是总体上陆地卫星项目保持了尽可能多的一致性，从而使比较变得可能。

在开始之前，创建一个新的文件夹来装入新文件。以下的说明参考了特定的陆地卫星场景，但是你可以选择任何你喜欢的场景。


### 下载陆地卫星5场景
1.	用你的Earthdata账号登陆到[EarthExplorer](http://earthexplorer.usgs.gov)
2.	在**搜索条件** 选项卡中，在**地址/位置** 搜索栏中键入“Dubai（迪拜）”，单击 **显示** 按钮，然后选择结果
3.	点击 **数据集** 选项卡
4.	点击 **陆地卫星**，然后是 **Landsat Collection 1 Level-1**，找到 _Landsat 4-5 TM C1 Level-1_ 数据集
5.	点击 _Landsat 4-5 TM C1 Level-1_ 旁的复选框进行选择
6.	点击 **附加条件** 选项卡
7.	将以下ID粘贴到 _陆地卫星产品标识符_ 中：`LT05_L1TP_160043_20011208_20180930_01_T1`.
8.	点击 **结果** 按钮
9.	点击结果数据产品下的下载图标（绿色向下箭头），然后点击 _Level-1 GeoTIFF 数据产品_ 选项旁边的 **下载** 按钮
10.	保存文件到项目文件夹中


### 下载陆地卫星8场景
1.	回到 **数据集** 选项卡，点击 _Landsat 4-5 TM C1 Level-1_ 数据集旁边的复选框取消选择
2.	点击 _Landsat 8 OLI/TIRS C1 Level-1_ 旁边的复选框进行选择
3.	点击 **附加条件** 选项卡
4.	将以下ID粘贴到 _陆地卫星产品标识符_ 中： `LC08_L1TP_160043_20181207_20181211_01_T1`
5.	点击 **结果** 按钮
6.	点击结果数据产品下的下载图标（绿色箭头向下），然后点击 _Level-1 GeoTIFF 数据产品_ 选项旁边的 **下载** 按钮
7.	保存文件到项目文件夹中


### 测试压缩包内容

你下载的Landsat Level 1产品是压缩的 `.tar.gz` 格式文件，称为 *压缩包*。需要解压缩包，使用的方法根据计算机的操作系统决定，或者你的网页浏览器可能会自动解压缩或完全解压缩它们。只要解压缩的压缩包是项目文件夹中的目录，就可以准确地遵循处理说明。

每个压缩包将解压到一个包含14个条目的目录中，大部分是TIFF图像。每个条目的名称均以陆地卫星产品ID开始并以波段数结尾，比如说：
 `LT05_L1TP_160043_20011208_20180930_01_T1_B1.TIF` 是陆地卫星5 `LT05_L1TP_160043_20011208_20180930_01_T1` 场景中波段1的读数

打开 `LT05_L1TP_160043_20011208_20180930_01_T1_B1.TIF` 来看看陆地卫星5场景中的波段1是什么样的


## 处理陆地卫星5图像

现在你已经拥有了想要的图像，你需要把图像处理一下，方可在实时地图中使用它们。你将完全从本章节的命令行开始工作，从前面下载的陆地卫星5压缩包开始。

在接下来的步骤中，你将把这些波段组合成一个单一图像，并将它们重新投影到Web Mercator (EPSG:3857)投影中，然后对图像进行颜色校正。


### 合成波段

本教程的目的是创建一个可见的图像，正常选择是用红绿蓝光波段来创建红绿蓝图像。但是陆地卫星5没有蓝光波段，所以你要用绿光替代蓝光，红光替代绿光，近红外光替代红光，从而做出一个假彩色图像。这对于科学分析是行不通的，但是对于可视化目的来说是可以的。

你使用的陆地卫星5波段数是3、2、1且分别映射到红、绿和蓝，要将这些波段叠加到RGB图像中，你需要使用 [`rio stack`](https://github.com/mapbox/rasterio/blob/master/docs/cli.rst#stack) 来合成波段1-3的TIFF文件并导出成新文件。

复制并粘贴以下命令到命令行：

```
rio stack --rgb LT05_L1TP_160043_20011208_20180930_01_T1/LT05_L1TP_160043_20011208_20180930_01_T1_B{3,2,1}.TIF landsat5_stack.tif
```

 `--rgb` 选项告诉查看软件（例如Photoshop）波段应该被解释为红、绿、蓝光。

这个命令行中的最后一个参数是新的输出文件 (`landsat5_stack.tif`). 这个参数 `LT05_L1TP_160043_20011208_20180930_01_T1_B{3,2,1}.TIF` 指定了需要合成的文件。该命令使用shell扩展来指定波段3、2、1，而不是单独命名每个波段文件。

命令运行后检查项目文件夹，你将看到新的 `landsat5_stack.tif` 文件。打开这个文件看看新的合成图像是什么样子。


### 重新投影图像

接下来你将使用 [`rio warp`](https://github.com/mapbox/rasterio/blob/master/docs/cli.rst#warp)来把合成图像重新投影到 [Web Mercator (EPSG:3857)](https://en.wikipedia.org/wiki/Web_Mercator_projection#EPSG:3857) 投影系中。 你要用 `rio warp` 通过双线性采样的方法来重新投影，从而不会导致像素化。

复制并粘贴以下命令到命令行：

```
rio warp --resampling bilinear --dst-crs EPSG:3857 landsat5_stack.tif landsat5_mercator.tif
```

选项 `--dst-crs EPSG:3857` 设置了Web Mercator的投影系。这个命令行中的最后一个参数是新的输出文件 (`landsat5_mercator.tif`)

运行该命令后，打开有重新投影图像的新文件 `landsat5_mercator.tif`，它看起来会和`landsat5_stack.tif` 很像，因为原始投影非常接近Web Mercator.


### 校正图像颜色

之后你将使用 [`rio color`](https://github.com/mapbox/rio-color)(Rasterio的颜色校正插件) 来改变图像的颜色，饱和度和对比度。记住 `rio color` 对可视化很有用，但并不适合分析。为进行分析，你需要一个单独的工作流程，其中包括顶层大气校准和大气校正。

复制并粘贴以下命令到命令行：

```
rio color landsat5_mercator.tif landsat5_color.tif gamma g 1.7 gamma r 1.4 sigmoidal rgb 10 0.4
```

这个命令对绿色波段应用1.7伽马，红色波段应用1.4伽马，并对所有波段应用 [sigmoidal contrast](http://www.imagemagick.org/Usage/color_mods/#sigmoidal) 。（如果你很好奇，可以在 `rio color` 命令中更改数字和操作，看看是否可以使图像更加清晰或具有吸引力）

打开新创建的 `landsat5_color.tif` 文件。你已经成功处理完成了陆地卫星5图像。


## 处理陆地卫星8图像

接下来你将处理陆地卫星8图像。

处理步骤和对陆地卫星5图像执行的基本相同，只是增加了一个重要的额外步骤。陆地卫星8的数据是由一个比陆地卫星5上的传感器具有更高辐射分辨率的传感器收集起来的。这意味着陆地卫星8数据将采用16位无符号整型，而陆地卫星5则是8位无符号整型。由于Mapbox只接受8位分辨率的图像，所以你需要将16位的陆地卫星8波段转换为8位。

在命令行中运行以下命令:

1.	合成波段：
 `rio stack --rgb LC08_L1TP_160043_20181207_20181211_01_T1/LC08_L1TP_160043_20181207_20181211_01_T1_B{3,2,1}.TIF landsat8_stack.tif`
 
2.	重新投影图像：
 `rio warp --resampling bilinear --dst-crs EPSG:3857 landsat8_stack.tif landsat8_mercator.tif`
 
之后你将使用 `rio color` 来把图像转换为8位格式。你还会在同一步骤中对图像进行颜色校正。

复制并粘贴以下命令到命令行：

```
rio color --co photometric=rgb --out-dtype uint8 landsat8_mercator.tif landsat8_color.tif sigmoidal rgb 20 0.2
```

打开新创建的 `landsat8_color.tif` 文件。你已经成功处理完成陆地卫星8图像。


## 用Mapbox GL JS来构建地图
现在你的图像已经处理好并准备上传，你将把它们作为瓦片集上传到你的Mapbox账户中，这样就可以在项目中使用它们了。


### 上传到Mapbox Studio

以下步骤展示了如何使用 Mapbox [tilesets](https://studio.mapbox.com/tilesets/) 页面把 GeoTIFFs上传到Mapbox，如果需要还可以使用 [Mapbox Uploads API](https://docs.mapbox.com/api/maps/#uploads) 来上传。

1.	导航到 [Tilesets](https://studio.mapbox.com/tilesets/) 页面
2.	点击 **新切片集** 按钮
3.	新窗口打开，选择最终的陆地卫星5文件 (`landsat5_color.tif`)，然后单击**确认**，当文件上传成功时页面会提示你
4.	对于最终的陆地卫星8文件 (`landsat8_color.tif`) 重复以上步骤

上传的切片集列举在你的 [tilesets](https://studio.mapbox.com/tilesets/) 页面。每个切片集都有各自的 [tileset ID](/help/glossary/tileset-id)，你可以通过点击列出的切片集右侧 **菜单** 栏找到它们。tileset ID允许你在使用开发工具（比如Mapbox GL JS或者Mapbox iOS 或者Android SDKs）时引用切片集.


### 将切片集和Mapbox GL JS作比较

接下来你将使用来自陆地卫星5和陆地卫星8切片集的tileset ID并用Mapbox GL JS来构建一个地图。这个项目使用[Mapbox GL Compare](https://github.com/mapbox/mapbox-gl-compare) 插件来比较陆地卫星5和陆地卫星8图像。

1.	打开你的文本编辑器并创建一个名为 `index.html`新文件
2.	复制并粘贴下面的代码到文本编辑器中，从而初始化一个 Mapbox GL JS地图
3.	将 `ACCESS_TOKEN` 占位符替换为你自己的Mapbox访问令牌，它位于你的 [账户页面](https://account.mapbox.com/)上
4.	用你的陆地卫星5切片集的tileset ID替换 `BEFORE_STYLE` 占位符，单击切片集右侧 **菜单** 可以在 [Tilesets](https://studio.mapbox.com/tilesets/) 页面上找到它们
5.	用你的陆地卫星8切片集的tileset ID替换 `AFTER_STYLE` 占位符，单击切片集右侧 **菜单** 可以在 [Tilesets](https://studio.mapbox.com/tilesets/) 页面上找到它们

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset='utf-8' />
    <title></title>
    <meta name='viewport' content='initial-scale=1,maximum-scale=1,user-scalable=no' />
    <script src='https://api.tiles.mapbox.com/mapbox-gl-js/{{constants.VERSION_MAPBOXGLJS}}/mapbox-gl.js'></script>
    <link href='https://api.tiles.mapbox.com/mapbox-gl-js/{{constants.VERSION_MAPBOXGLJS}}/mapbox-gl.css' rel='stylesheet' />
    <style>
      body {
        margin: 0;
        padding: 0;
      }

      #map {
        position: absolute;
        top: 0;
        bottom: 0;
        width: 100%;
      }
    </style>
</head>
<body>

<style>
body {
  overflow: hidden;
}

body * {
  -webkit-touch-callout: none;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

.map {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 100%;
}
</style>
<script src='https://api.mapbox.com/mapbox-gl-js/plugins/mapbox-gl-compare/v0.1.0/mapbox-gl-compare.js'></script>
<link rel='stylesheet' href='https://api.mapbox.com/mapbox-gl-js/plugins/mapbox-gl-compare/v0.1.0/mapbox-gl-compare.css' type='text/css' />

<div id='before' class='map'></div>
<div id='after' class='map'></div>

<script>
mapboxgl.accessToken = '{{ <UserAccessToken /> }}';

var beforeTileset = 'BEFORE_STYLE';
var afterTileset = 'AFTER_STYLE';

var beforeMap = new mapboxgl.Map({
  container: 'before',
  style: {
    version: 8,
    sources: {
      'raster-tiles': {
        type: 'raster',
        url: 'mapbox://' + beforeTileset,
        tileSize: 256
      }
    },
    layers: [{
      id: 'simple-tiles',
      type: 'raster',
      source: 'raster-tiles',
      minzoom: 0,
      maxzoom: 22
    }]
  },
  center: [55.1720, 25.0859],
  zoom: 11
});

var afterMap = new mapboxgl.Map({
  container: 'after',
  style: {
    version: 8,
    sources: {
      'raster-tiles': {
        type: 'raster',
        url: 'mapbox://' + afterTileset,
        tileSize: 256
      }
    },
    layers: [{
      id: 'simple-tiles',
      type: 'raster',
      source: 'raster-tiles',
      minzoom: 0,
      maxzoom: 22
    }]
  },
  center: [55.1720, 25.0859],
  zoom: 11
});

var map = new mapboxgl.Compare(beforeMap, afterMap, {});

</script>

</body>
</html>
```

在浏览器中打开文件。你将看到初始化的Mapbox GL JS地图显示在浏览器窗口中。左右拖动滑块可以比较迪拜景观的差异。


## 完成作品

你找到了自己的卫星图像，对其进行处理以获得最佳视觉效果，并展示了可视化景观随时间的变化。你还学习了如何使用命令行分析工具来处理你可能希望与Mapbox一起使用的任何类型栅格图像。


## 下一步

想了解更多，请完成 [Georeferencing imagery](/help/tutorials/georeferencing-imagery/) 教程，该教程将指导你完成一个手工处理影像配准图像或栅格数据（并未附带地理信息系统）的过程。你还可以查看 [Mapbox GL JS examples](https://www.mapbox.com/mapbox-gl-js/examples/) 页面，了解如何进一步扩展页面应用程序。
