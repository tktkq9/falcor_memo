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
imguiでBSDの設定Fやカメラや表示したいマテリアルなどの指定を行い、そのパラメーターをシェーダーに渡して、その選んだマテリアルごとの見た目を確認する  

sliceViewerがonならTODO : 

キーやマウス操作はマテリアルの変更とクリックしたピクセル情報表示のみ  

out : RGBA32Float、UnorderedAccess  
このパスはoutのみ  
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTczMzcxNjMwNiwtMTg2NTY1MjA4MSwyOT
Q4MzM3NiwtNDM1NTg0MTU3LDgxNTI1MTI4LC0xMjQzNjM5ODg3
LC01NTIyMTM2MjgsLTg4NjU2MjEwMiw0MjMwOTAwNTAsLTE2Mz
I3MDg1NzQsLTEwNTU0MTQ4OTYsMTc4MTk3MDUzMCwtMTk5MTk5
NDMwOCwxNDE3MDY5OTkwLDEwNTEyMjE2MywxMjkzMDE0ODcxXX
0=
-->