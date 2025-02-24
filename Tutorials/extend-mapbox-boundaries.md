---
title: Extend Mapbox Boundaries
标题：扩展 Mapbox 边界

description: Extend the Mapbox Boundaries tileset.
描述：扩展Mapbox边界切片集

level: 3
topics:
- data
language:
- No code
thumbnail: extendEnterpriseBoundaries
prereq: Familiarity with front-end development, command line tools, and GIS. Access to Mapbox Boundaries.
基本要求：熟悉前端开发、命令行工具和GIS，访问Mapbox边界。

prependJs:
  - "import Note from '@mapbox/dr-ui/note';"
  - "import BookImage from '@mapbox/dr-ui/book-image';"
contentType: tutorial
---

{{
  <Note title='Access to Mapbox Boundaries' imageComponent={<BookImage />}>
    <p>Access to the Boundaries tilesets are controlled by Mapbox account access token. If you do not have access on your account, <a href='https://www.mapbox.com/contact/'>contact a Mapbox sales representative</a> to request access to Boundaries tilesets.</p>
  </Note>
}}

{{
  <Note title='访问Mapbox边界' imageComponent={<BookImage />}>
    <p>对边界切片集的访问由Mapbox账号访问令牌控制。如果你不能访问账号， <a href='https://www.mapbox.com/contact/'>联系Mapbox销售代表</a> 请求访问边界切片集的权限</p>
  </Note>
}}


In some cases, Mapbox Boundaries may cover most of your needs, but you may need a few specific school districts, neighborhoods, or sub-market boundaries for your customers. You can extend Mapbox Boundaries with any custom data and deliver the same frontend, API-driven services for data visualization, analysis, and geofencing applications around the world.

This guide walks through how to format, tile, and host data to work in your application alongside Mapbox Boundaries.

![final map in Mapbox GL JS](/help/img/data/extend-ent-boundaries-final-product.gif)

在一些情况下，Mapbox边界可能满足你的大部分需求，但是你可能需要一些特定的学区、社区或次级市场边界来满足你的客户。你可以使用任何自定义数据来扩展Mapbox边界，并提供相同的前端、API驱动的服务，用于在全世界范围内进行数据可视化、分析和地理围栏应用程序。

本指南将介绍如何格式化、切片和托管数据，以便在应用程序中与Mapbox边界一起工作。


## Getting started

Here are a few resources you'll need throughout this tutorial:

- **Access to Mapbox Boundaries**. Mapbox Boundaries are available as a part of an Enterprise plan. If you do not have an Enterprise plan or if you do have an Enterprise plan and would like to add access to Mapbox Boundaries, <a href='https://www.mapbox.com/contact/'>contact a Mapbox sales representative</a> to request access. Access to the Boundaries tilesets are controlled by Mapbox account access token.
- **Data**. In this tutorial, you'll learn how to add sub-county data from US Census to Mapbox Boundaries. Use the [US Census FTP site](ftp://ftp2.census.gov/geo/tiger/TIGER2016/COUSUB/) to download all files in the US Census Subcounty directory.
  - _Note: The process outlined in this guide requires that your custom data:_
    - _Uses polygon or multi-polygon geometries._
    - _Has metadata for each polygon feature denoting an identifier._
