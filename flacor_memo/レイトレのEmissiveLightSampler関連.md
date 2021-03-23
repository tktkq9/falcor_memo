# レイトレのEmissiveLightSampler関連
めっちゃ量あるのでこっちに分割  
メッシュライトのサンプラーと、そのメッシュライトの空間構造構築（LightBVHSamplerの場合）  
サンプラーはEmissiveUniformSamplerとLightBVHSamplerの2種類  
メッシュライトのみを集めたクラスであるLightCollectionを用いてサンプルする  

EmissiveUniformSamplerはLightCollectionから一様確率でトライアングルを選ぶだけ  

LightBVHSamplerは
Githubの[ray-tracing-gems/Ch_18_Importance_Sampling_of_Many_Lights_on_the_GPU/](https://github.com/Apress/ray-tracing-gems/tree/master/Ch_18_Importance_Sampling_of_Many_Lights_on_the_GPU)  
にも書かれているように、  
資料[Ray Tracing Gems Chapter 18 Importance Sampling of Many Lights
on the GPU](https://www.realtimerendering.com/raytracinggems/)  
の内容が実装されている  

一応LightCollectionの構築方法も18.4.1 LIGHT PREPROCESSINGにあり、おそらくこれが実装されている  

# EmissiveLightSampler
メッシュライト（要するに任意の形のエリアライト）のレイトレサンプラー  
例えばPathTracerHelpers.slangのsampleSceneLights()でライトサンプルするときに使う  

## EmissiveLightSampler.h、cpp
_EMISSIVE_LIGHT_SAMPLER_TYPEをaddDifine()するだけ  
これ自体はベースクラスであり、これを継承しているEmissiveUniformSamplerやLightBVHSamplerが本体  

サンプラーの種類を指定するUniform or LightBVHのEnumをMogwaiとかで扱えるようpybind11もしてる  

## EmissiveLightSampler.slang
EmissiveLightSampler::sampleLight()でサンプルし、  
方向と位置と法線やPDFや光の強さなどのレイトレ用情報をTriangleLightSampleに格納し渡す  

Uniform = EMISSIVE_LIGHT_SAMPLER_UNIFORMの場合はExperimental.Scene.Lights.EmissiveUniformSampler、  
LightBVH = EMISSIVE_LIGHT_SAMPLER_LIGHT_BVHの場合はExperimental.Scene.Lights.LightBVHSampler  
のサンプラー構造体をEmissiveLightSamplerにtypedefして統一的に使えるようにしているだけ  
本体はこれらのslang  

### EmissiveLightSamplerInterface.slang
EmissiveLightSamplerのインターフェースやその関数で使う構造体（関数無しメンバ変数のみ）の宣言がされている  

# EmissiveUniformSampler
EmissiveLightSamplerのUniform = EMISSIVE_LIGHT_SAMPLER_UNIFORMバージョン  

lightCollectionをシェーダー変数に設定し、  
その中のトライアングルの1点を一様サンプリングによって選択するサンプラー  

## EmissiveUniformSampler.h、cpp
EmissiveLightSamplerのタイプとしてUniformを設定して、  
Scene->getLightCollection(pRenderContext)を呼んでシェーダーにlightCollectionを作って設定してるだけ  

Optionsを設定する実装もあるが、現在は設定できるOptionはなにもなし  
TODOになってるのでおそらく今後実装される  

## EmissiveUniformSampler.slang
シーン中のlightCollectionに属するすべてのトライアングルから  
1Dサンプルを使って一様に選択  
そのトライアングルに対し2Dサンプルを使って一点をサンプルして返すサンプラー  

sampleLight) : 一様サンプリングして返す  
lightCollectionの中から一様にトライアングルを選び、  
sampleTriangle()でトライアングルの1点サンプル  
トライアングル選択で1Dサンプル1つ、1点選びは2Dサンプル1つ消費  

evalPdf() : トライアングル一様選択によるpdfとevalTrianglePdf()によるトライアングル1点サンプリングによるpdfの積を返す  

EmissiveLightSamplerHelpers.slangでも書いているが、  
トライアングル1点サンプリングのpdfに余計な係数がついていることに注意  

## EmissiveLightSamplerHelpers.slang
三角形のサンプルとpdf計算の関数がまとめられたやつ  
サンプルの取り方やpdfの計算は[PBRT 13.6.5 Sampling a Triangle](http://www.pbr-book.org/3ed-2018/Monte_Carlo_Integration/2D_Sampling_with_Multidimensional_Transformations.html#SamplingaTriangle) のやつ  
ただし、pdfはdistSqrと1/cosThetaの部分が余計についている  
この余分なのは正確にはpdfではなくRay Tracing Gems Chapter 18 式（12）の  
| n_{Y_k} dot ( - l ) | / | X - Y_k |^2  
の項に対応する  

# LightBVHSampler  
Githubの[ray-tracing-gems/Ch_18_Importance_Sampling_of_Many_Lights_on_the_GPU/](https://github.com/Apress/ray-tracing-gems/tree/master/Ch_18_Importance_Sampling_of_Many_Lights_on_the_GPU)  
にも書かれているように、  
資料[Ray Tracing Gems Chapter 18 Importance Sampling of Many Lights
on the GPU](https://www.realtimerendering.com/raytracinggems/)  
の内容が実装されている  

EmissiveUniformSamplerはlightCollectionをシェーダーに設定したが、  
こちらはそれだけではなく、_lightBVH構造体変数の値も設定する  
こちらでは、その_lightBVHによる重要度サンプリングによりトライアングルの1点を選択する  

## LightBVHSampler.h、cpp
18.4.3 LIGHT IMPORTANCE SAMPLINGの実装  
すでに出来上がっているLightBVHを使ってエミッシブメッシュのトライアングルサンプルをする設定などを管理するクラス  
Optionsでそのサンプル方法を設定でき、デフォルトではCh18の式（17）全部入り、リーフノードの複数トライアングルはUniformでサンプルとなっている（Ch18を見ると、これがクオリティと速さ両方を考慮したとき一番バランスが取れている設定っぽい）  
また。Optionの中にはビルドオプションLightBVHBuilder::Options buildOptionsも入っており、こちらはLightBVHの構築アルゴリズムとアップデートの設定になっている  
こちらのデフォルトはノードのトライアングルは10個、ビンニング分割数16、分割はxyz軸全考慮、SAOH（Ch18式（16））、ライトコーン使うといった設定になっている（これもCh18を見ると、アップデート速度とクオリティのバランスが一番いいっぽい設定になっている）  

LightBVH自体とそのrefitはLightBVHクラス、LightBVHの構築はLightBVHBuilderに任せてある  
これらのメンバ変数を持ち、これのコンストラクタやビルド命令などはこのクラスが行う  
このように、以下のLightBVH関連のクラスの大本となってもいる  

また、サンプルの処理としてはシェーダー変数_lightBVHを設定するのみ  
正確にはmpBVHのsetShaderData(()をここから呼び出し変数に設定している  

imguiはメンバ変数のUI表示+サンプラーのオプション設定  

LightBVHの更新は、  
ライトの位置とかが変更された時はallowRefittingがtrueの場合はrefit、そうでなければリビルドされ、  
また、UIでLightBVHの変更が必要なオプションが変わったときはリビルドされる  

### LightBVHSampler.slang
トライアングルの1点サンプリングはEmissiveUniformSamplerと同じだが、  
トライアングルの選択はLightBVHを使っていくサンプリング用シェーダー関数  
EmissiveUniformSamplerと同様に、（つまりIEmissiveLightSamplerと同様に）  
sampleLight()とevalPdf()の2つの関数とそれを計算するためのprivate関数群といった内容になっている  


sampleLight() : "18.4.3 LIGHT IMPORTANCE SAMPLING" によるサンプリング  
sampleLightViaBVH()によりLightBVHからトライアングルを選択し、  
sampleTriangle()によりそのトライアングルの面の内の1点をサンプリングし、そのTriangleLightSample lsを求める関数  
トライアングル選択で1Dサンプル1つ、トライアングルの1点選択で2Dサンプル1つを消費する  
EmissiveUniformSamplerと同じようにトライアングルの1点選択はEmissiveLightSamplerHelpers.slangの関数を使っているので、トライアングルのpdfは正確なpdfではなく、式（12）の余分な係数がついていることに注意  
- sampleLightViaBVH() : LightBVHからトライアングルを重要度サンプリングによって選択する関数  
traverseTree()によってリーフノードまでの重要度サンプリングとそのpdf計算、  
pickTriangle()によってリーフノードでのトライアングル選択の重要度サンプリングとそのpdf計算をし、  
そのトライアングルインデックスと2つのpdfの積を返す  
  - traverseTree() : sampleLightViaBVH()で呼ばれ、LightBVHに対しどのリーフノードを選ぶかを式（17）によって選ぶ関数  
式（17）はcomputeImportance()によって計算され、トップノードからリーフノードに達するか、インポータンス = 0になるまでサンプルuと式（18）のu再利用によってノードを選択し続ける  
リーフノードのノードインデックスと、そのリーフノード選択によるpdfも計算して返す  
  - computeImportance() : 式（17）を計算して返す関数  
_DISABLE_NODE_FLUX（fluxを使わないか）、_USE_BOUNDING_CONE（cosθ'_iを使うか）、_USE_LIGHTING_CONE（cosθ'使うか）の計算をそれぞれdefineに従って行う  
ここで呼ばれているboundCosineTerm()がθ'_i = max(0, θ_i - θ_u）によるcosθ'_i（と後で使うcosθ_u）を計算する関数  
_USE_LIGHTING_CONEの下にある処理がθ'_i = max(0, θ - θ_o - θ_u）によるをcosθ'計算している  
最後にこれら全部を合わせて式（17）の値を返す  
  - boundCosineTerm() : 式（17）、Figure 18-3.のθ_uを計算し、ついでにθ'_i = max(0, θ_i - θ_u）を計算し返す関数  
つまり_USE_BOUNDING_CONEでのNdotLを計算する関数
ここのθ_uの計算方法がSolidAngleBoundMethodによって変更される  
ちなみにAABB内にShading point（origin）が入っていた場合はθ_u = π  
    - Sphere : boundSphereSubtendedConeAngle()呼び出し  
  AABB半径を計算し、その角度を返す  
    - BoxToAverage : boundBoxSubtendedConeAngleAverage()呼び出し  
  originからAABB8点への正規化ベクトル8つを平均してコーン方向を計算し、  
  そのコーン方向とoriginからAABB8点へのベクトルとの角度の内最も大きいものを返す  
  ちなみにboundCosineTerm()ではorigin - centerでθ_iを計算しているので、コーン方向が一致していないことに気を付けるた方がいいかも  
    - BoxToCenter : boundBoxSubtendedConeAngleCenter()呼び出し  
  3つの方法の中で一番最小のθ_uを計算する  
  高速化のため計算がちょっと分かりづらいが、やっていることはoriginから最も近いAABBの4点の内、最も角度が大きいものを返す  
  - pickTriangle() : traverseTree()によって得たリーフノードインデックスからノードを取得し、その中のトライアングルを選択し、そのトライアングルインデックスとpdfを返す関数  
  _USE_UNIFORM_TRIANGLE_SAMPLINGの場合は等確率選択によってトライアングルを選ぶ  
  そうじゃない場合は、各々のトライアングルのpdfをcomputeTriangleImportance()によって取得し、cumulative distribution function (CDF) の方法を使ってトライアングル取得  
  参考文献 : [13.3 Sampling Random Variables](http://www.pbr-book.org/3ed-2018/Monte_Carlo_Integration/Sampling_Random_Variables.html#)
    - computeTriangleImportance() : shading pointのからトライアングルの各点までの方向とshading pointの法線の内積NdotLに対し、  
三角形までの最小距離で割ったものをpdfとして計算  
バックフェイスの場合はpdf = 0　　
    最小距離はcomputeSquaredMinDistanceToTriangle()によって取得（この関数の処理は「ゲームプログラミングのためのリアルタイム衝突判定」のp136の三角形と点の最小距離点導出が参考になるかも。TODO : 要確認）
todoコメントにもあるように本当は最小距離の点とshading pointのNdotLを計算したほうがいい（遅くならなければ）  

evalPdf() : TriangleHit情報からpdfだけ計算して返す関数  
トライアングル1点サンプリングのpdfはEmissiveUniformSamplerと同じだが（よって、余計な係数がついていることに注意）、  
トライアングル選択の方はLightBVHを使っているのでそれ用の計算が行われる  
つまり、evalBVHTraversalPdf()でノード選択のpdf（とリーフノードインデックス）が計算され、  
evalNodeSamplingPdf()でリーフノードのトライアングル選択pdfが計算される  
これらの積を返す  
ちなみにevalBVHTraversalPdf()のために、トライアングルインデックスではなく、それに対応する事前に格納したノードトラバース情報（_lightBVH.triangleBitmasks[]）であるbitmaskを用いる  
- evalBVHTraversalPdf() : bitmaskによってトラバースしつつ、  
sampleLightViaBVH()と同じようにcomputeImportance()でpdfを計算し、  
そのpdfとリーフノードインデックスを返す  
- evalNodeSamplingPdf() : pickTriangle()と同じように、  _USE_UNIFORM_TRIANGLE_SAMPLINGの場合は一様サンプリングのpdf、  
そうじゃない場合は各トライアングルのpdfをcomputeTriangleImportance()から計算し、そのCDFによるpdfを返す  

### LightBVHSamplerSharedDefinitions.slang
SolidAngleBoundMethodの定義のみ  
これはboundCosineTerm()でのみ使われる、ある点から見たAABBに対し、それを囲むコーンの角度（θ_u）を計算する、3種類の関数の内の1つを選ぶためのenum  


## LightBVHBuilder.h, cpp
18.4.2 ACCELERATION STRUCTUREの実装  
LightBVHをビルドするためのクラス  
build()に作りたいLightBVHインスタンスを渡してビルドして役目終了のクラス  

create()時点（コンストラクタ時点）でば何もしない  
LightBVHSamplerから渡されたLightBVHBuilderのOptionを設定するだけ  

renderUI()はLightBVHBuilderのOptionの設定のみ  

あと残り90％くらいは全部build()関数とその仲間たちのみ  
### build()周り
LightCollectionのトライアングルを、BuildingData dataに設定、  
分割方法としてSplitHeuristicFunctionをgetSplitFunction()から取得し  
この設定内容とSplitHeuristicFunctionをもとにbuildInternal()でnodesとtriangleIndicesとtriangleBitmasks作成  
そのあと出来たnodesに対しcomputeLightingConesInternal()でnodeそれぞれのcosConeAngleとconeDirectionを計算、設定する  
最後にLightBVHの後処理をして完了  

- buildInternal() : これを再帰してdata.nodesを作成する（リーフノード以外のcosConeAngleとconeDirectionはあとで計算するのでこの時点では0）
ついでにdata.trianglesDataもnth_element()で分割に合わせて再配置される    
nodesは親から始まり、2つの子ノードに再帰的に分割されていく。残りのトライアングルがmaxTriangleCountPerLeaf以下になったらリーフノードにして終わり  
  - リーフノード処理では、ノードにリーフの情報を入れ、  
 （ノード基本情報 + トライアングルの数 + このリーフに対するtriangleIndicesの最初の位置（オフセット）。また、リーフではこの時点でcomputeLightingCone()によりconeDirectionとcosConeAngleを計算する）  
triangleIndicesにそのリーフのトライアングルのtriangleIndexを全部入れ、  
triangleBitmasksにはtriangleIndexの場所にbitmask（depth0から順に、左=0 or 右 = 1のビットを詰めていったもの。つまりトップノードからリーフに到達するまでの情報）を設定する  
  - nodes配列はdepth0（トップ）ノード、depth1左ノード、depth2左ノード、...、depthN左ノード、depthN左ノードの左リーフノード、depthN左ノードの右リーフノード、depthN右ノード、...  
といったように配置されている  
  - ノードのfluxは以下のノードすべてのfluxの合計、ほかのやつもそんな感じ  
- computeLightingCone() : - triangleRangeの中のすべてのトライアングルのconeDirectionの平均ベクトルを計算、  
その平均ベクトルと各トライアングルのconeDirectionベクトル間の角度をcomputeCosConeAngle()で計算し、最も角度の大きいcosThetaを返す（ここの計算ではすべてtd.cosConeAngle = 1なのでconeDirection間の計算のみ）  
リーフノード生成の時と、SAOH判定の時のuseLeafCreationCost = true（分割によるコストよりも分割せずリーフノードにした方がコストが低い場合がリーフノードにするオプション）の時のリーフノードコスト計算時に使用される  
- computeLightingConesInternal() : buildInternal()完了の後にnodesのconeDirectionとcosConeAngleを計算する関数  
リーフノードですでに計算してあるconeDirectionとcosConeAngleをもとに、リーフ以外のすべてのnodeに対し再帰的にconeDirectionとcosConeAngleを計算する  
これの計算としてconeUnionOld()が使われ2つのノード間のconeDirectionとcosConeAngleが計算されている（coneUnion()はなんか使われてない。まだユニットテストとかしてないから安全面の観点から使われてないっぽい）  
  - coneUnionOld() : 2つのノードのconeDirectionの平均ベクトルと各coneDirectionの角度を計算し、  
各々のconeDirection角度 + cosConeAngle角度の内、大きいほうを返す  
coneDirectionは計算した平均ベクトルを返す  
Okdと書いているのはタイトな角度になってないからっぽい  
  - coneUnion() : coneUnionOld()よりタイトに2つのノードを合わせたノードのconeDirectionとcosConeAngleを計算するらしい  
TODO : まだ使われてないので見るの保留  
- computeCosConeAngle() : coneDirとotherConeDirの角度 + cosOtherThetaの角度を計算し、cosThetaの角度よりも大きければそちらに更新する関数  
computeLightingCone()でリーフノードのconeDirectionとcosConeAngleの計算として、  
computeSplitWithBinnedSAOH()でbinごとのトライアングルのconeDirectionとcosConeAngleの計算と各分割に対するコスト計算ためのconeDirectionとcosConeAngleの計算に使用される  

### SplitHeuristicFunction集
LightBVHを分割する際の分割場所判定関数群  
SplitResult、つまり分割軸とtriangleRangeに対する分割場所を返す（この分割場所はトライアングルインデックスとかではなく、begin(data.trianglesData) + triangleRange.begin + offsetのoffsetに対応する）    
これらの関数はgetSplitFunction(SplitHeuristic heuristic)によって取得される  

- SplitHeuristic::Equal : computeSplitWithEqual() : トライアングル数に対し等分割するSplitResultの計算  
3軸の内最も距離が長い軸で分割する。分割場所はtriangleRange.middle()  
- SplitHeuristic::BinnedSAH : computeSplitWithBinnedSAH()、SAH or VHによるSplitResultの計算、式（15）  
splitAlongLargestオプションによって距離最大軸 or 3軸の内最もコストが低い軸でのコストとSplitResultをbinAlongDimension()から計算、  
リーフノードとして考えた時よりコストが低かったらそのSplitResultを返す関数（そうでなければ分割しないタイプのSplitResultを返す）  
  - binAlongDimension() : 指定した軸に対して、binningによるコスト計算し、SplitResultを計算するラムダ式  
  各binに対応するトライアングルのAABBをもとに各binのトライアングル包括AABBを計算、  
そこから各分割箇所ごとのコストをevalSAH()により計算し、一番コストが低い分割位置を導出  
そのコストとoverallBestSplitのコストを比較しSplitResultを更新する  
    - evalSAH() : SAH or VHのコストの式（15）の分子片方部分を計算する関数  
- SplitHeuristic::BinnedSAOH : computeSplitWithBinnedSAOH() : SAOH or VOHにおるSplitResultの計算、式（16）  
cosConeAngleの処理部分とコスト計算部分以外は大体computeSplitWithBinnedSAH()と同じ  
cosConeAngleはconeDirectionを作った後でないと計算できないので、  
binとコスト計算の時にcosConeAngle以外いったん全部合成した後に、cosConeAngleをcomputeCosConeAngle()により別に計算してからevalSAOH()でコスト計算する  
    - evalSAOH() : SAOH or VODのコストの式（15）の分子片方部分を計算する関数  
    M_Ωだけ別関数computeOrientationCost()で計算される
    - computeOrientationCost() : 式（15）のM_Ωの計算  
    Equation 1 in Conty & Kulla, "Importance Sampling of Many Lights with Adaptive Tree Splitting", 2018.の計算をしているらしい  


## LightBVH.h, cpp
LightBVH本体  
主にシェーダー側で使われる変数やバッファーの保持と、そのシェーダー変数への設定処理を担当する  
シェーダー変数はLightBVH.slangで定義されており、以下のようになっている  
- nodes : mpBVHNodesBuffer = mNodes、LightBVHの本体  
トラバースの際に必要なデータが入っており、次のノードの場所や、式（18）で必要な値が入っている  
トップノードを0として、左ノードから順に格納されている。そのため子ノードの左側は自分のインデックス+1で計算できるので、リーフじゃないノードは右側のインデックスのみを持っている  
リーフノードはトライアングル数とtriangleIndicesのオフセットが格納されている  
- triangleIndices : mpTriangleIndicesBuffer、左側のリーフノードから順にリーフノード内のトライアングルインデックスが格納されている  
このインデックスをもとにlightCollectionからトライアングル情報を取得し、なんやかんやする  
- triangleBitmasks : mpTriangleBitmasksBuffer、配列番号はトライアングルインデックスに対応しており、どうトラバースすればそこにたどり着くかの情報をbitmaskとして格納してある  
最も左のビットがトップノード->その子ノードの移動情報に対応しており、0で左、1で右にノード移動するようになっている  
これはトライアングルインデックスからそれを選ぶ際のpdfを計算するためのevalPdf()でのみ使われる  

このLightBVHの作成自体はLightBVHBuilderによって行われ、  
mpLightCollectionからmNodes、mpTriangleIndicesBufferのデータ、mpTriangleBitmasksBufferのデータが作られ、  
uploadCPUBuffers()でバッファーに送られ、  
一方updateNodeIndices()の方では、refit()で必要となるデータとして、ツリーの各depthの位置と数情報をまとめたmPerDepthRefitEntryInfoと、そのmPerDepthRefitEntryInfoでアクセスされる、nodes配列のノードインデックスを上から順に詰めていった配列であるmpNodeIndicesBufferを作成する（ただしリーフノードは問答無用で一番右にまとめてある。リーフノードのdepthが同じ保証はないのだが、リーフノードは個別処理をしなければいけないので）  

このようにシェーダー変数割り当てとしての機能だけではなく、シーンのトライアングルの位置などが変更した際のrefit()処理も担当する  
これはLightBVHRefit.cs.slangのシェーダー処理によってGPU側で行われる（そしてそれ用のComputePassの作成と実行が行われている）  
ここで、updateNodeIndices()によって作成されたmpNodeIndicesBufferとmPerDepthRefitEntryInfoが使われ、  
LightBVHRefit.slangの処理によってnodesのリーフノード部分の更新、  
その更新情報をもとに、下から順にリーフじゃないノード更新をする  
LightBVHRefit.slangから分かるように、あくまでノードの中身の値SharedNodeAttributesを書き換えるだけでノード構造の更新は行われないので、  最善のLightBVHからは遠ざかっていく可能性あり  
TODO : そしてなぜかfluxの更新が行われていない。これは多分改善したほうがいいと思う  

残りのcomputeStats()やらrenderStats()やらそれで使われる変数mBVHStatsは、  
imguiに表示するLightBVH情報の計算と表示をするためのもの。デバッグ用  

syncDataToCPU()は使われてないっぽいのでスルー  

### LightBVH.slang
LightBVHのシェーダー変数用の構造体  
LightBVHがLightBVHSampler.slangで使われる読み込み用  
RWLightBVHがLightBVHRefit.slangで使われる更新用  
変数の説明はすでにLightBVH.h, cppで説明してあるのでそっちで  

### LightBVHTypes.slang
node本体  
CPU-GPU間のデータ受け渡し用にPackedNodeが使われ、  
直接dataはいじらずに、pack、unpack用の関数を用いる  
この関数からInternalNode構造体とかLeafNode構造体とかSharedNodeAttributes構造体とかを取得したり逆にこの構造体を作ってpackしたりといったことを行い、CPU、GPU双方でなんやかんやする  
なのでdataの中身とかは別に知らんくていい  

### LightBVHRefit.slang
リーフノード更新用のエントリーポイントupdateLeafNodes()と、  
リーフじゃないノード更新用のエントリーポイントupdateInternalNodes()のやつ  

先にupdateLeafNodes()でリーフノードのトライアングルをLightCollectionから取得し、
SharedNodeAttributesを再計算するだけ  
計算方法はだいたいLightBVHBuilderとおんなじ感じ  
TODO : ただしfluxは更新してない。どうして…どうして…  

そのあとupdateInternalNodes()で指定した深さのリーフじゃないノードのSharedNodeAttributesを再計算していく  
こちらも計算方法はだいたいLightBVHBuilderとおんなじ感じ  
TODO : なおflux

このノードの計算順序は下から順に行わないとおかしくなるが、  
GPU側でその制御はしてないので、CPU側でちゃんとやらなきゃいけないことに注意  

# LightCollection
18.4.1 LIGHT PREPROCESSINGの実装  

create()、update()、setShaderData()、getActiveLightCount()、getStats()はScene側で、  
getMeshLightTriangles()はLightBVH側で呼ばれる  


## LightCollection.h, cpp



シーンのエミッシブメッシェを集め、EmissiveSampler系列で扱えるようデータ形成とシェーダー変数設定をするためのクラス  
このクラスはScene側で管理され、他の場所からはすでに出来上がったLightCollectionをSceneクラス（とシェーダー変数）を利用するという形  
主な処理はcreate()とupdate()、そしてその情報の取得とシェーダー変数設定といった感じになっている  
TODO : ただしupdate()はちょっと怪しいところあり（後述）。要Falcor4.3の変更点チェックと実行チェック  
ちなみにNVAPIがないと実行できない  

create()では、高速化のためいろいろな処理をGPUに任せてある  
まずinit()とprepareMeshData()でCPU側での処理としてbuild()とupdateTrianglePositions()で使うコンピュートパス作成と、pMaterial->isEmissive()なメッシュ情報の配列mMeshLightsとそのmTriangleCounとそのメッシュのトライアングルオフセットtriangleOffsetsのみ、そしてそのBufferを生成する  
その後build()で以下の処理を順に行なっていく
- prepareMeshData() : 上でも書いたが、mMeshLightsのBuffer、そしてtriangleOffsetsとそのBufferを作成する  
triangleOffsetsはシーンの（エミッシブによらず）メッシュインスタンス数分だけ配列作成し、mMeshLightsのやつだけトライアングルオフセットを設定する  
（ただし、このtriangleOffsetsはPathTraser.slangでのみ呼ばれるgetTriangleIndex()でしか使われていない）
- prepareTriangleData() : gTriangleDataとgFluxDataのBufferを作って、buildTriangleList()を呼びmpTriangleListBuilder（BuildTriangleList.cs.slangを実行するコンピュートパス）を実行  
これはmMeshLightsのメッシュごとに、GPU側でシーンの情報をもとにしてgTriangleDataを設定していく  
なおgFluxDataはここで割り当てを行うわけではない（のでこの関数で作成されるのはどうかとおもう）  
- integrateEmissive() : GPU側に設定したgTriangleDataをもとに、gFluxDataの計算をGPU側で行う  
まず、mIntegrator（EmissiveIntegrator.ps.slangのためのパスなどの構造体）で、全てのトライアングルをテクスチャーサイズに合わせてラスタライズし、その色とweightをRWByteAddressBufferであるgTexelSumに加算していく  
この加算にinterlockk処理NvInterlockedAddFp32というNVAPIを使っている  
その後、格納したgTexelSumに対し、mpFinalizeIntegration（FinalizeIntegration.cs.slangのコンピュートパス）を使いgFluxDataのデータ設定を行ってこの関数は終わり  
- prepareSyncCPUData() : copyDataToStagingBuffer()の外部関数。ただし外部で呼ばれてないので直接copyDataToStagingBufferだけでいい  
mStagingBufferValidやmCPUInvalidDataを見て、GPUで計算したgTriangleDataとgFluxDataをCPUリードバック用バッファーmpStagingBufferにコピーする処理  
リードバックとフェンスはsyncCPUData()の方で行う（この後すぐに呼ばれる）  
create()関数以外に外部で扱われるgetMeshLightTriangles()によるsyncCPUData()でも呼ばれる  
- updateActiveTriangleList() : syncCPUData()でGPUで計算したデータを呼び戻し、  
GPUで計算したデータをもとにfluxが0でないトライアングルのみを格納したmpActiveTriangleListを作成して完了  
このデータを使って効率よくLightLBVなどでサンプリングする  

update()では位置変更があったかの判定のみをし、必要ならupdateTrianglePositions()を呼び出す  
updateTrianglePositions()ではmpTrianglePositionUpdater（UpdateTriangleVertices.cs.slangのコンピュートパス）にすでにinit()で処理済みシェーダー 変数を設定と実行をし、GPU内のmpTriangleDataを更新する  
CPU側との同期はmCPUInvalidDataとmStagingBufferValidをもとにgetMeshLightTriangles()によるsyncCPUData()で行われる  
ただし、トライアングルの面積更新によるfluxの更新はFinalizeIntegration.cs.slangを実行するmpFinalizeIntegrationコンピュートパスで行われるのだが、この実行は現状create()関数でしか行われないのでその部分がちゃんと更新されていない可能性があることに注意  
- update()ではシーンのエミッシブメッシュの位置変更のみを更新する  
一応更新必要なものだけ集めてはいるが、その後呼ばれる更新用シェーダーの変数設定関数updateTrianglePositions()のコメントからもわかるように、  
更新はGPUで行っており、CPU側でそのメッシュだけ集めてシェーダー変数設定するよりも、GPU側で全直しした方がオーバーヘッドが少ないので全更新している  
TODOコメントにもあるように今後excuteIndirect()でそこもどうにかする予定があるのかもしれないが、現状updatedLightsの収集は特に意味なし  

computeStats()はimguiに表示するようの情報作成  
SceneクラスのrenderUI()でのみ呼ばれているので、LightCollection.hの情報を確認したければSceneのimguiを見るとよさそう  

### BuildTriangleList.cs.slang
PackedEmissiveTriangle（LightCollectionで必要なトライアングルデータをパックしたもの）の配列であるgTriangleDataを作成するためのシェーダー  
CPU側で割り当てたメッシュ取得用情報CBの値をもとにgSceneのメッシュを取得し、PackedEmissiveTriangleの変数に該当する値を割り当てていくだけ  

### EmissiveIntegrator.ps.slang
Ray Tracing Gems Chapter 18.4.1に書いてある三角形からfluxを計算するためのシェーダーその1  
ここではトライアングルのテクスチャーの解像度をもとにトライアングルを引き延ばしピクセルシェーダーでミップ0テクスチャートライアングルを描画できるようにする  
そのピクセルシェーダーから得た値をgTexelSumに格納していくことにより、そのトライアングルのラディアンスの合計を計算する感じ  
NvInterlockedAddFp32はNVAPIの機能の一つで、なんかatomic処理であるInterlockedAddなやつらしい  
[# Unlocking GPU Intrinsics in HLSL](https://developer.nvidia.com/unlocking-gpu-intrinsics-hlsl)  

このgTexelSumをFinalizeIntegration.cs.slangで処理する  

### FinalizeIntegration.cs.slang
Ray Tracing Gems Chapter 18.4.1に書いてある三角形からfluxを計算するためのシェーダーその2  
トライアングルごとにgTexelSumをluminanceに変換し、トライアングルの面積で割ってpiかけることでfluxを計算する  
gTriangleDataと同じ配置でgFluxData配列にその値を入れて終わり  

### UpdateTriangleVertices.cs.slang
update()で行われる、トライアングルの位置とかスケール更新をgTriangleDataに反映するだけ  

ちなみに面積の変更もあるのだが、create()以外ではFinalizeIntegration.cs.slangは呼ばれないので、update()関連処理ではfluxが更新されないことに注意  

## その他
### BBox.h
LightBVH系列でだけ使われるAABB  
ヘッダーコメントにもあるように、すでにFalcorにあるAABB.h、cppがあるので、このようなstruct名になっている  
AABB.htはcenterとextentだが、これはminPointとmaxPointでAABBを表現している  
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjA1MjYwNTgxXX0=
-->