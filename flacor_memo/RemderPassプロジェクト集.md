# RenderPassプロジェクト集

Mogwaiで利用されるレンダーパスのdll作成プロジェクト集    

パス作成関数create()はpybind11によってMogwaiでpythonから呼ばれ、  
パスコンパイルcompile()、インプットアウトプット情報設定関数reflect()はMogwai上でのパス作成時に呼ばれ、  
excute()がレンダーグラフの描画時、つまりつなげたパスごとに順々に呼ばれていく  

renderUI()はMogwaiエディターでのUI表示設定  

その他Mogwai上でキー操作を行った時とかホットリロードとかシーン読み込みとかの実装も場合によっては行う  

Mogwaiはpythonで実行されるので、パスの変数設定関数などMogwai側で呼び出したいその他関数をpybind11によって呼び出せるようにするように、以下のような関数も実装している  

    extern "C" __declspec(dllexport) void getPasses(Falcor::RenderPassLibrary& lib) {…}  

## AccumulatePassプロジェクト
### AccumulatePass

渡したリソースをフレームごとに蓄積し、アウトプットリソースにその平均を返すパス  

カメラの移動などのシーン変化やリソースの変更、パス設定の変更などで蓄積がリセットされる  

蓄積の仕方は全て総和による平均で、制度によって以下の違いがある    
Single : float精度総和バッファ  
Double : double精度総和バッファ、floatバッファ2つ用いることによってdouble精度を再現している  
SingleCompensated : カハンの加算アルゴリズム、float総和バッファの他に、総和で精度から漏れてしまった数値が補正バッファに格納される  

in : ShaderResource
out : RenderTarget | UnorderedAccess | ShaderResource & RGBA32Float
inとoutのリソースサイズは同じでなければならない  

## Antiailiasingプロジェクト

アンチエイリアシングパス  
FXAAとTAAの2種類がある  

Antiailiasing.cppは各パスのpython登録処理のみ  

### FXAA

FXAAはTAAが使えない時用。サブピクセルを使わず、周辺ピクセルの輝度差によりエッジを検出し周辺の色とブレンドする  

FullScreenPassを使ってFXAA.slangを通すだけ  
故に
in : Texture2D 
out : Texture2D
assertしてないが、inとoutは同じサイズ、ミップである必要がある  