- **QGIS**. QGIS is an open source GIS application. You can download it from [http://www.qgis.org/](http://www.qgis.org/en/site/). You should be somewhat familiar with the QGIS interface before starting this tutorial.
- **Tippecanoe**. [Tippecanoe](https://github.com/mapbox/tippecanoe#tippecanoe) is a command line tool for creating Mapbox tilesets.
- **Mapbox CLI**. You'll use the [Mapbox CLI](https://github.com/mapbox/mapbox-cli-py/) (command line interface) to upload the tileset you create with Tippecanoe to your Mapbox account.

## 准备开始
以下是本教程中需要用到的一些资源：

- **访问 Mapbox 边界** Mapbox 边界作为企业计划的一部分是可以使用的。如果你没有企业计划或者你有企业计划并且想要添加对Mapbox边界的访问， <a href='https://www.mapbox.com/contact/'>联系Mapbox销售代表</a> 请求访问。对边界切片集的访问由Mapbox账户访问令牌控制。
- **数据** 在本教程中，你将学习如何将美国人口普查的次级郡数据添加到Mapbox边界。使用 [美国人口普查FTP站点](ftp://ftp2.census.gov/geo/tiger/TIGER2016/COUSUB/) 来下载美国人口普查次级郡目录中的所有文件。
  - _注意：本指南中概述的过程需要你的自定义数据:_
    - _使用多边形或者多面多边形几何_
    - _具有表示标识符的每个多边形特性的元数据_
- **QGIS**. QGIS是一个开源的GIS应用程序。你可以从[http://www.qgis.org/](http://www.qgis.org/en/site/)下载。在开始本教程之前，你应该对QGIS界面有一定了解。
- **Tippecanoe**. [Tippecanoe](https://github.com/mapbox/tippecanoe#tippecanoe) 是用于创建Mapbox切片集的命令行工具。
- **Mapbox CLI**. 你将使用[Mapbox CLI](https://github.com/mapbox/mapbox-cli-py/) (命令行界面) 把用Tippecanoe创建的切片集上传到你的Mapbox 账户。


## Format Data

You'll do two things to format the data before creating Mapbox tilesets:

1. **Concatenate the data** into one GeoJSON format file per administrative level.
1. **Generate a unique identifier** as a property value named id for each feature that matches the Mapbox Boundaries format.

![screenshot of a map of the United States in QGIS](/help/img/data/extend-ent-boundaries-map.png)

## 格式化数据

在创建Mapbox切片集之前，你需要做两件事来格式化数据：

1. 将**数据连接** 到每个管理级别的单独GeoJSON格式文件中。
1. 为了每个匹配Mapbox边界格式的特性而**生成唯一标识符** ，以此作为名为id的属性值。

![QGIS中的美国地图截图](/help/img/data/extend-ent-boundaries-map.png)


### Concatenate the data

In QGIS, open all the US sub-county shapes. Then, you'll need to:

1. Merge the subcounty Shapefiles together into one master Shapefile.
1. Create a unique identifier named `id`.
1. Project the data to `EPSG:4326 (WGS84)`.
1. Export the polygon data with needed properties to GeoJSON file.
1. Export the point data with needed properties to a GeoJSON file.

### 连接数据

在QGIS中，打开所有美国次级郡的地形图，然后你需要：

1. 将次级郡的Shapefile文件合并为一个主要的Shapefile文件
1. 创建一个名为id的唯一标识符
1. 将数据投影到 `EPSG:4326 (WGS84)`
1. 将具有所需属性的多边形数据导出到GeoJSON 文件
1. 将具有所需属性的点数据导出到GeoJSON 文件


### Generate a unique identifier

Next, you'll generate a unique identifier for each feature. The globally unique identifier should have the format:

- The leftmost 2 characters are the ID of the admin level 0 parent polygon &mdash; this is the 2-digit ISO country code.
- The third character from the left is `A`, `P`, or `S` representing admin, postal, or stats.
- The fourth character from the left is a number representing the admin or postal level.
- The remaining characters are the ID correspond to the individual admin or postal feature.
- To complete steps 1 and 2 on data format, either use your own data processing pipeline (such as PostGIS, SQLServer, Oracle Spatial) or a GIS tool such as ArcGIS or the open source tool QGIS.

The unique identifier will take the form `[CC-AL-D*]` where:

- `CC` is the two digit country code.
- `AL` is the two digit representation of admin-level (A for administrative, P for postal, S for statistical, and L for the level 0-5).
- `D*` is the administrative code for the feature.

In this example, to create the unique identifier in QGIS, open the merged layer properties and create a custom expression to add a new string field.

<img alt='screenshot of the add field window in QGIS' src='/help/img/data/extend-ent-boundaries-add-field.png' class='wmax360 mx-auto block'>

Sub counties should fall to level admin-3 in the Boundaries hierarchy &mdash; one level of detail down from counties at admin-2. Thus the unique ID should be the following formula `USA3` + `{admin code of sub county feature}`:

<img alt='screenshot of where to assign an id in QGIS' src='/help/img/data/extend-ent-boundaries-unique-id.png' class='wmax480 mx-auto block'>

When you are done editing the id, save the layer edits and export the shape file as a 6-coordinate-precision GeoJSON file in coordinate system `WGS84 (EPSG:4326)` named `admin-3-us-poly.geojson`.

Next create a point layer from the `INTERPLONG` and `INTERLAT` columns, which correspond to feature centroids. Export this centroid point layer to a GeoJSON file with coordinate precision 6 in coordinate system `WGS84 (EPSG:4326)` named `admin-3-us-point.geojson`.

{{
  <Note imageComponent={<BookImage />}>
    <p>If you are working with data that does not have pre-calculated centroid points, calculate the centroid using a GIS tool.</p>
  </Note>
}}

### 生成唯一标识符

接下来你将为每个特性生成一个唯一标识符。全球唯一标识符的格式应该是：

- 最左边的2个字符是管理级别0的父多边形的ID&mdash; 这是2位ISO国家代码
- 左边的第三个字符是表示管理级别，邮政级别或统计级别的`A`, `P`, or `S` 
- 左边的第四个字符是表示管理或邮政级别的数字
- 其余字符是对应于单一管理或邮政特性的ID
- 要完成数据格式的步骤1和步骤2，可以使用你自己的数据处理通道 (比如PostGIS, SQLServer, Oracle Spatial) 或者GIS工具比如ArcGIS或开源工具QGIS

唯一标识符的形式为`[CC-AL-D*]` ，其中：

- `CC` 是两位数的国家代码
- `AL` 是管理级别的两位数表示（A代表管理级别，P代表邮政级别，S代表统计级别，L代表0-5级）
- `D*` 是该特性的管理代码

在此例中，要在QGIS里创建唯一标识符，需打开合并图层属性并创建一个自定义表达式来添加新的字符串字段。

<img alt='screenshot of the add field window in QGIS' src='/help/img/data/extend-ent-boundaries-add-field.png' class='wmax360 mx-auto block'>

在边界层级结构中，次级郡应该下降到admin-3级别&mdash; 从admin-2的郡向下的一个详细级别。因此唯一ID应该遵循以下公式 `USA3` + `{次级郡特性的管理代码}`:

<img alt='screenshot of where to assign an id in QGIS' src='/help/img/data/extend-ent-boundaries-unique-id.png' class='wmax480 mx-auto block'>

编辑id之后，保存图层编辑并将shapefile文件导出为 `WGS84 (EPSG:4326)` 坐标系中名为 `admin-3-us-poly.geojson`的6-坐标-精度的GeoJSON文件。

接下来从 `INTERPLONG` 和 `INTERLAT` 列创建一个点的图层，它对应于特征质心。将这个质心点图层导出到坐标精度为6的GeoJSON 文件，该文件位于坐标系统 `WGS84 (EPSG:4326)` 中，名为 `admin-3-us-point.geojson`

{{
  <Note imageComponent={<BookImage />}>
    <p>如果你处理的数据没有预先计算的质心点，请使用GIS工具计算质心</p>
  </Note>
}}


## Create tiles

To create tilesets from your custom data, use the command line tool [Tippecanoe](https://github.com/mapbox/tippecanoe#tippecanoe) to transform your GeoJSON features to vector tiles. You'll need to create two source layers and merge them together into a single tileset source:

1. One source layer with a unique name for all polygons with an `id` property.
1. One source layer for with a unique name for centroid points with an `id` property. _Note: you can also encode other data beyond the `id` property, but it will result in bigger tiles and require you to set a higher min zoom for your tileset._

## 创建切片集

要从自定义数据创建切片集， 使用命令行工具 [Tippecanoe](https://github.com/mapbox/tippecanoe#tippecanoe) 来将 GeoJSON 特性转换为向量切片。你需要创建两个源图层并将它们合并到一个单独的切片集里。

1. 为所有具有 `id` 属性的多边形提供唯一名称的源图层
1. 为具有 `id` 属性的质心点提供唯一名称的源图层 _注意：你还可以对 `id` 属性之外的其他数据进行编码，但这会导致更大量的切片集并会因此要求你设置更高值的最小缩放_


### Create the polygon source layer

Start by creating a polygon source layer. Use the following Tippecanoe command to transform the example files.

Substitute the name of your layer in for the `-l` parameter, and adjust the minimum zoom of the tileset with the `-Z` parameter. Typically admin-2 features will have `-Z2`, admin-3 will have `-Z4`, admin-4 will have `-Z6`, and admin-5 will have `-Z7`.

```bash
tippecanoe -P -f -o admin-3-poly-us.mbtiles -l boundaries_admin_3_us -Z4 -z12 admin-3-us-poly.geojson -y "id"
```

### 创建多边形源层

开始先创建一个多边形源图层，使用下面的Tippecanoe命令来转换示例文件。

将图层名称替换为 `-l` 参数，并使用 `-Z` 参数调整切片集的最小缩放。通常，admin-2的特性有 `-Z2`, admin-3 有`-Z4`, admin-4 有 `-Z6`, admin-5 有 `-Z7`.

```bash
tippecanoe -P -f -o admin-3-poly-us.mbtiles -l boundaries_admin_3_us -Z4 -z12 admin-3-us-poly.geojson -y "id"
```


### Create the point source layer

Next, create a point source layer using the following Tippecanoe command. You can adjust the `-Z` value to specify the minimum zoom of the tileset &mdash; this will help keep tile sizes small. Generally, you can pick a higher minimum zoom level the more dense your features are.

```bash
tippecanoe -P -f -o admin-3-point-us.mbtiles -l points_admin_3_us -Z4 -z12 -r0 admin-3-us-point.geojson -y "id"
```

### 创建点源层

接下来使用下面的Tippecanoe命令来创建一个点源层。你可以调整 `-Z` 值以指定切片集的最小缩放 &mdash; 这将有助于保持切片是小尺寸的。通常来说，你选择的最小缩放级别越高，你的特性就越密集。

```bash
tippecanoe -P -f -o admin-3-point-us.mbtiles -l points_admin_3_us -Z4 -z12 -r0 admin-3-us-point.geojson -y "id"
```


### Create a single tileset

Then, join the two polygon and point layers into one vector tileset using the following command:

```bash
tile-join -f -o admin-3-us.mbtiles admin-3-poly-us.mbtiles admin-3-point-us.mbtiles
```

Upload the tileset to your Mapbox account. Start by installing the [Mapbox CLI](https://github.com/mapbox/mapbox-cli-py/). Adjust the tileset name to use your Mapbox account name and level with a suffix to denote that it is unique from Mapbox Boundaries.

```bash
pip install mapboxcli
export MAPBOX_ACCESS_TOKEN=MY-UPLOAD-SCOPE-TOKEN
mapbox upload "admin-3-us.mbtiles" "MY-ACCOUNT-NAME.enterprise-boundaries-a3-us"
```

{{
  <Note title='Alternative upload process' imageComponent={<BookImage />}>
    <p>Alternatively, you can upload your tileset using <a href='https://www.mapbox.com/studio/tilesets/'>Mapbox Studio</a>.</p>
  </Note>
}}

### 创建一个切片集

然后使用以下命令将这两个多边形和点图层连接到一个向量切片集中：

```bash
tile-join -f -o admin-3-us.mbtiles admin-3-poly-us.mbtiles admin-3-point-us.mbtiles
```

上传切片集到你的Mapbox账号。开始安装 [Mapbox CLI](https://github.com/mapbox/mapbox-cli-py/). 调整切片集名称，使用带后缀的Mapbox账号名称和级别，使Mapbox边界具有独特性。

```bash
pip install mapboxcli
export MAPBOX_ACCESS_TOKEN=MY-UPLOAD-SCOPE-TOKEN
mapbox upload "admin-3-us.mbtiles" "MY-ACCOUNT-NAME.enterprise-boundaries-a3-us"
```

{{
  <Note title='选择上传过程' imageComponent={<BookImage />}>
    <p>比如，你可以使用 <a href='https://www.mapbox.com/studio/tilesets/'>Mapbox Studio</a>上传你的切片集</p>
  </Note>
}}


## Composite tiles

After the tileset has been successfully added to your account, you can use the syntax below to composite Boundaries with custom boundaries in a map [style](/help/glossary/style/). This puts all tile data into one API request, maximizing performance and minimizing API calls. Compositing also improves label placement calculations across tilesets.

```json
"sources": { "composite": { "url": "mapbox://mapbox.enterprise-boundaries-a3-v2,MY-MAPBOX-ACCOUNT.enterprise-boundaries-a3-us", "type": "vector" } }
```

{{
  <Note imageComponent={<BookImage />}>
    <p>Find the complete list of Mapbox Boundaries tilesets in the <a href="https://www.mapbox.com/vector-tiles/enterprise-boundaries-v2">reference documentation</a>.</p>
  </Note>
}}

## 合成切片

在切片集成功地添加到你的账户后，你可以使用下面的语法，在map [style](/help/glossary/style/)中用自定义边界来合成边界。这将把所有切片数据放入到一个API请求中，最大化性能并最小化API调用。同时，这种合成提高了切片集的标签放置的计算速度。

```json
"sources": { "composite": { "url": "mapbox://mapbox.enterprise-boundaries-a3-v2,MY-MAPBOX-ACCOUNT.enterprise-boundaries-a3-us", "type": "vector" } }
```

{{
  <提示 imageComponent={<BookImage />}>
    <p>在 <a href="https://www.mapbox.com/vector-tiles/enterprise-boundaries-v2">参考文档</a>中找到Mapbox边界切片集的完整列表</p>
  </Note>
}}


## Final product

Here is an example style showing how the new US Admin Level 3 data works alongside the base Mapbox Boundaries product.

![final map in Mapbox GL JS](/help/img/data/extend-ent-boundaries-final-product.gif)


## 最终产品

下面是一个示例样式，展示了美国Admin Level 3新数据是如何与Mapbox边界产品协同工作的。

![final map in Mapbox GL JS](/help/img/data/extend-ent-boundaries-final-product.gif)


## Next steps

Learn more about how you can use Mapbox Boundaries:

- [Point-in-polygon query with Mapbox Boundaries](/help/tutorials/point-in-polygon-query-with-enterprise-boundaries/): Determine what polygons exist at a single point using the Mapbox Tilequery API.
- [Data-joins with Mapbox Boundaries](/help/tutorials/data-joins-with-enterprise-boundaries/): The data-join technique involves inner joins between local data, such as the unemployment rate by US state, to vector tile features, such as admin boundaries in Mapbox Boundaries, using data-driven style notation.

## 下一步

了解更多关于如何使用Mapbox边界：

- [带Mapbox边界的点内多边形查询](/help/tutorials/point-in-polygon-query-with-enterprise-boundaries/): 使用Mapbox Tilequery API 来确定在某点上存在哪些多边形
- [与Mapbox 边界的数据连接](/help/tutorials/data-joins-with-enterprise-boundaries/): 数据连接技术使用了数据驱动的表达风格，包含了将本地数据（如按美国各州划分的失业率）和向量切片特性（如Mapbox边界中的管理边界）之间的内部连接。
