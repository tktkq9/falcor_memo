# FlacorプロジェクトRenderPasses関連
TODO: NVAPIの理解

## BaseGraphicsPassおよび関連するクラス

## BaseGraphicsPass
これ自体はGraphicsVars、GraphicsState、GraphicsProgramを生成と保持し、間接的にこれらのメンバ変数を実行するだけのクラス  
（サブクラスで利用されている）ParameterBlockSharedPtrによってShaderVarにアクセスできるようgetRootVar()が実装されている  
drawやdispatch処理さえないので、完全にサブクラス作成前提のインターフェース的な役割  


## GraphicsState
RenderContextのdrawInstancedなどに渡される2つのうちの1つでパイプラインステートオブジェクトのようなもの  
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


## GraphicsVars
TODO : 詳しく読む
シェーダーに渡す変数、バッファーなどを管理  
RenderContextのdrawInstancedなどに渡される2つのうちの1つ  


## ShaderVar


# BaseGraphicsPassのサブクラス

## FullScreenPass
画面に自動で四角形を描画し、ピクセルシェーダーのみを扱う場合のパス  

    using SharedPtr = ParameterBlockSharedPtr<FullScreenPass>;
によって以下のようにシェーダーの変数を

    mpMainPass["ToyCB"]["iResolution"] = float2(width, height);
    mpMainPass["ToyCB"]["iGlobalTime"] = (float)gpFramework->getGlobalClock().getTime();  
のように設定できるようになっている    

頂点シェーダー（とviewportMaskの立っているビットの数だけレンダーターゲットを作成するためのジオメトリシェーダー）とその頂点バッファーなどの作成と設定はFullScreenPassのコンストラクタで自動で行われる  
複数回呼ばれるのを想定して、頂点バッファーはgFullScreenDataによって再利用されている  

故に頂点とジオメトリシェーダーはすでに用意されている  
頂点シェーダー（FullScreenPass.vs.slang）は頂点とuvをそのまま流すだけ  
ジオメトリシェーダー（FullScreenPass.gs.slang）はviewportMaskの立っているビット分のレンダーターゲットを作成するためのもの（参考文献：[ジオメトリシェーダを使用した複数画面描画](https://sites.google.com/site/monshonosuana/directxno-hanashi-1/directx-107)、[DirectX12 でシングルパスキューブマップ描画](https://blog.techlab-xe.net/directx12-render-cubemap-singlepass/)）  


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTk2ODI5NjUzMiwtMjEzNTY0Mzk4NSwxNj
UxNTA2MDk4LC0xNTQ4NzI2OTYsMTcyNTAwNDQxMCwtNzgwNTM1
ODk2LDE1ODY4NTIzNzgsNDQ1MzI0MjUwLC0xMzQ5ODE4NzM2LD
c4ODY3NzA0MSwtNjQxNTExNDUsLTE3NjYyNTIwODMsLTQxMDk2
MDY3NCwxNDkwMzEyNDAzLC0xNTEzNTgyMzQ3LC00MTgxNzc2NT
UsLTE1ODgxNzM4MDMsNTgxODIzMSwzOTA5MDc4MDMsLTE1MjA0
MTMwNzddfQ==
-->