# RenderPassプロジェクト集

Mogwaiで利用されるレンダーパスのdll作成プロジェクト集    

パス作成関数create()はpybind11によってMogwaiでpythonから呼ばれ、  
パスコンパイルcompile()、インプットアウトプット情報設定関数reflect()はMogwai上でのパス作成時に呼ばれ、  
excute()がレンダーグラフの描画時、つまりつなげたパスごとに順々に呼ばれていく  

renderUI()はMogwaiエディターでのUI表示設定  

その他Mogwai上でキー操作を行った時とかホットリロードとかシーン読み込みとかの実装も場合によっては行う  

Mogwaiはpythonで実行されるので、パスの変数設定関数などMogwai側で呼び出したいその他関数をpybind11によって呼び出せるようにするように、以下のような関数も実装している  

    extern "C" __declspec(dllexport) void getPasses(Falcor::RenderPassLibrary& lib) {…}  

## AccumulatePass

渡したリソースをフレームごとに蓄積し、アウトプットリソースにその平均を返すパス  

カメラの移動などのシーン変化やリソースの変更、パス設定の変更などで蓄積がリセットされる  

蓄積の仕方は全て総和による平均で、制度によって以下の違いがある    
Single : float精度総和バッファ  
Double : double精度総和バッファ、floatバッファ2つ用いることによってdouble精度を再現している  
SingleCompensated : カハンの加算アルゴリズム、float総和バッファの他に、総和で精度から漏れてしまった数値が補正バッファに格納される  

in : ShaderResource
out : RenderTarget | UnorderedAccess | ShaderResource & RGBA32Float
inとoutのリソースサイズは同じでなければならない  

## Antiailiasing

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

## BlitPas
blit実行用パス  
リソースをレンダーターゲットへサンプラー使って描画  

設定はサンプラーがPointかLinearか選べるのみ  

in リソース
out リソース
arrayとmipは1のみ、各々のリソースのサイズは制限なし  

## BSDFViewer
読み込んだシーンのマテリアルの見た目を確かめるためのパス  
imguiでBSDFの設定やカメラや表示したいマテリアルなどの指定を行いマテリアルの見た目を確認する用  

sliceViewerがoffなら球体を（useDirectionalLightじゃなければレイトレで）BxDF.slangのBSDFを用いて描画する  
球体がレイトレの場合は単純なそのフレーム1サンプルのみのDXRも用いてない（そこらへんはすべてBSDFViewer.cs.slangで実装されている）描画なので、完全確認用だと思われる（AccumulatePassをこの後にかましてもいいかも）  

sliceViewerがonなら指定したtexCoordsuvの情報に対するuvを緯度経度に変換した画像を表示  
sliceViewerの場合はレイトレではないので、もしかしたらほかのパスで使えるのかもしれない  

キーやマウス操作はマテリアルの変更とクリックしたピクセル情報表示のみ  

out : RGBA32Float、UnorderedAccess  
このパスはoutのみ  

## CSM
カスケードシャドウマップ（正確には[SDSM (Sample Distribution Shadow Map)](https://software.intel.com/content/www/us/en/develop/articles/sample-distribution-shadow-maps.html)）のパス  

分割の仕方は
- Linear : 等間隔
- Logarithmic ; 対数間隔
- PSSM : Parallel Split Shadow Map。シャドウのピクセルの大きさが画面から見たときにちょうど1ピクセルの大きさに近くなるよう分割箇所を調整する感じのやつ  
[ASURA 平行分割シャドウマップ](http://www.project-asura.com/program/d3d11/d3d11_009.html)
[GPU Gems 3 Chapter 10. Parallel-Split Shadow Maps on Programmable GPUs](https://developer.nvidia.com/gpugems/gpugems3/part-ii-light-and-shadows/chapter-10-parallel-split-shadow-maps-programmable-gpus)

見た目重視ならPSSMを選ぶといいはず（pssmLambda = 0.5がGPU Gems 3で使われている値）  

シャドウマップのエッジフィルターは
- pcf系列 : Percentage Closer Filtering [ref1 (fixed and stochastic)](https://developer.nvidia.com/gpugems/gpugems/part-ii-lighting-and-shadows/chapter-11-shadow-map-antialiasing)
- vsm系列 : Variance Shadow Map [ref1](https://hexadrive.jp/lab/demo/572/), [ref2](http://asura.iaigiri.com/XNA_GS/xna33.html)
  - evsm : Exponential Variance Shadow Mapping [ref1](https://www.martincap.io/project_detail.php?project_id=9)

比較例 
[vsm vs esm vs evsm](https://www.cg.tuwien.ac.at/research/publications/2013/ADORJAN-2013-ASE/ADORJAN-2013-ASE-thesis.pdf)
[pcf vs vsm vs evsm](https://www.martincap.io/project_detail.php?project_id=9)
[vsm系列の方がpcfより1.5倍速い説](https://community.khronos.org/t/shadow-filtering-pcf-better-than-vsm/67180)
ざっと調べた感じ、vsmの方がpcfより速そう  


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0NDAyOTgwODUsMjA0ODI2ODQ3MCw1MD
g0NDg4MDksNjI4MjE1MzIzLC0xMjQ4MzM4MDcwLC03NzExNTE1
MzksLTE2NjI4ODEyMzcsMjAwOTU1NzQwNiw4OTQyMjgwOTEsLT
E1Mjk1ODkzMDgsMzEwNjA2MDE1LC0xMjI5NzU0NTM1LC0xNzY5
OTY3NDcwLDIzNDYzMjM2NiwxMjI4Njc4MDI1LDIwNzk2OTg5NT
UsLTIwNzcwNDA0NDQsLTU0NTM2MjAzNyw5MDI4MDcyNTAsMjEw
NDkwNDUzMF19
-->