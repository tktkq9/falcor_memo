# FlacorプロジェクトRenderPasses関連

## BaseGraphicsPassおよび関連するクラス

## BaseGraphicsPass
これ自体はGraphicsVars、GraphicsState、GraphicsProgramを生成と保持し、間接的にこれらのメンバ変数を実行するだけのクラス  
（サブクラスで利用されている）ParameterBlockSharedPtrによってShaderVarにアクセスできるようgetRootVar()が実装されている  
drawやdispatch処理さえないので、完全にサブクラス作成前提のインターフェース的な役割  


## GraphicsState
TODO : 詳しく読む
RenderContextのdrawInstancedなどに渡される2つのうちの1つ  
ルートシグネチャー、モデルのリソース、パイプライン、複数のビューポート、シザー、FBOなどGraphicsVars以外のものを設定、管理  
また、ブレンドやデプスステンシルやカリングなどの設定管理はGraphicsStateObjectの方に押し付けている  
大体は直接このクラスを通して設定するが、ProgramKernelsとRootSignatureはgetGSO()によってGraphicsVarsからもらってくる  

## StateGraph
その名の通り汎用的に使える状態遷移図  
ただし、GraphicsState、ComputeState、RtProgramでしか使われていない  

GraphicsState、ComputeState、RtProgramでの使われ方としては、NodeTypeの格納、検索、再利用  
描画に必要なデータ（例えばルートシグネチャーやFBOなど）をもとに各状態をwalk()を使って検索、なければ作成し、
同じ状態を持つ場合は状態に設定したdataに割り当てたNodeType（例えばGraphicsStateではGraphicsStateObject）をgetCurrentNode()によって取得するために使われている  

これらの設定をもとにGraphicsStateObjectを作成する  
ただしすでに同じ設定のものがあるかmpGsoGraphをもとに検索され再利用される

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
eyJoaXN0b3J5IjpbLTE5MzE0NDAwNzgsLTEzNDk4MTg3MzYsNz
g4Njc3MDQxLC02NDE1MTE0NSwtMTc2NjI1MjA4MywtNDEwOTYw
Njc0LDE0OTAzMTI0MDMsLTE1MTM1ODIzNDcsLTQxODE3NzY1NS
wtMTU4ODE3MzgwMyw1ODE4MjMxLDM5MDkwNzgwMywtMTUyMDQx
MzA3NywtMzQ2NDAyMzI5LDU0MTU2NDgxNiwyMTk0MjkxMTQsMT
YzMjkyOTEyMiwzMjg3NjgwNjUsMTMxMDAwNDAyOCwxOTM0MTgz
NTgxXX0=
-->