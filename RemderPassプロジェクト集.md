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
in : FBOに対応するフォーマット  
out : FBOのフォーマット  

TODO : 設定できる値とシェーダー はよくわからん。FXAAのアルゴリズムとか使い方とかを要確認

### TAA
TAAは前フレームの描画情報を用いたアンチエイリアス。ディファード、レイトレでよく使われている  

in : モーションベクター 
in : FBOに対応するフォーマット  
out : FBOのフォーマット  
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0ODQyNTkxMTUsMTc4MTk3MDUzMCwtMT
k5MTk5NDMwOCwxNDE3MDY5OTkwLDEwNTEyMjE2MywxMjkzMDE0
ODcxXX0=
-->