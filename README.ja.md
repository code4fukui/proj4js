# PROJ4JS [![Build Status](https://api.travis-ci.org/proj4js/proj4js.svg?branch=master)](https://travis-ci.org/proj4js/proj4js)

Proj4jsは、データム変換を含む、ある座標系から別の座標系へ点座標を変換するためのJavaScriptライブラリです。当初は[PROJ](https://proj.org/)（[当時はPROJ.4として知られていました](https://proj.org/faq.html#what-happened-to-proj-4)）およびGCTCP C（[アーカイブ](https://web.archive.org/web/20130523091752/http://edcftp.cr.usgs.gov/pub/software/gctpc/)）の移植版として開発され、[MetaCRS](https://trac.osgeo.org/metacrs/wiki)プロジェクト群の一部となっています。

## インストール

お好みの方法でインストールしてください：

```bash
npm install proj4
bower install proj4
component install proj4js/proj4js
```

または、[最新リリース](https://github.com/proj4js/proj4js/releases)の `dist/` フォルダから `proj4.js` ファイルを手動で取得してください。

ダウンロードを希望しない場合は、Proj4jsは[cdnjs](https://www.cdnjs.com/libraries/proj4js)でもホストされているため、ブラウザアプリケーションで直接使用することもできます。

## 使用方法

基本的なシグネチャは以下の通りです：

```javascript
proj4([fromProjection, ]toProjection[, coordinates])
```

投影法（Projections）はprojまたはwkt文字列で指定可能です。

座標は `{x:x,y:y}` 形式のオブジェクト、または `[x,y]` 形式の配列で指定できます。

3つの引数がすべて指定された場合、結果として座標はprojection1からprojection2に変換され、入力時と同じ形式で返されます。

```javascript
var firstProjection = 'PROJCS["NAD83 / Massachusetts Mainland",GEOGCS["NAD83",DATUM["North_American_Datum_1983",SPHEROID["GRS 1980",6378137,298.257222101,AUTHORITY["EPSG","7019"]],AUTHORITY["EPSG","6269"]],PRIMEM["Greenwich",0,AUTHORITY["EPSG","8901"]],UNIT["degree",0.01745329251994328,AUTHORITY["EPSG","9122"]],AUTHORITY["EPSG","4269"]],UNIT["metre",1,AUTHORITY["EPSG","9001"]],PROJECTION["Lambert_Conformal_Conic_2SP"],PARAMETER["standard_parallel_1",42.68333333333333],PARAMETER["standard_parallel_2",41.71666666666667],PARAMETER["latitude_of_origin",41],PARAMETER["central_meridian",-71.5],PARAMETER["false_easting",200000],PARAMETER["false_northing",750000],AUTHORITY["EPSG","26986"],AXIS["X",EAST],AXIS["Y",NORTH]]';
var secondProjection = "+proj=gnom +lat_0=90 +lon_0=0 +x_0=6300000 +y_0=6300000 +ellps=WGS84 +datum=WGS84 +units=m +no_defs";
// 後の例ではこれらの2つを再定義しません。
proj4(firstProjection,secondProjection,[-122.305887, 58.9465872]);
// [-2690575.447893817, 36622916.8071244564]
```

ライブラリは、標高（elevation）やメジャー（measure）を含む座標も解析可能です。これも `{x:x,y:y,z:z,m:m}` 形式のオブジェクト、または `[x,y,z,m]` 形式の配列として指定できます。

```javascript
proj4(firstProjection,secondProjection,[-122.305887, 58.9465872,10]);
// [-2690575.447893817, 36622916.8071244564, 10]
```

投影法が1つだけ指定された場合、WGS84 *から* 投影されるとみなされます（fromProjectionがWGS84になります）。

```javascript
proj4(firstProjection,[-71,41]);
// [242075.00535055372, 750123.32090043]
```

座標が指定されていない場合は、2つのメソッドを持つオブジェクトが返されます。そのメソッドは、最初の投影法から2番目の投影法へ投影する `forward` と、2番目から最初へ投影する `inverse` です。

```javascript
proj4(firstProjection,secondProjection).forward([-122.305887, 58.9465872]);
// [-2690575.447893817, 36622916.8071244564]
proj4(secondProjection,firstProjection).inverse([-122.305887, 58.9465872]);
// [-2690575.447893817, 36622916.8071244564]
```

また上記と同様に、投影法が1つだけ指定された場合は、WGS84からの変換とみなされます：

```javascript
proj4(firstProjection).forward([-71,41]);
// [242075.00535055372, 750123.32090043]
proj4(firstProjection).inverse([242075.00535055372, 750123.32090043]);
// [-71, 40.99999999999986]
```
注: この例で `40.99999999999986` という浮動小数点値が生成されているのは、ある座標参照系から別の座標参照系への変換には、ある程度の精度のばらつきが伴うという事実を表しています。

## 名前付き投影法

投影法を文字列として定義し、その方法で参照したい場合は、`proj4.defs` メソッドを使用できます。このメソッドは、名前と投影法を指定する2つの方法で呼び出すことができます：

```js
proj4.defs('WGS84', "+title=WGS 84 (long/lat) +proj=longlat +ellps=WGS84 +datum=WGS84 +units=degrees");
```

または配列で指定します：

```js
proj4.defs([
  [
    'EPSG:4326',
    '+title=WGS 84 (long/lat) +proj=longlat +ellps=WGS84 +datum=WGS84 +units=degrees'],
  [
    'EPSG:4269',
    '+title=NAD83 (long/lat) +proj=longlat +a=6378137.0 +b=6356752.31414036 +ellps=GRS80 +datum=NAD83 +units=degrees'
  ]
]);
```

その後、以下のように実行できます：

```js
proj4('EPSG:4326');
```

proj定義全体を書き出す代わりに、デフォルトでproj4には以下の投影法が事前定義されています：

- 'EPSG:4326'、以下のエイリアスがあります
    - 'WGS84'
- 'EPSG:4269'
- 'EPSG:3857'、以下のエイリアスがあります
    - 'EPSG:3785'
    - 'GOOGLE'
    - 'EPSG:900913'
    - 'EPSG:102113'

定義された投影法は、`proj4.defs` 関数を通じてアクセスすることもできます（`proj4.defs('EPSG:4326')`）。

`proj4.defs` は、名前付きエイリアスを定義するためにも使用できます：

```javascript
proj4.defs('urn:x-ogc:def:crs:EPSG:4326', proj4.defs('EPSG:4326'));
```

## 軸の順序

デフォルトでは、proj4は投影座標系（デカルト座標系）には `[x,y]` の軸順序を使用し、地理座標系には `[x=経度,y=緯度]` を使用します。提供されたprojまたはwkt文字列の軸順序を強制するには、以下のシグネチャで `enforceAxis` を `true` に設定して使用します：

```javascript
proj4(fromProjection, toProjection).forward(coordinate, enforceAxis);
proj4(fromProjection, toProjection).inverse(coordinate, enforceAxis);
```

```javascript
proj4('+proj=longlat +ellps=WGS84 +datum=WGS84 +units=degrees +axis=neu', firstProjection).forward([41, -71], true);
// [242075.00535055372, 750123.32090043]
proj4('+proj=longlat +ellps=WGS84 +datum=WGS84 +units=degrees +axis=neu', firstProjection).inverse([242075.00535055372, 750123.32090043], true);
//[40.99999999999986, -71]
//the floating points to answer your question
```

## グリッドベースのデータム調整

proj定義で `+nadgrids=` を使用するには、まずNTv2の `.gsb` ファイル（例: https://github.com/OSGeo/proj-datumgrid から取得）をArrayBufferに読み込み、それを `proj4.nadgrid` に渡します。例：

```javascript
const buffer = fs.readFileSync('ntv2.gsb').buffer
proj4.nadgrid('key', buffer);
```

その後、定義内で指定したキーを使用します。例: `+nadgrids=@key,null`。詳細は [Grid Based Datum Adjustments](https://proj.org/usage/transformation.html?highlight=nadgrids#grid-based-datum-adjustments) を参照してください。

## TypeScript

TypeScriptの型定義は [DefinitelyTyped リポジトリ](https://github.com/DefinitelyTyped/DefinitelyTyped) に追加されています。

```bash
$ npm install --save @types/proj4
```

## 開発

ビルドツールをセットアップするには、nodeとgrunt-cliがインストールされていることを確認し、`npm install` を実行します。

完全なビルドとブラウザテストを実行するには：

```bash
node_modules/.bin/grunt
```

Nodeテストを実行するには：

```bash
npm test
```

カバレッジ付きでNodeテストを実行するには：

```bash
npm test --coverage
```

デフォルトの投影法（緯度経度とメルカトル）のみを含むビルドを作成するには：

```bash
node_modules/.bin/grunt build
```

カスタム投影法のみを含むビルドを作成するには、コロンの後に投影法コード（'lib/projections' 内のファイル名から '.js' を除いたもの）のカンマ区切りリストを含めます。例：

```bash
node_modules/.bin/grunt build:tmerc
#includes transverse Mercator
node_modules/.bin/grunt build:lcc
#includes lambert conformal conic
node_modules/.bin/grunt build:omerc,moll
#includes oblique Mercator and Mollweide
```
