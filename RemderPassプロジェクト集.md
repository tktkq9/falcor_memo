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

（GBufferBaseをベースクラスとする）GBufferをベースクラスとする
- GBufferRaster
- GBufferRT

GBufferBaseをベースクラスとする
- VBufferRaster
- VBufferRT
- 
の4つがMogwaiのパスとして実装されている  

### GBufferBase
全てのGBuffer系クラスのルートクラス  
主にサブクラス生成のPythonへのバインドと共通の変数定義、  
そして、CPUSampleGenerator系クラスの生成と設定（Center設定の場合は生成無し）によるカメラのジッター生成の設定を行う  

サブクラスにもmpSampleGeneratorがあるが、こっちはCPUSampleGeneratorとは違うので注意  
このクラスのmpSampleGeneratorはカメラのジッター生成以外では利用されない  

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

Rasterと書いているが、kGBufferExtraChannelsからも分かるように、これから得られるGBufferはラスタライズだけでなくレイトレも扱う前提で実装されている  

#### surfSpreadAngle補足
surfSpreadAngleはRay Coneで扱われ、  
Ray Coneはレイトレのtex LOD計算アルゴリズムである  
レイをコーンと考えて、距離と反射した面に応じてコーンを広げていき、そのコーンの低面積からLODを計算する  
参考文献 : [Texture Level of Detail Strategies for Real-Time Ray Tracing](https://www.ea.com/seed/news/texture-level-of-detail-strategies-for-real-time-ray-tracing)、[Rey Tracing Gems : CHAPTER 20（左とおなじやつ）](https://www.realtimerendering.com/raytracinggems/)  
surfSpreadAngleの計算関数computeScreenSpaceSurfaceSpreadAngle()は参考文献の式(32)を行っている  

### GBufferRT
GBufferのサブクラスで、  
DXRで書かれたGBufferRT.rt.slangを実行し、  
それによるカメラからのレイトレを行い、  
ヒット情報からGBufferのkGBufferChannelsと以下のkGBufferExtraChannelsを計算し、  
それらすべてをRWTexture2Dに格納して出力するパス  

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
- RayCones : レイコーンでLOD計算をする。GBufferRT.h曰く実装されてないらしいが、シェーダーを見るに多分実装されてる  

RayDifferentialsとRayConeはシェーダーにも書いてあるように、[Rey Tracing Gems : CHAPTER 20](https://www.realtimerendering.com/raytracinggems/)が使われている    
また、RayDifferentialsは[pbrt-v3 10 Texture](http://www.pbr-book.org/3ed-2018/Texture.html)でも詳しく説明されている  
参考文献によると、RayConeはRayDifferentialsに比べ絵のクオリティは微小に下がるが、ペイロードのサイズ、処理速度ともに優秀なので、反射を考慮するならRayConeのほうがいいが、  
このレンダリングパスは初期ヒットのみなので、シェーダーを見た感じでもRayDifferentials方が処理量が少ないので、RayDifferentialsを設定したほうがいいかも   

### VBufferRaster

### VBufferRT


 
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTIyMzMzNDU1OCwtODgzNzkzMTAsLTEzMD
k5MTU2NjEsNzY0NTgzMjM2LC0xNTI0ODA1NDAzLC0xNTUzODc5
MDA2LDY2OTE3MTA2Miw1ODc3MjYxNSwtNjIxNTk4ODk4LDExOT
IxNjIxMTQsNzMxNDkwODMyLDQ1ODk2NzAxNiwtMTMzNDQzNjE3
NSwxMzE5NTcwMDUsNjA1MjY0MzYzLDkzNjUzNjQ4LDc5MzE1ND
I2MSwtMTA3NjQyOTY4LC00Njk0Mjc0NjUsOTc4NjIxODVdfQ==

-->