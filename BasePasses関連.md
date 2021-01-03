# Falcorプロジェクト内のBasePasses関連
Falcorプロジェクト内のRenderPassesディレクトリに入っているクラスについてと、  
（一部は違うが）ベースクラスBaseGraphicsPassの中身について  

FalcorプロジェクトじゃないほうのRenderPassesに入っている各レンダーパスプロジェクトはまた別のクラスRenderPassを使っているのでこれはあくまでSamplesディレクトリに入っているプロジェクト用  


TODO : NVAPIの理解　GraphicsStateObject

# BaseGraphicsPassおよび関連するクラス

# BaseGraphicsPass
これ自体はGraphicsVars、GraphicsState、GraphicsProgramを生成と保持し、間接的にこれらのメンバ変数を実行するだけのクラス  

（サブクラスで利用されている）ParameterBlockSharedPtrによってShaderVarにアクセスできるようgetRootVar()が実装されている  
drawやdispatch処理さえないので、完全にサブクラス作成前提のインターフェース的な役割  
この実装と  

    using SharedPtr = ParameterBlockSharedPtr<FullScreenPass>;
のようなSharedPtrの実装によってシェーダーの変数を

    mpMainPass["ToyCB"]["iResolution"] = float2(width, height);
    mpMainPass["ToyCB"]["iGlobalTime"] = (float)gpFramework->getGlobalClock().getTime();  
のように設定できるようになっている    
これのサブクラスでこの機能が使われている  

## GraphicsState
RenderContextのdrawInstancedなどに渡される引数のうちの1つでパイプラインステートオブジェクトのようなもの  
ルートシグネチャー、モデルのリソース、パイプライン、複数のビューポート、シザー、FBOなどGraphicsVars以外のものを設定、管理  
ブレンドやデプスステンシルやカリングなどの設定管理はGraphicsStateObjectの方に押し付けている  

また、パイプラインステートオブジェクトのラッパークラスであるGraphicsStateObjecの作成も担当しており、mpGsoGraphによる設定変更確認をもとにパイプラインオブジェクトの作成or再利用判定も行っている  

大体は直接このクラスを通して設定するが、ProgramKernelsとRootSignatureはgetGSO()によってGraphicsVarsからもらってくる  

### GraphicsStateObject
パイプラインステートオブジェクトのラッパークラス  
作成に必要なデータがこのクラスのDescにすべて格納され、このデータによりパイプラインステートオブジェクトがapiInit()により作成される（このコードはD3D12GraphicsStateObject.cppの方で定義されている）  

ちなみにapiInit()中に作成されているnvApi...はNVAPI用の処理  
TODO: NVAPIの理解  

### StateGraph
その名の通り状態遷移図  
汎用的に使えるが、GraphicsState、ComputeState、RtProgramといったパイプラインステートオブジェクト系でしか使われていない  

GraphicsState、ComputeState、RtProgramでの使われ方としては、NodeType（パイプラインステートオブジェクトのクラスが割り当てられる）の格納、検索、再利用  
描画に必要なデータ（例えばルートシグネチャーやFBOなど）をもとに各データに対応する状態遷移をwalk()を使って検索、なければ作成  
そして同じデータ割り当てに対応する状態遷移を見つけた場合は、設定したNodeTypeオブジェクト（例えばGraphicsStateではGraphicsStateObject）をgetCurrentNode()によって取得、つまりNodeTypeオブジェクトの再利用を行えるようにするために使われている  

## GraphicsProgram
GraphicsStateで使われるProgramクラスのサブクラス  
中身はそのままProgramクラスを作るためのラッパーcreate関数系とpybind11のやつのみ  

ざっくりProgramクラスを説明すると、シェーダーを管理し、そこからシェーダーブロブやルートシグネチャーを保持するProgramKernelsを取得するためのもの  
詳しくは[ProgramVersion, Program, Shaderファイル関連](https://github.com/tktkq9/falcor_memo/tree/main/ProgramVersion%2C%20Program%2C%20Shaderファイル関連.md)  

## GraphicsVars
RenderContextのdrawInstancedなどに渡される引数のうちの1つで変数部分を担当  
ProgramVarsのサブクラスであり、ProgramVarsと同じくシェーダー変数の変更と管理を担当する  
RenderContextのdrawInstancedなどに渡される2つのうちの1つ  

ProgramVarsの詳細は[ProgramVars, ParameterBlock, ProgramReflection関連](https://github.com/tktkq9/falcor_memo/tree/main/ProgramVars%2C%20ParameterBlock%2C%20ProgramReflection関連.md)  


# BaseGraphicsPassのサブクラス（一部例外あり）

## ComputePass
コンピュートシェーダー処理用のパス  
主にコンピュートシェーダーファイルなどからパスを作成して実行する機能のみ  

これはコンピュートなので、BaseGraphicsPassのサブクラスではない  
BaseGraphicsPassのGraphic系クラスの代わりに、ComputeContext、ComputeProgram、ComputeVars、ComputeStateが使われている  

ParameterBlockSharedPtrにより、シェーダーへの変数名アクセス可能  



## FullScreenPass
画面に自動で四角形を描画し、ピクセルシェーダーのみを扱う場合のパス  

ParameterBlockSharedPtrにより、シェーダーへの変数名アクセス可能  

頂点シェーダー（とviewportMaskの立っているビットの数だけレンダーターゲットを作成するためのジオメトリシェーダー）とその頂点バッファーなどの作成と設定はFullScreenPassのコンストラクタで自動で行われる  
複数回呼ばれるのを想定して、頂点バッファーはgFullScreenDataによって再利用されている  

故に頂点とジオメトリシェーダーはすでに用意されている  
頂点シェーダー（FullScreenPass.vs.slang）は頂点とuvをそのまま流すだけ  
ジオメトリシェーダー（FullScreenPass.gs.slang）はviewportMaskの立っているビット分のレンダーターゲットを作成するためのもの（参考文献：[ジオメトリシェーダを使用した複数画面描画](https://sites.google.com/site/monshonosuana/directxno-hanashi-1/directx-107)、[DirectX12 でシングルパスキューブマップ描画](https://blog.techlab-xe.net/directx12-render-cubemap-singlepass/)）  


## RasterPass
モデルのdraw、drawIndexedするだけのシンプルなパス  
モデルの設定などはgetState()によってGraphicsStateに直接設定するなど、メンバ変数を直接いじる必要がある  

ParameterBlockSharedPtrにより、シェーダーへの変数名アクセス可能  

## RasterScenePass
Sceneを描画するだけのパス  

モデルやシェーダーやそういう系の設定やキーイベントなどはSceneとこのPassが自動で行うようになっているので、  
使い方はSceneを渡して描画ターゲットを渡すだけ  

### Scene  
ライト、カメラ、制的モデル、動的モデル、シェーダーとマテリアル、それらのキーイベントによる操作など、シーン全体の管理とデータ保持クラス  

詳しくは[Scene関連](https://github.com/tktkq9/falcor_memo/blob/main/Scene%E9%96%A2%E9%80%A3.md)にまとめる



<!--stackedit_data:
eyJoaXN0b3J5IjpbLTI2MzMyNjY0LDIyOTA0MjIyNiwtOTU4Nj
M5ODcyLDExMDcwODkxMSwtMTA2MzI0MTI3OSwxMjI2NzAxNTUz
XX0=
-->