TODO : 設定できる値とシェーダー はよくわからん。FXAAのアルゴリズムとか使い方とかを要確認
[WebGL レイマーチングでアンチエイリアス（FXAA）してみる](https://qiita.com/edo_m18/items/c211fea23b4747a8da3c)  
[Filtering Approaches for Real-Time Anti-Aliasing](http://iryoku.com/aacourse/downloads/09-FXAA-3.11-in-15-Slides.pptx)

### TAA
TAAは前フレームの描画情報を用いたアンチエイリアス。ディファード、レイトレでよく使われている  

まず今の色の自分含め周りの9ピクセルで平均と偏差をとり、色の誤差範囲*gColorBoxSigmaを計算し、  
モーションベクターのうち、周りのピクセルで一番大きいのを持ってきて昔の色を取得し、  
昔の輝度が今の輝度の誤差範囲内に収まっていた場合はその差からブレンド値*gAlphaを計算しブレンドする  

ちゃんとしたアルゴリズムの説明はここらへんが参考になるらしい
http://www.gdcvault.com/play/1023521/From-the-Lab-Bench-Real  
http://cwyman.org/papers/siga16_gazeTrackedFoveatedRendering.pdf  
https://de45xmedrsdbp.cloudfront.net/Resources/files/TemporalAA_small-59732822.pdf  

in : モーションベクターTexture2D
in : Texture2D 
out : Texture2D
ただし、すべてのテクスチャーのサイズ、ミップは同じ必要があり、  
SampleCount = 1である必要がある  

## BlitPasプロジェクト
### BlitPas
blit実行用パス  
リソースをレンダーターゲットへサンプラー使って描画  

設定はサンプラーがPointかLinearか選べるのみ  

in リソース
out リソース
arrayとmipは1のみ、各々のリソースのサイズは制限なし  

## BSDFViewerプロジェクト
### BSDFViewer
読み込んだシーンのマテリアルの見た目を確かめるためのパス  
imguiでBSDFの設定やカメラや表示したいマテリアルなどの指定を行いマテリアルの見た目を確認する用  

sliceViewerがoffなら球体を（useDirectionalLightじゃなければレイトレで）BxDF.slangのBSDFを用いて描画する  
球体がレイトレの場合は単純なそのフレーム1サンプルのみのDXRも用いてない（そこらへんはすべてBSDFViewer.cs.slangで実装されている）描画なので、完全確認用だと思われる（AccumulatePassをこの後にかましてもいいかも）  

sliceViewerがonなら指定したtexCoordsuvの情報に対するuvを緯度経度に変換した画像を表示  
sliceViewerの場合はレイトレではないので、もしかしたらほかのパスで使えるのかもしれない  

キーやマウス操作はマテリアルの変更とクリックしたピクセル情報表示のみ  

out : RGBA32Float、UnorderedAccess  
このパスはoutのみ  

## CSMプロジェクト
### CSM
カスケードシャドウマップのパス  
シーン上のライト配列のうち最初のライトのCSMを作成し、そのCSMをもとにシーンのカメラから見える影パラメーターを描画したものを返す（0で完全に影、1で完全に影なし）  

まずシャドウマップのz方向範囲として[SDSM (Sample Distribution Shadow Map)](https://software.intel.com/content/www/us/en/develop/articles/sample-distribution-shadow-maps.html)を使うかどうかの設定がある  
パスのinputからのDepthテクスチャーにParallelReductionのMinMaxシェーダーを使い、Depthの最小値、最大値からCSMを作る範囲を決定する  
SDSMを使わない場合はUIで設定した範囲で計算される  

分割の仕方は
- Linear : 等間隔
- Logarithmic ; 対数間隔
- PSSM : Parallel Split Shadow Map。シャドウのピクセルの大きさが画面から見たときにちょうど1ピクセルの大きさに近くなるよう分割箇所を調整する感じのやつ  
[ASURA 平行分割シャドウマップ](http://www.project-asura.com/program/d3d11/d3d11_009.html)
[GPU Gems 3 Chapter 10. Parallel-Split Shadow Maps on Programmable GPUs](https://developer.nvidia.com/gpugems/gpugems3/part-ii-light-and-shadows/chapter-10-parallel-split-shadow-maps-programmable-gpus)

見た目重視ならPSSMを選ぶといいはず（pssmLambda = 0.5がGPU Gems 3で使われている値）  

シャドウマップのエッジぼやかしフィルターは
- pcf系列 : Percentage Closer Filtering [ref1 (fixed and stochastic)](https://developer.nvidia.com/gpugems/gpugems/part-ii-lighting-and-shadows/chapter-11-shadow-map-antialiasing)
- vsm系列 : Variance Shadow Map [ref1](https://hexadrive.jp/lab/demo/572/), [ref2](http://asura.iaigiri.com/XNA_GS/xna33.html)
pcf系列とは違う点として、これらを使うときはシャドウマップ生成後GaussianBlurパスによってブラーがかけられ、シャドウマップの参照にddx, ddyによるミップ取得があるため、ミップマップ生成とトライリニアサンプラーが必要となる    
  - evsm : Exponential Variance Shadow Mapping [ref1](https://www.martincap.io/project_detail.php?project_id=9)
2がExponentialの正項のみ、4が正負両方使う  

比較例 
[vsm vs esm vs evsm](https://www.cg.tuwien.ac.at/research/publications/2013/ADORJAN-2013-ASE/ADORJAN-2013-ASE-thesis.pdf)
[pcf vs vsm vs evsm](https://www.martincap.io/project_detail.php?project_id=9)
[vsm系列の方がpcfより1.5倍速い説](https://community.khronos.org/t/shadow-filtering-pcf-better-than-vsm/67180)
ざっと調べた感じ、同じくらいの画像クオリティ比較で、vsmの方がpcfより速そう  

in : Depth。ない場合はこのパスで自作するようになっているとあるが、おそらく今は渡さないとだめっぽい（ただしinに線をつなげてなくても、 RenderDataに kDepthが入っていれば動くので、つないでもつながなくても大丈夫。というかレンダーグラフの処理がすべての使われているリソースをRenderDataぶちこんで全て渡すという処理なので線の意味がなくなっている）  
out : VisivilityFBO、カメラから見た影情報。Rが影に対応し、1で影なし、0で完全に影となっている。（コード側でしか設定できないが）visualizeCascadesをオンにしている場合はgbaにシャドウマップの分割に対 する色が格納される  

#### そのほかパス設定関連
cascadeBlendThresholdはカスケードするときのカスケード間の境界部分が現れないように、分割をお互いオーバーラップさせる用の変数  
このオーバーラップした部分はシェーダーによって2つの分割マップのブレンドを計算する  

#### そのほか実装補足
mDepthPass（つまりDepthPass.slangによる描画）は現在は機能していないっぽい  
作るだけ作っているが、実行はされていない（コメントアウトされている）  
故にレンダーグラフのどっかにDepthを作るパスが必要  

calcPssmPartitionEnd()の計算は[GPU Gems 3 Chapter 10. Parallel-Split Shadow Maps on Programmable GPUs](https://developer.nvidia.com/gpugems/gpugems3/part-ii-light-and-shadows/chapter-10-parallel-split-shadow-maps-programmable-gpus)の10.2.1 Step 1: Splitting the View Frustumの部分  

## DebugPassプロジェクト
様々なデバッグ用パスをまとめるプロジェクト
DebugPass.cppはそれらのパス作成をdllにするための処理のみ  

Mogwaiで使えるパスは以下となっている
- ColorMapPass : インプットテクスチャーのRGBAの内の一つにColorMap.slangの指定カラーマップ処理を適用したものを出力
- InvalidPixelDetectionPass : インプットテクスチャーがnan（赤） or inf（緑）かどうかチェックする用。floatテクスチャー専用
- SplitScreenPass : 2つ画像を渡し、線を境に2つの画像を別々に表示（どちらもピクセル位置不変で表示）
- SideBySidePass : 2つ画像を渡し、線を境に2つの画像を別々に表示（どちらも左端（＋オフセット）から表示）

また、これらのパスの内、SideBySidePassとSplitScreenPassはComparisonPass クラスをベースクラスとしている  
- ComparisonPass : 2つのinput画像を（別々にサブクラスで指定したシェーダーによる処理を行ったあと）左右に表示するパス    

### ColorMapPass
インプットの画像のうちの1チャンネル（_CHANNEL）に対し、ColorMap.slangにあるカラーマップのうちの1つを_COLOR_MAPで指定して適用したものを出力するパス  

カラーマップの両端をテクスチャーのどの値に対応付けるか値範囲を指定でき、  
さらにmAutoRangeオンの場合は、テクスチャーの最小最大値を計算し、それを値範囲として指定される（ComputeParallelReduction（CSMとは微妙に違う）を使ってGPU処理で行われFenceが発生することに注意）  
ただし、指定した範囲の最小（最大）の方が小さ（大き）ければそちらが優先される  

in : texture2D, ShaderResource  
out : texture2D, RenderTarget  
inのサイズとフォーマットはほぼ任意  

### InvalidPixelDetectionPass
インプットのテクスチャーのinfとnan検出用パス  
赤がnan, 緑がinf, 黒が正常  

インプットテクスチャーはTexture2D<float4>でシェーダーに渡して、floatとして判定するので、intテクスチャーとかは多分意味なし  

### ComparisonPass 
SplitScreenPassとSideBySidePassのアブストラクト的なクラス  
共通処理部分のみ実装されており、直接使うようなものではない  
2つの画像を指定したgSplitLocationを境界として別々に描画するパスとなっている  
つまり、シェーダー内で2つの画像を合成するとかComparison的な数値を出すとかそういうものではなく、目視でComparisonしやすくするためのものである  

in : leftInput、Texture2D、左側（mSwapSidesオンの場合は右）に描画するテクスチャー  
in : rightInput、Texture2D、右側（mSwapSidesオンの場合は左）に描画するテクスチャー  
out : texture2D、input2つの画像を左右に描画したテクスチャー

#### Comparison.ps.slang
これもまたサブクラスで使うシェーダーのインターフェースと共通処理部分となっており、直接使うものではない  
ただし、サブクラスで使われるほぼすべての処理が実装されており、サブクラスのシェーダーの方ではICalcPixelColorの実装と、ここで実装されているcompare()の呼び出しだけとなっている  

input2つの画像による任意の色計算処理ICalcPixelColorのみがインターフェースとなっており、各々の画像のpixelPosに対応する色になんらかの処理をしたものを返す  
これをcompare()関数で呼び出して各々の画像、gSplitLocationを境界として描画される  
（分割箇所では分割を示すための線の描画処理も行われている。gDividerColorとgDividerSizeで色とサイズ指定  
あと、gDrawArrowsがオンの場合、マウス位置の場所にgArrowTexを描画する機能もある）  

### SplitScreenPass
ComparisonPass のサブクラス  
pixelPosの色 = 左右どちらかのpixelPosの色を表示し、  
2つの画像の境界線をマウス左クリックでつかんでx軸移動できるようにしたパス  
（例えば、レイトレのハイサンプルローサンプル比較とかでよくあるやつ）

mDrawArrowsで境界線を動かしている間矢印を表示できたりする  

### SideBySidePass
ComparisonPass のサブクラス  
左右の画像どちらも左端（+gLeftBound）から表示するため用のパス  

こちらは境界線をマウス移動とかはできないが、  
gLeftBoundの操作を行えるようになっている  


## DepthPassプロジェクト
### DepthPass
深度バッファーのみ出力するパス  
シーンのメインカメラの深度が出力される  
シーンの情報だけあればいいので入力は無し  

シェーダーはアルファテストをするためにShading.slangのprepareShadingData(()が呼ばれている  
故にちょっとはコストがかかっている  

## ErrorMeasurePassプロジェクト
### ErrorMeasurePass
2つの画像間の[平均絶対誤差(MAEまたの名をL1)と平均二乗誤差(MSEまたの名をL2)](https://mathwords.net/rmsemae#MSEMean_Squared_Error)を計算するパス（RGBのみ、Aは計算されず出力は0固定）  

また、ピクセル全体の平均誤差が（ComputeParallelReductionのGPUによる総和によって）計算され、設定していればこれを外部ファイルに保存できる  
また、mReportRunningErrorがオンならこの誤差のフレーム推移として[指数移動平均（EMA）](https://ja.wikipedia.org/wiki/%E7%A7%BB%E5%8B%95%E5%B9%B3%E5%9D%87#%E6%8C%87%E6%95%B0%E7%A7%BB%E5%8B%95%E5%B9%B3%E5%9D%87)も計算されUIに表示される  

in : Source、比較画像その1、こちらは必ずセット  
in : Reference、比較画像その2、代わりにファイルから読み込むこともできる  
in : WorldPosition、ピクセル場所のワールド位置をテクスチャーにセットしたもの。これがセットされた場合w = 0だとその場所は0になる。セットしなくてもいい  
out : RGBA32Float、パスの設定に対応する誤差とか計算した画像を出力する。Referenceが無い場合はSourceをそのまま出力  

また、キーボードO or Shift + Oで出力をSource、Reference、誤差と切り替えできる

## ForwardLightingPassプロジェクト
### ForwardLightingPass
BRDF描画パス  
BRDF描画の他にノーマルとモーションベクターも計算して出力する  

in : visibilityBuffer、CSMパスで作成されるやつ  
in : depth、DepthPassで作成されるやつ、mUsePreGenDepthオンの時のみ  
out : color、RGBA32Float  
out : motionVecs、RG16Float  
out : normals、RGBA8Unorm  

#### シェーダー補足
Raster.slangにあるstruct VSOutのINTERPOLATION_MODEとnointerpolationは以下参照  
[Interpolation Modifiers Introduced in Shader Model 4](https://docs.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-struct#interpolation-modifiers-introduced-in-shader-model-4)  
Vertex shader outputs that are used for pixel shader inputs are linearly interpolated to get per-pixel values during rasterization. To set the method of interpolation, use any of the following values, which are supported in [shader model 4](https://docs.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-sm4) or later. The modifier is ignored on any vertex shader output that is not used as a pixel shader input.  

mEnableSuperSamplingによるINTERPOLATION_MODE sampleは、ピクセルシェーダーに値を渡すときpixel中心補間（linear）ではなくサンプルポイント補間での値となる   

## GBufferプロジェクト
（GBufferBaseをベースクラスとする）GBuffer（基本的なGBufferが定義されている。出力処理はサブクラスにお任せ）があり、これをベースクラスとする

- GBufferRaster : ラスタライズで計算した汎用GBuffer。GBufferクラスのデータ+エクストラデータをラスタライズによって出力する。ラスタライズだけでなくレイトレ用のデータも入っている汎用GBuffer    
- GBufferRT : レイトレで計算した汎用GBuffer。GBufferクラスのデータ+エクストラデータ（こちらはGBufferRaster とは微妙に違う）をレイトレによって出力する。ラスタライズだけでなくレイトレ用のデータも入っている。1サンプルしかとらないので、DOFがある場合はノイズあり    

と。GBufferBaseをベースクラスとする
- VBufferRaster : ラスタライズで計算したレイトレ用GBuffer。深度とScene/HitInfo.slangのHitInfoのみ（どちらもGBufferRasterで出力されている） 
- VBufferRT : レイトレで計算したレイトレ用GBuffer。Scene/HitInfo.slangのHitInfoのみ（GBufferRTでも出力されている）。timeバッファーもあるが、これはこのパスのプロファイリング用  

の4つがMogwaiのパスとして実装されている  


### GBufferBase
全てのGBuffer系クラスのルートクラス  
主にサブクラス生成のPythonへのバインドと共通の変数定義、  
そして、CPUSampleGenerator系クラスの生成と設定（Center設定の場合は生成無し）によるカメラのジッター設定を行う  

サブクラスにもmpSampleGeneratorがあるが、こっちはCPUSampleGeneratorとは違うので注意  
このクラスのmpSampleGeneratorはカメラのジッター以外では利用されない  

### GBuffer
GBufferRasterとGBufferRTのベースクラス  
カリング設定と、以下のレンダーターゲット情報を宣言するだけのクラス（実際にレンダーターゲットを作り描画処理部分はこれのサブクラスが行う）  

    struct ChannelDesc
    {
        std::string name;       ///< Render pass I/O pin name.
        std::string texname;    ///< Name of corresponding resource in the shader, or empty if it's not a shader variable.
        std::string desc;       ///< Human-readable description of the data.
        bool optional = false;  ///< Set to true if the resource is optional.
        ResourceFormat format = ResourceFormat::RGBA32Float;
    };
    const ChannelList GBuffer::kGBufferChannels =
    {
        { "posW",           "gPosW",            "world space position",         true, ResourceFormat::RGBA32Float },
        { "normW",          "gNormW",           "world space normal",           true, ResourceFormat::RGBA32Float },
        { "tangentW",       "gTangentW",        "world space tangent",          true, ResourceFormat::RGBA32Float },
        { "texC",           "gTexC",            "texture coordinates",          true, ResourceFormat::RGBA32Float },
        { "diffuseOpacity", "gDiffuseOpacity",  "diffuse color and opacity",    true, ResourceFormat::RGBA32Float },
        { "specRough",      "gSpecRough",       "specular color and roughness", true, ResourceFormat::RGBA32Float },
        { "emissive",       "gEmissive",        "emissive color",               true, ResourceFormat::RGBA32Float },
        { "matlExtra",      "gMatlExtra",       "additional material data (IoR, doubleSided, specularTransmission, metallic)",     true, ResourceFormat::RGBA32Float },
    };

これらの値を計算するシェーダーがGBufferHelpers.slangとなっている  

また、GBufferParams.slangのGBufferParams作成と管理も担当している  

### GBufferRaster
    This pass renders a fixed set of G-buffer channels using rasterization.

GBufferのサブクラスで、GBufferのkGBufferChannelsに対応するレンダーターゲットに加えて、以下のRWTexture2DのGBufferRaster.3d.slanghによる描画を行い出力するパス  

    // Additional output channels.
    // TODO: Some are RG32 floats now. I'm sure that all of these could be fp16.
    const ChannelList kGBufferExtraChannels =
    {
        { "vbuffer",          "gVBuffer",            "Visibility buffer (CSMのやつではなく、DXRのHitInfoバッファー)",                true, ResourceFormat::RG32Uint    },
        { "mvec",             "gMotionVectors",      "Motion vectors",                   true /* optional */, ResourceFormat::RG32Float   },
        { "faceNormalW",      "gFaceNormalW",        "Face normal in world space",       true, ResourceFormat::RGBA32Float },
        { "pnFwidth",         "gPosNormalFwidth",    "position and normal filter width ( length(abs(ddx(x)) + abs(ddy(x)) )", true, ResourceFormat::RG32Float   },
        { "linearZ",          "gLinearZAndDeriv",    "linear z (and derivative)",        true, ResourceFormat::RG32Float   },
        { "surfSpreadAngle",  "gSurfaceSpreadAngle", "surface spread angle (Ray Coneによるtexlod用))",    true, ResourceFormat::R16Float    },
    };
    
また、このクラス内でmpDepthPrePassGraph（DepthPassのみのレンダーグラフ）の作成と実行も行われ、Depthの作成とrenderData["depth"]への格納が行われる  

kGBufferExtraChannelsの内容から分かるように、これから得られるGBufferはラスタライズだけでなくレイトレでも扱う前提で実装されている  

#### surfSpreadAngle補足
surfSpreadAngleはRay Coneで扱われ、  
Ray Coneはレイトレのtex LOD計算アルゴリズムである  
レイをコーンと考えて、距離と反射した面に応じてコーンを広げていき、そのコーンの低面積からLODを計算する  
参考文献 : [Texture Level of Detail Strategies for Real-Time Ray Tracing](https://www.ea.com/seed/news/texture-level-of-detail-strategies-for-real-time-ray-tracing)、[Rey Tracing Gems : CHAPTER 20（左とおなじやつ）](https://www.realtimerendering.com/raytracinggems/)  
surfSpreadAngleの計算関数computeScreenSpaceSurfaceSpreadAngle()は参考文献の式(32)を行っている  

### GBufferRT
    This pass renders a fixed set of G-buffer channels using ray tracing.
GBufferのサブクラスで、レイトレでGBufferを作成するパス    
DXRで書かれたGBufferRT.rt.slangを実行し、  
それによるカメラからのレイトレを行い、  
ヒット情報からGBufferのkGBufferChannelsと以下のkGBufferExtraChannelsを計算し、  
それらすべてをRWTexture2Dに格納して出力する  

    // Additional output channels.
    const ChannelList kGBufferExtraChannels =
    {
        { "vbuffer",        "gVBuffer",         "Visibility buffer"(CSMのやつではなく、DXRのHitInfoバッファー),                true /* optional */, ResourceFormat::RG32Uint    },
        { "mvec",           "gMotionVectors",   "Motion vectors",                   true /* optional */, ResourceFormat::RG32Float   },
        { "faceNormalW",    "gFaceNormalW",     "Face normal in world space",       true /* optional */, ResourceFormat::RGBA32Float },
        { "viewW",          "gViewW",           "View direction in world space"(各ピクセルごとのレイの方向。おそらくデバッグ用),    true /* optional */, ResourceFormat::RGBA32Float }, // TODO: Switch to packed 2x16-bit snorm format.
        { "time",           "gTime",            "Per-pixel execution time"(各ピクセルごとの実行処理時間。おそらくプロファイリング用),         true /* optional */, ResourceFormat::R32Uint     },
    };


LODの対応として、以下の設定ができる  
- UseMip0 : なにもしない
- RayDifferentials : レイディファレンシャルでLOD計算をする。ただしWarningで出るように、現段階では座標系の左右巻き変換においてうまくいかないかもとのこと
- RayCones : レイコーンでLOD計算をするのだが、GBufferRT.hように、実装されてない。シェーダーを見るにRayDifferentialsと同じ実装になっている

RayDifferentialsとRayConeはシェーダーにも書いてあるように、[Rey Tracing Gems : CHAPTER 20](https://www.realtimerendering.com/raytracinggems/)の計算が使われている  
参考文献によると、RayConeはRayDifferentialsに比べ絵のクオリティは微小に下がるが、ペイロードのサイズ、処理速度ともに優秀なので、もしRayConeが実装された場合はそちらを使用してもいいかもしれない（反射しないのでペイロードの恩恵は受けられないけれども）
RayCone実装するなら、反射しないので式(26)からLODを求め、直接LODを指定できる

    ShadingData prepareShadingData(VertexData v, uint materialID, MaterialData md, MaterialResources mr, float3 viewDir, float lod)

を呼び出せばいいんじゃないですかね知らんけど  


1ピクセル1サンプルだが、DOFじゃない場合はランダムサンプルじゃないので、GBufferRasterと大して変わらないかも。使いたいGBufferの違いくらい（GBufferRasterにくらべ足りない変数が多いので、多分ベンチマーク用な気がする）  
ただしDOFの場合は1サンプルしかなくノイズが発生するはずなので、フィルターパスやテンポラルパスがさらに必要かも  

### VBufferRaster
    This pass renders a visibility buffer using ray tracing...
と書いているが間違い。クソコピペ。正確には  

    This pass renders a visibility buffer using rasterization.
    The visibility buffer encodes the mesh instance ID and primitive index,
    as well as the barycentrics at the hit point.
visibility bufferはGBufferRaster、GBufferRTのvbufferと同じで、Scene.HitInfoシェーダーのHitInfo  
ラスタライズ処理によってこれを描画し出力する（あと深度も出力する）  
つまり、レイトレのためのGBuffer生成パス  
このように必要最低限のバッファーのみ出力するため、GBufferではなくGBufferBaseのサブクラスとなっている  
シェーダーはVBufferRaster.3d.slang  

GBufferRasterと違い、RasterizerStateがないが、その場合はGraphicsStateObject側でデフォルトのRasterizerStateが生成されているので問題なし（つまりデフォルトのRasterizerState設定が使われている）  

シェーダー処理では、Scene/HitInfo.slangで定義されているHitInfoに必要なデータをvsのアウトプットから取得し、それをHitoInfoに入れencodeしているだけ  
アルファテストもしている（オプション）  
シェーダーにもあるが、FalcorのDXRおよび通常DXRではtriangle stripsはサポートしておらず、triangle listsのみサポートしており、barycentricsにはtriangle lists前提での点BCに対応する値だけが入っていることに注意  

out : depth、D32Float
out : vbuffer、Scene.HitInfoシェーダーのHitInfoデータが格納されている、これを使うにはdecode()を呼ばないといけない、RenderTarget | UnorderedAccess、RG32Uint  

### VBufferRT
    This pass renders a visibility buffer using ray tracing.
    The visibility buffer encodes the mesh instance ID and primitive index,
    as well as the barycentrics at the hit point.
VBufferRaster（とGBufferRasterとGBufferRTのvbuffer）と同じvisibilityバッファーをレイトレで描画し出力するパス  
ただし深度は出力しない（visibilityから計算できるのでわざわざここで出力しない）  
こちらも必要最低限のバッファーのみ出力するため、GBufferではなくGBufferBaseのサブクラスとなっている  

これもGBufferRTと同じくDOF対応もしている  
サンプルはここでのみ使われ、SAMPLE_GENERATOR_DEFAULTを使っている  
シェーダーに渡すframeCountはこのサンプル用  

アルファテストもしている（オプション）  

シェーダーでは、DOfのあるなしにかかわらずRayGenで1サンプルのみレイを飛ばし、closesthitとmissでHitInfoに情報を入れるのみ  
アルファテストがオンの場合はanyhitでその処理を行っている  
要するに基本的な処理のみ  

GBufferRT1ピクセル1サンプルだが、DOFじゃない場合はランダムサンプルじゃないので、おそらくVBufferRasterと大差なし  
DOFありの場合は1サンプルしかなくノイズが発生するはずなので、フィルターパスやテンポラルパスがおそらく必要  

 out : vbuffer、Scene.HitInfoシェーダーのHitInfoデータが格納されている、これを使うにはdecode()を呼ばないといけない、UnorderedAccess、RG32Uint  
 out : time、各ピクセルの実行にかかった時間をそのまま格納、おそらくプロファイリング用、optional、R32Uint  

## ImageLoaderプロジェクト

### ImageLoader
パスに設定した画像ファイルを読み込み、  
ミップやsRGB設定などを適用し、RTVとして出力するパス  

なので、処理はテクスチャー読み込みとblitのみ  
解像度はdstに依存。おそらくデフォルトのFBO設定をそのまま出力  

out : dst、画像を出力、formatは他のパス（or デフォルト処理）でdstに設定したものをそのまま  

## MegakernelPathTracerプロジェクト
PathTracerを使ったレイトレパス  
レイトレアルゴリズム全部入り  
おそらく質と早さの両取りをしたい場合用（実際には設定によっては無駄になる処理も多数あると思われるので、これでどのアルゴリズムを選べばいいかを確認しそれ用のパスを新たに作った方がいい）  

PathTracerクラスの内容と、それ関連のシェーダーを使い、PathTracer.rt.slangを実行する  

PathTracer.cppや、PathTracer.rt.slangとPathTracer.slangで使われる関数は[レイトレ関連](https://github.com/tktkq9/falcor_memo/blob/master/flacor_memo/%E3%83%AC%E3%82%A4%E3%83%88%E3%83%AC%E9%96%A2%E9%80%A3.md)の方でまとめてある  

ちなみにMegaの名を冠しているが、デフォルトは1ピクセル1サンプルのみで、反射屈折回数は3（PathTracerParams.slangで初期化している値）  
デフォルトはそこまで重くなさそうだし、Megaではない  
MinimalPathTracerに比べて多機能というのを表現したい感じ？  

### MegakernelPathTracer.h, cpp
このクラス自体は、PathTracer.rt.slang用のRtProgramやRtProgramVarsなどを作成し、変数を割り当てるのと、継承元のPathTracerクラスの処理を活用したexcute()処理のみ  

ライトの構築と設定、各レイトレに関するアルゴリズムの設定、UI表示などは全部PathTracerクラスが担当する  

excute()については、  
最初にPathTracerクラスのbeginFrame()を呼び、  
MegakernelPathTracerで新たに定義する必要のあるdefineを行い、  
PathTracer.rt.slang用のシェーダー変数を割り当て、  
mpScene->raytrace()を行うだけ  
あとは最後にPathTracerクラスのbeginFrame()を呼ぶ  

入力はPathTracerクラスが担当し、出力がこちらで定義される  
out : gOutputColor、描画結果  
out : gOutputAlbedo、最初にヒットした面のdiffuse + specular or 背景色、何用？  
out : gOutputTime、各ピクセルのレイトレ完了までにかかった時間  

### PathTracer.rt.slang
ライトサンプルする際に必要なkRayTypeShadowのmiss、anyhitシェーダー（ライトの場所はサンプリングで決めて、その位置までぶつからないか確認するためだけのシェーダー）と、  
反射屈折した方向にレイを飛ばし、次のトライアングルを見つけるためにある、kRayTypeScatterのmiss、anyhit、closesthitシェーダー（こちらはヒットした情報HitInfoを格納する。ヒットしなかったらHitInfo::kInvalidIndexになる）と、  
raygenerationシェーダーを定義しているシェーダーファイル  
RtProgram作る用    

raygenerationシェーダーは  
まず、loadShadingData()によってG or VBufferからShadingData（トライアングルのマテリアル的な情報）とHitInfo（トライアングルの位置情報）を取得し、  
その情報をもとにPathData（パストレ中に共有されたり更新されたりする情報）を初期化  
これらの情報を使ってtracePath()をkSamplesPerPixel回実行し、その結果から色をサンプリングしていく  
このtracePath()の実装詳細がPathTracer.slangの方にまとまっている  
最後にoutputに対応する値を入れて終わり  

raygeneration以外は全部PathTracer.slangの方で実行される  

HitGroup内でのTraceRay()呼び出しによるリカーシブ処理はされておらず、かわりに反射屈折はtracePath()のforループで行っているのでkMaxRecursionDepth = 1で問題なし  

こんな感で、このシェーダーファイルはCPU側との橋渡し用となっている  

### PathTracer.slang
tracePath()とその内部で呼ばれている関数たち  
レイトレ（パストレ）アルゴリズム実装部分  
複数回反射屈折、ボリューム考慮、ライト多数、NEE、MIS、RayFootpring処理などなどといった、  
PathTracerParams構造体のレイトレアルゴリズム設定のだいたい（すべて？）に対応している  

tracePath()の流れとしては、  
for 反射屈折合計回数をkMaxBounces回するとして（forループ2回目から反射屈折1回目となる）、  
　　evalDirect()によってライトサンプリングとTraceRay()によるラディアンス加算（NEEする and ボリューム内でなければ）  
　　ロシアンルーレットで終了判定とスループット更新  
　　generateScatterRay()でBSDFサンプリングとかによる次の反射屈折方向とスループット更新、pdf設定、透過の場合はinteriorListも更新（pdfかスループットが0なら終了判定）   
　　反射屈折によるrayFootprint更新（depth == 0の場合はraygenerationシェーダーですでに計算しているのでスルー）  
　　透過の場合はレイオリジンをそれ用に更新  
　　kMaxBouncesかkMaxNonSpecularBouncesを超えてて、NEE and !MIS and isLightSamplableなら最後の反射によるライトヒットが終わっているのでここで終了（別にやってもいい気もするけど）  
　　kDisableCausticsですでにデフューズ反射していてスペキュラー反射が起こったら終了  
　　traceScatterRay()でTraceRay()で次のレイヒット点を見つける  
　　ヒットした場合はhandleHit()でPathDataとShadingDataを更新  
　　ミスの場合はhandleMiss()でPathDataのラディアンスに香料追加して終了  
終わり  
ここで呼ばれているPathTracer.slangの関数を以下にまとめる  

- evalDirect() : ライトを選んでシャドウレイとばしてラディアンスの加算  
まず、generateShadowRay()でライトサンプリングしてとばす位置と方向計算  
traceShadowRay()でその点のビジブル判定  
見えてたらPathDataのラディアンスにgenerateShadowRay()で計算してたラディアンスを加算  
  - traceShadowRay() : kRayTypeShadowのTraceRay()でビジブル判定するのみ  
- traceScatterRay() : kRayTypeScatterによるTraceRay()で次のトライアングル見つける or miss  
forループはinteriorList用、これをしないとプライオリティの低い（映さない）面でレイトレが止まってしまうのを防ぐ用（「InteriorListの使われ方について」で詳細説明している）  
- handleHit() : traceScatterRay(()で次のサーフェース点を見つけれた時のPathData  pathとShadingData  sdの更新処理  
基本的にはその点のsd取得、その点に至ったことによるpath.rayFootprintの更新（反射による角度の変更はここでは行わない）とpath.originとpath.length（lengthは距離ではなく、反射回数。変数名がおかしい気がする）の更新を行う  
その他処理として、kUseNestedDielectricsnの時はpath,interiorListをもとにスループットpath,thpのvolumeAbsorption処理と、  
ヒットしたサーフェイスがエミッシブな時かつ、エミッシブライトを使う設定になっている時のラディアンスpath.Lの更新も行う  
TODO : このエミッシブライトを使う設定の条件の一部に!kUseNEE || kUseMIS || !isLightSamplableがあるがなぜこれが必要なのか分からん  
あと、kDisableCausticsの時はShadingData sdのDiffuse処理をしている  
- handleMiss() : envMapSamplerを使って環境マップのラディアンスを取得してPathDataのラディアンスLに加算するだけ  

ちなみに、MIS部分の処理はevalDirect()、handleHit()、handleMiss()などのライトサンプリングでpath.Lに加算するときに計算されている  

#### InteriorListの使われ方について
traceScatterRay()を見た感じ、  
☆プライオリティが高いサーフェース内 -> プライオリティが低いサーフェースエンター  
の場合以外は次のサーフェースにヒットした時点でTraceRay()を終了している  
逆に☆の場合はInteriorListにその情報は入るものの、ヒットした際の反射屈折処理とそのマテリアルによるスループットの減衰は無視される（スループットはプライオリティの高いマテリアルのが適用される）  
このようなことから、コードこそ違えど、Ray Tracing Gems Chapter 11: Automatic Handling of Materials in Nested Volumes の実装となっているっぽい  

ちなみにこの実装だと内部にいるときの屈折だけでなく、反射もオッケーだと思われ  
（ちゃんとkUseLightsInVolumesじゃない場合!path.isInsideVolume()でライト計算しない処理している。逆にkUseLightsInVolumesの場合は、現状ライト計算する際のボリュームの減衰が考慮されていないので非奨励。それっぽいことがuseLightsInVolumesのコメントにも書いてる）  

このようなことから、球の内部に液体があるとかの場合、球の中が空洞になっていないと、中の液体は完全に無視されるといったことになるのでそこを気を付けたほうがいいかも  

#### USE_ENV_LIGHTとUSE_ENV_BACKGROUNDの違い
USE_ENV_LIGHTはレイトレでのUSE_ENV_BACKGROUNDのダイレクトライトサンプリングするかどうかのフラグ（ただしUSE_ENV_BACKGROUNDをオンにしてなくてもバックグラウンドライトの値で評価する）  
USE_ENV_BACKGROUNDはレイトレのサンプリング以外全般のバックグラウンドライトを使うかどうかフラグ  
なので、基本的にはどちらもオン or どちらもオフになっていると思われる  

## MinimalPathTracerプロジェクト

ヘッダーのコメントにも書いてある通り、  
様々なアルゴリズムを取り除いた単純な（機能がMinimalな）レイトレ用パス  
MegakernelPathTracerとは対照的なパス  
単純なレイトレなのでバイアスがかかっておらず、おそらく確認用に使われるであろうもの  
（ちなみに散乱方向のサンプリングはsample_cosine_hemisphere_concentric()でサンプルされているので微妙にバイアスを取り除けていないように見える。さらに透過屈折は考慮されていない）  
ただし、収束させるにはかなりの数のサンプルを取らないといけない  

inputの内容から、事前にGBufferRTパスを実行している前提のパス  

payloadがやたらサイズがでかいので遅いかもしれない  
参考 : [### DirectXの話 第166回 DXRのパフォーマンスの話](https://sites.google.com/site/monshonosuana/directxno-hanashi-1/directx-166)

### MinimalPathTracer.h, cpp
MinimalPathTracer.rt.slangを実行する用クラス  

このクラス（とMinimalPathTracer.rt.slang）で特徴的なものとして、  
- kMaxRecursionDepth = 2u : スキャッターTraceRay()内でダイレクトライトの計算をするためにシャドウTraceRay()するのでリカーシブは2  
ただし、反射処理する場合はペイロードに次のレイ情報を入れ、forループによるtraceScatterRay()でどうにかしようとしている  
- mComputeDirect ( = kComputeDirect) : オフにするとIndirectなものだけ計算される  
オフで無視されるのは、GBuffer点でのエミッシブとAnalyticなライト、次以降の反射点のエミッシブ、環境光  
つまり、次以降の反射点でのAnalyticなライトのみ計算される  
オンにするとdirectのみではなく、direct + indirect計算になる  

in : GBufferRTの大体のやつ  
out : gOutputColor、最終結果、ポスト処理とかしたい場合は他のパスのやつとか必要  

### MinimalPathTracer.rt.slang
基本Analyticなライト以外は重要度サンプリングしないレイトレ  
メッシュのエミッシブは適当にぶつかった点のもののみを加算し、環境光はscatterMiss()の時のみ  

上でもちょっと説明したが、コードの流れは、  
GBufferの点でのAnalyticなライト効果のサンプルevalDirectAnalytic()（すべてのライトの内一つ、それがエリアライトならその1点）とその点でのエミッシブを加算（kComputeDirectがオフならどちらも行わない）  
generateScatterRay()で反射方向を計算（cosサンプリング）  
for kMaxBounces
　　traceScatterRay()で反射処理用のTraceRay()  
　　scatterClosestHit()したらその点でのevalDirectAnalytic()、kComputeDirectがオンならメッシュ点のエミッシブも加算  
　　scatterMiss()したらkComputeDirectがオンなら環境光を加算、break  
　　次の反射方向をgenerateScatterRay()で計算し、ScatterRayDataに格納  
done
加算した色を出力して終わり  

あと、シャドウレイはevalDirectAnalytic()で使われるのでtraceScatterRay()の時用にリカーシブ設定が必要  

#### その他
traceShadowRay()でRAY_FLAG_ACCEPT_FIRST_HIT_AND_END_SEARCHが設定されているが、  
このフラグはIgnoreHit()が呼ばれた場合はちゃんとignoreして次のヒット確認に移る仕様になっているので、  
shadowAnyHit()のアルファテストはちゃんと働く  

## PassLibraryTemplateプロジェクト
Falcorのドキュメント[To create a Render Pass Project:](https://github.com/NVIDIAGameWorks/Falcor/blob/4.2-release/Docs/Getting-Started.md#to-create-a-render-pass-project)に書かれてある、  
新たなパスをSource/RenderPasses/make_new_pass_project.batで作るときにコピーされるプロジェクト  

make_new_pass_project.batで呼び出されるmake_new_pass_project.pyにコピー処理が書いてある　　  

中身は一通りの必要な関数が定義されているが、それらは何もしない関数となっている  

## PixelInspectorPassプロジェクト
GBuffer系列の共通部分 + tone mapping前後の色を入力として、  
その画像をマウスで選択した場所の情報をimguiに表示するためのパスプロジェクト  
PixelInspectorData.slangのデータを区分けしてimguiに表示する  
Geometry、Material、ShadingDataとかレイトレ用のVisibilityデータとかがある  
デバッグ用  

似たようなクラスとしてPixelDebug.cppがあるが、  
あっちはパスではなく任意のパスの付属品みたいなものであり、さらに機能も簡易版みたいなものなので、  
より詳しく見たい場合はこちらを使う感じ？  

### PixelInspectorPass.h, cpp
PixelInspector.cs.slangのComputeProgramやVarsなどを作成し、  
インプットに設定したテクスチャーとマウスで選択している場所（テクスチャーの解像度に合わせてスケールされたtex座標）をそのシェーダーに設定し実行する  
そして、その実行結果PixelDataを読み戻し、imguiにその情報を表示するためのコンピュートパス   
（読み戻ししているのでmpPixelDataBuffer->map(Buffer::MapType::Read)でflush()が呼ばれることに注意）  

全てのinputにテクスチャーを設定する必要はなく、設定されていない部分は無効な値（だいたいは0）が自動的に設定され、  
CPU側ではmAvailableInputsやmIsInputInBoundsでimguiの表示非表示を制御する  

imguiによる確認が目的のパスなので入力のみ  
input : GBuffer系列の共通部分 + tone mapping前後の色


### PixelInspectorData.slang
imguiに表示するための情報であるPixelData構造体の定義用ファイル  
CPU、GPU両対応（HostDeviceShared.slanghのやつ）  

meshInstanceIDとtriangleIndexの初期値はkInvalidIndex = 0xffffffff  
その他の初期値は0に設定される  

### PixelInspector.cs.slang
マウスで選択している場所のインプットテクスチャー値、マテリアル値、ShadingDataの値をPixelDataにそのまま設定するだけ  

コンピュートシェーダーだがこのような処理なので1スレッドで1回処理のみ  

## SkyBoxプロジェクト
スカイボックス表示用パス  
cube.objがある前提  

シェーダーとクラスの処理を見る感じ、  
cube.objをフロントカルで描画し、そのcube.objのローカル位置 = スカイボックスへの方向 = テクスチャーの位置として処理している  
そのためcube.obはシーン全体を覆う、大きい球体である想定の実装っぽい  

TextureCubeと_SPHERICAL_MAPのどちらも対応している  

### SkyBox.h, cpp
cube.objを読み込み、SkyBox.slangのGraphicsProgramを作成し、  
そのシェーダーを使ってcube.objを描画するためのパス  
cube.objが元のシーンを覆うようになっている想定っぽいのでフロントカル設定されている  
インプットによるdepthでバックグラウンドが見えているとこだけ描画する  

in : depth、合成判定用  
out : depth、inputに対しcube.objも描画した後のdepth  
out : target、cube.objによるSkyBoxの描画結果、depth通過した部分だけ描画されている  

### SkyBox.slang
cube.objによってスカイボックスを描画するためのシェーダー  
球体を描画しつつ、その球体のローカルポジション = スカイボックスへの方向を、  
_SPHERICAL_MAPへのuv変換 or TextureCubeをサンプルするための方向と扱うことによって、  
スカイボックス描画を行っている  

## SSAOプロジェクト
インプットのdepthやnormalテクスチャーからAOマップをSSAOによって計算し、  
（そして設定によってはブラー処理をし）  
インプットcolorInにこのAOマップを蒸散しcolorOutに出力するパス  

内部でAOを作るパス、ブラーをかけるグラフ（複数のパス）、AOマップを合成するパスといったように、  
複数のパスやグラフが実行される（ただしCPUへの読み戻しは行わないので遅くなるわけではない）  

AOを計算する際の位置サンプリングとして、SampleDistributionのEnumで定義されているRandom、UniformHammersley、CosineHammersleyがあり、  
RandomはただのlinearRandだが、  
UniformHammersleyとCosineHammersleyは、Hammersleyサンプリングによって得られた2Dまでのサンプルに対し、  
Uniform[13.6.1 Uniformly Sampling a Hemisphere](http://www.pbr-book.org/3ed-2018/Monte_Carlo_Integration/2D_Sampling_with_Multidimensional_Transformations.html#UniformlySamplingaHemisphere)、  
または、Cosine[13.6.2 Sampling a Unit Disk](http://www.pbr-book.org/3ed-2018/Monte_Carlo_Integration/2D_Sampling_with_Multidimensional_Transformations.html#SamplingaUnitDisk)  
にサンプリングされるといった違いとなっている  

Hammersleyサンプリング自体はlow discrepancyなサンプリングで、ざっくりいうとHaltonサンプリングの違うバージョン    
[7.4.1 Hammersley and Halton Sequences](http://www.pbr-book.org/3ed-2018/Sampling_and_Reconstruction/The_Halton_Sampler.html#HammersleyandHaltonSequences)や、[## 20.3 Quasirandom Low-Discrepancy Sequences](https://developer.nvidia.com/gpugems/gpugems3/part-iii-rendering/chapter-20-gpu-based-importance-sampling)参照  


### SSAO.h, cpp
内部でAOを作るパス（SSAO.ps.slang）、ブラーをかけるグラフ（複数のパス）、AOマップを合成するパス（ApplyAO.ps.slang）を作成と設定をし、順次実行していくためのクラス  

SSAO.ps.slangのため、SampleDistributionによるランダム配列とノイズテクスチャーの生成も行われる  
SampleDistributionの方ではAOサンプル回数分だけSampleDistributionによるランダム値の計算（さらに、  
        float dist = (float)i / (float)mData.kernelSize;
        dist = glm::mix(0.1f, 1.0f, dist * dist);
        mData.sampleKernel[i] *= dist;

による距離変化もかける）  
ノイズテクスチャーについては単純な線形ランダムの値をテクスチャーに入れただけのものを作成する  


実行の流れとしては、  
1. 内部でAOを作るパスの実行  
2. mApplyBlurがオンなら作ったAOマップに対しGaussianBlurクラスによるブラー処理適用  
3. AOマップを合成するパスを実行し、AOマップをcolorInに乗算したものをcolorOutに出力  

となっている  

in : colorIn、ApplyAO.ps.slangでAOマップを適用する対象  
out : colorOut、ApplyAO.ps.slangでAOマップを適用したもの  
in : depth : SSAO.ps.slangでワールド位置の計算用＋AO計算のための遮蔽判定用  
in : normals、SSAO.ps.slangでノイズテクスチャーによる回転を求める際に使用  
internal : AoMap、このパス内で計算しているAOマップ、これを蒸散したものを出力するのでこれは出力しない  

### SSAO.ps.slang
SSAOを実行し、AOマップを作るためのシェーダー  
画像処理なのでピクセルシェーダーのみ  

ノイズテクスチャーでピクセルごとの法線軸に対しランダムな回転を決定し、  
SampleDistributionの設定により設定され、CPU側で事前に計算されたランダム値（Random以外ならサンプル値）のレイベクトルを取得  
そのベクトルにノイズテクスチャーの回転をかけ、そのピクセルのワールド位置からベクトル + radiusの距離レイを飛ばした先のdepthを計算    
それと元画像のdepthを比較し、遮蔽されているかどうか計算  
これをkernelSize回行い、AOを求める  

このようにランダム値によってAOの計算を行うのでブラーが必要になる可能性がある  

### SSAOData.slang
SSAO.ps.slangで使われるAOマップ作製の設定をまとめた構造体  
CPU側で設定され、SSAO.ps.slangのStaticCB.gDataに割り当てられる  

### ApplyAO.ps.slang
SSAO.ps.slangで作った（そして設定されていればブラーもかけられた）AOマップをcolorInに乗算するためのシェーダー  
画像処理するだけなのでピクセルシェーダーのみ  


## SVGFPassプロジェクト
[Spatiotemporal Variance-Guided Filtering: Real-Time Reconstruction for Path-Traced Global Illumination](https://research.nvidia.com/publication/2017-07_Spatiotemporal-Variance-Guided-Filtering%3A)  
の実装、レイトレ用  
レイトレのノイズを減らすフィルターパス  
論文に書いてあるようにカラーからilluminationを抽出し、それにフィルターをかけて、元に戻すという処理を行っている  

linearZを使っているので、それを出力しているGBufferRasterがある前提のパス  
ちなみに、GBufferRTの方はlinearZがないので使えない。このことから多分GBufferRTは実用ではなくお試し実装しただけのやつっぽい  

だいたいは論文の実装のままだが、  
σ_lのデフォルト値、alphaの計算方法とそのデフォルト値が違うのと、  
mesh IDによる前フレームとのピクセル判定も行っているが、これだけやっていないという違いはある  
（ほかにもあるかも）  

### SVGFPass.h, cpp
SVGF用のテクスチャー作成やシェーダーのためのFullScreenPass作成と実行、前フレームのテクスチャーへの割り当てと管理を行う  

以下に、複数のシェーダー実行の流れをまとめる  
1. mpPackLinearZAndNormalによりSVGFPackLinearZAndNormal.ps.slangを実行し、  
gLinearZAndNormalを作成してSVGFReproject.ps.slangに渡す  
2. mpReprojectionによりSVGFReproject.ps.slangを実行し、gColorとgEmissionとgAlbedoからilluminationとそのvarianceを計算、  
gMotionとgLinearZAndNormalと前フレームのgLinearZAndNormalで以前と現在のピクセルが同じか判定し、  
テンポラル蓄積された（同じじゃなかったら蓄積無しの現在の）gIllumination（a値がvariance）とgHistoryLengthとgMoments（illuminationのmoment、SVGFFilterMoments.ps.slang用）を出力  
3. mpFilterMomentsによりSVGFFilterMoments.ps.slangを実行し、  
蓄積フレームが少ないところは4.2 Variance estimationの後半に書いてある、現在の計算結果からluminanceとvarianceの計算を行いgIlluminationを更新  
4フレーム以降の場合はなにもせず、gIlluminationをそのまま出力  
4. mFilterIterations回mpAtrousによりVGFAtrous.ps.slangを実行し4.3 Edge-avoiding a-trous wavelet transformを行う  
2つのバッファーを交換しながら、gStepSizeを増やしてVGFAtrous.ps.slangを繰り返すことによってgIlluminationのエッジ検出フィルターを行う  
論文では5回やっているが、こちらのデフォルト値は4回となっている  
論文に書いてあるように、イテレーション1回目の出力をgPrevIllumに割り当てている  
5. mpFinalModulateによりSVGFFinalModulate.ps.slangを実行、  
SVGFReproject.ps.slangのillumination計算の逆変換を行いgColorを出力、pOutputTextureにそれをコピー  
あと、pPrevLinearZAndNormalTextureとmpPrevReprojFbo（gPrevMomentsとgPrevHistoryLength）を更新  

in : Albedo、illumination計算用その1  
in : Color、illumination計算用その2   
in : Emission、illumination計算用その3  
in : WorldPosition、実は使われてないのでいらない  
in : WorldNormal、w_nの計算や、前フレームと同じ状態かの判定に使用される  
in : PositionNormalFwidth、前フレームと同じ状態かのnormal判定で使用される（これで割ってノーマルの差異をスケールする）  
in : LinearZ、linear depthとその微分値、w_zの計算や、前フレームと同じ状態かの判定に使用される  
in : MotionVec、前フレームのピクセル値を得るため用  
internal : Previous Linear Z and Packed Normal、前フレームのLinearZとWorldNormalをパックしたもの  
internal : Previous Lighting、使われていない、なにこれ  
internal : Previous Moments、SVGFFilterMoments.ps.slangで使う蓄積フレームが足りてないときのvariance計算に使われるもの  
out : Filtered image、フィルターかけた後のColor  

#### その他
mVarainceEpsilonは使われてないっぽい  

σ_zがないのは論文奨励値の1で固定されているから  

### SVGFPackLinearZAndNormal.ps.slang
linearZテクスチャーとノーマルテクスチャーを一つのテクスチャーにパックしたものを出力するためのシェーダー  
出力されたものはSVGFReproject.ps.slangでgLinearZAndNormalとして扱われる  

出力はxyがlinearZ、zwがパックされたノーマルとなっている  
ちなみにこのlinearZは、GBufferRasterに書いてあるように、xがlinear zで、yはその微分値となっている  

このように、このシェーダーは直接アルゴリズムと関係ある処理ではなく、テクスチャー減らしのための最適化用シェーダー  

### SVGFReproject.ps.slang
illuminationを抽出し、以前フレームでの情報をもとにillumination（とmomentと蓄積カウント）の蓄積処理をするシェーダー  
4.1 Temporal filteringの部分、ただし論文に書いてあるmesh IDによる判定は行っていない  

まずilluminationは (gColor - gEmission) / gAlbedo によって抽出される  
TODO : この計算の根拠調査、素材の色 * そこにあたる光の強さが描画結果になると思われるので多分そんな感じ  

次にgMotionをもとに、現在のピクセルに対応する以前のピクセルが同じ場所かをgPrevLinearZAndNormalによるdepthとnormalからisReprjValid()関数で判定し、  
変化がない場合はgPrevIllum、gPrevMoments、gPrevHistoryLengthのピクセルにより以前のデータの読み込みを試みる  
読み込めた場合は、  
illuminationをalpha = max(gAlpha, 1.0 / historyLength) 、  
moments（4フレーム未満の時にVGFFilterMoments.ps.slangでvarianceを計算するためのμ）をalpha = max(gMomentsAlpha, 1.0 / historyLength) で  
exponential moving averageによって蓄積する  
（ない場合はOutHistoryLengthをリセットし現在計算されたものだけを出力）  
ちなみに論文ではlpha = 0.2が適正となっているが、こちらではalphaの計算が特殊なうえに、gAlphaの方は0.05で設定されてある  

以前のデータの読み込みについての詳細は、4.1 Temporal filteringの最後の段落に書いてあるように、  
以前のデータの取得にはサンプリング処理が行われ、  
中心点から半ピクセルずらした2*2 tap bilinearによるサンプリングを試み、  
もし（isReprjValid()でない、または重みが小さすぎる理由で）ピクセルが無い場合は中心点を含む3*3ピクセルのうちisReprjValid()を満たすピクセルによる平均を求める  
これも通らなかった場合は蓄積リセットとなる  

### SVGFFilterMoments.ps.slang
4.2 Variance estimationの後半に書いてある、蓄積フレームが4未満の場合にluminanceとそのvarianceに7×7 bilateral filterをするためのシェーダー  
  
テンポラルな蓄積が4フレーム未満の場合は、luminanceとそのvarianceはSVGFReproject.ps.slangで蓄積するだけではなく、  
SVGFReproject.ps.slangから得られた出力をもとに式（3）w_z、式（4）w_n、式（5）w_l（ただし、w_lの分母はσ_lのみとなっている）による重みを計算し、  
その重みと現在のluminance（moment）で7×7サンプリングし、新しいluminance（variance）を計算して返す  
また、varianceは 4 / 蓄積frame数 の補正をかける  

4フレーム以降の場合はSVGFReproject.ps.slangのluminanceをそのまま返す  

### SVGFAtrous.ps.slang
gIlluminationに対するエッジ検出フィルター処理  
つまり、4.4 Edge-stopping functionsによる重み計算によって、4.3 Edge-avoiding a-trous wavelet transformによるIllumination計算を行うためのシェーダー    
これ自体はa-trous wavelet transform1回分の処理となっており、gStepSizeを変えつつ複数回このシェーダーを実行することによってイテレーションを回す  

gLinearZAndNormalとgIlluminationとgPhiNormal（σ_n、デフォルトは論文と同じ値）とgPhiColor（σ_l、デフォルトは論文（4）と違い10）のパラメーターにより、  
シェーダー内容自体は式（1- 5）をやって新しいgIllumination（Illuminationとそのvariance）を計算しているだけ  
（σ_zは論文では1が奨励されているので、もはや設定できないようになっている）  

computeVarianceCenter()はw_lの分母にあるg_3x3(Var)のためのガウシアンフィルター関数  

gHistoryLengthは使われてないし、gAlbedoもサイズ取得のみでしか使われておらず他のテクスチャーでそれやればよくね？って感じなのでこの2つは多分いらない  

### SVGFFinalModulate.ps.slang
SVGFReproject.ps.slangのillumination計算の逆処理によって、  
フィルターかけたあとのgIlluminationからgColorを計算し出力しているだけ  

### SVGFCommon.slang
上記5つのシェーダーで共通で使用されるなんやかんや  

特徴的なのはcomputeWeight()で、  
式（3）w_z、式（4）w_n、式（5）w_lを計算して返す関数  
この関数では計算されていないw_zとw_lの分母部分は、引数のphiDepthとphiIllumで事前に計算されている前提となっている  
（これは、この関数が使われているSVGFFilterMoments.ps.slangと、SVGFAtrous.ps.slangのイテレーションごとで計算が違うため）  

### その他
ちなみに発展形として時間変化も考慮したA-SVGF[https://shikihuiku.wordpress.com/2019/06/27/%E3%83%A1%E3%83%A2a-svgf%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6/]があるがこちらは未実装（？）  
ただし論文にもあるようにSVGFより重い  

## TemporalDelayPassプロジェクト
接続したインプットに対し、指定したフレーム数分前のインプット内容を出力するパス  
この処理のために、フレーム数分そのままcopyResource()でコピーしていくだけ  

### TemporalDelayPass.h, cpp
imguiで設定したmDelayフレーム数だけ、それ用のrenderDataを作り、  
execute()ではそのrenderDataをそのままcopyResource()でコピーしていくだけの処理  
シェーダーいらず  

in : src、コピー元  
internal : maxDelay-i、iは内部で保存されている指定以外のフレーム数  
out : maxDelay、mDelay前のsrc  


## ToneMapperプロジェクト
トーンマッピングパス  

imguiによりシャッタースピード or 露光、f値（mAutoExposureがtrueの場合はシェーダー側でLuminanceの平均を計算して自動調整）と  
色温度で画像の明るさ基準を設定できる  

また、トーンマップ関数はToneMapperOperatorというenumによって以下の5種類の中から選べる    
- Linear : そのまま出力
- Reinhard : Reinhard関数、color / (luminance + 1)
- ReinhardModified : [Extended Reinhard](https://64.github.io/tonemapping/#extended-reinhard)、color * (1 + luminance / whiteMaxLuminance^2 ) / ( 1 + luminance )のはずなんだが、  
( 1 + luminance )^2がかけられている。ミス？
- HejiHableAlu : John Hable's ALU approximation of Jim Heji's operator、リンク切れしているのでどんなのかなぞ、多分[Hable John Uncharted2 Hdr Lighting](https://www.slideshare.net/ozlael/hable-john-uncharted2-hdr-lighting)142ページあたり  
TODO : ちゃんと探す  
実装を見るにHableUc2の亜種  
- HableUc2 : アンチャーテッド2で使われているやつ、同じくコードに書いてある奴はリンク切れ  
参考文献は[Hable John Uncharted2 Hdr Lighting](https://www.slideshare.net/ozlael/hable-john-uncharted2-hdr-lighting)142ページあたり  
- Aces : [ACES Filmic Tone Mapping Curve](https://knarkowicz.wordpress.com/2016/01/06/aces-filmic-tone-mapping-curve/)の内容  

参考として、これらの説明とその結果は[[Tone Mapping](https://64.github.io/tonemapping/)](https://64.github.io/tonemapping/)の方にも大体まとまっている  
ただしアンチャーテッド2のパラメーター設定がちょっと違う  

### ToneMapper.h, cpp
上に書いたトーンマップ設定をimguiで設定できるようにし、  
Luminance.ps.slangとToneMapping.ps.slangシェーダーのFullScreenPassを作成、設定に合わせて変数計算と割り当てと実行を行うクラス  
ToneMapping.ps.slangがトーンマップ用で、Luminance.ps.slangがluminance平均計算用  

mAutoExposureがtrueの場合はLuminance.ps.slangのFullScreenPassが実行され、  
luminanceを出力した後mipmapを作成し、luminance平均を求める  
この値を使って、[トーンマップいろいろ](http://hikita12312.hatenablog.com/entry/2017/08/27/002859)とかに書いてある、y = f(k / bar{L}) x)の調整を行う（このシェーダーではk = 0.042）  

mAutoExposureがfalseの時はluminance平均による補正は行われず、  
かわりに、シャッタースピード or 露光、f値による計算が行われる  

これらの補正＋mWhitePointによる色温度補正が考慮され、  
色補正行列mColorTransformが計算されシェーダーで適用される（luminance平均は別処理で行われる）  

in : src、トーンマップする前  
out : dst、トーンマップした後  

### ToneMapperParams.slang
ToneMapperOperatorというenumの定義と、トーンマップシェーダーで使うパラメーターToneMapperParamsを定義しているだけ  
HostDeviceShared.slanghを使っているのでGPU、CPU双方でこれを共有している  

### Luminance.ps.slang
mAutoExposureがtrueの場合にのみ実行されるcolorをluminanceに変換するだけのシェーダー  
平均はこれを実行した後のmipmap作成によって行われる  

### ToneMapping.ps.slang
トーンマップ用シェーダー  

srcのcolorに対し  
設定されていたらluminance平均処理を行い、  
シャッタースピード or 露光、f値、色温度による補正行列colorTransformをcolorにかけ、  
設定されたToneMapperOperatorによる関数によってトーンマップされる  
_TONE_MAPPER_CLAMPが設定されていたらsaturate()も行われて終わり  

## Utilsプロジェクト
小道具パス集  
足し算掛け算パスとガウシアンブラーパスの2つだけ  

### Utils.cpp
2つのパスをpybind11する処理のみ  

### Composite.h, cppとシェーダー
入力A、Bを足し算 or 掛け算するだけのパス  
CompositeMode.slanghは足し算掛け算のどちらをするかのdefineを定義しているだけ  
Composite.cs.slangはその足し算掛け算するだけコンピュートシェーダー  
このシェーダーのComputePassを作って実行するだけ  
scaleA、scaleBによるA、Bそれぞれのスケールもできるようになっている  

### GaussianBlur.h, cppとシェーダー
ガウシアンブラーをかけるパス  
シェーダーは1Dにガウス重み足し合わせで、これに対する_HORIZONTAL_BLURと_VERTICAL_BLURのFullScreenPassそれぞれを作り、  
横方向にかけるパス実行 -> この結果に対し縦方向にかけるパス実行、といった感じで計算回数を節約している  

kernelWidthがn x nフィルターのnの部分、中心を必ず含まなければいけないので、自動的に奇数に設定される  
mSigmaがガウス分布関数にあるσ  
これらからガウス重みをCPU側で事前計算し、その配列をシェーダーに渡すといった感じ  

## WhittedRayTraceプロジェクト
MegakernelPathTracer、MinimalPathTracerに続き、3つ目のパストレーサーパス  
このパスもMinimalPathTracerに近いが、  
- 反射レイの方向がBSDFサンプルせず鏡面反射のみであるということと、  
- RayCone、RayDiffによるLOD処理をすることと、  
- evalDirectAnalytic()で複数のライトから1つサンプルするのではなく、すべてのライトに対して1回サンプルする  
- あとダイレクトとインダイレクトを分けることはｄけいない  

という点が異なる  
こんな感じでMinimalPathTracerと比較しようにもちょっと微妙な感じなので何用なのかちょっとわからん  

上の差異を除いてだいたいはMinimalPathTracerと同じで、  
RayCone、RayDiffの部分はRayFootprint.slangやMegakernelPathTracerの処理とだいたい同じ  

インプットはGBufferRaster、GBufferRTのどちらも対応している  
GBufferRasterにのみあるgSurfaceSpreadAngleはRayConeMode::RayTracingGems1の設定の時のみ使われるが、  
scatterClosestHit()の時にRayTracingGems1では追加するsurfaceSpreadAngleが0のままでいっているので多分このモード含め使用しなさそう  

このプロジェクトのファイルの説明はMinimalPathTracerとほとんど同じなので省略  


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTYyMTUyODk5LC00NTI0NjY2NDUsMTMxND
MxOTgwMiw1NjA0NTYxOTIsLTI4ODIxNjQ2NSwtNTE0OTQ0NjU4
LC0xMDE2NjY2MjgwLC0xMjg0NjM3MDQzLDEyMDIxNDcwODAsMT
I2Mjc4NTU2LC05MTgyMDI3NzAsLTE4MjMzMjE2NzEsMjAzMjI1
NzY2MCwxMDIxMTYwNTA2LDg0MzAwODcxMiwzODM4NTEzLDI3MD
k3MDI0NywtNDUyNjA4NzAsLTE2MzUyMzc1MDksLTIxMzkwNzcw
NjddfQ==
-->