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
また、パイプラインオブジェクトの作成も（GraphicsStateObjectが実際は行うが）担当しており、mpGsoGraphによる設定変更判定をもとにパイプラインオブジェクトの作成or再利用判定も行っている  

大体は直接このクラスを通して設定するが、ProgramKernelsとRootSignatureはgetGSO()によってGraphicsVarsからもらってくる  
これらの設定をもとにGraphicsStateObjectを作成する  
ただしすでに同じ設定のものがあるかmpGsoGraphをもとに検索され再利用される  

## StateGraph
その名の通り汎用的に使える状態遷移図  
ただし、GraphicsState、ComputeState、RtProgramでしか使われていない  

GraphicsState、ComputeState、RtProgramでの使われ方としては、NodeTypeの格納、検索、再利用  
描画に必要なデータ（例えばルートシグネチャーやFBOなど）をもとに各状態をwalk()を使って検索、なければ作成し、
同じ状態を持つ場合は状態に設定したdataに割り当てたNodeType（例えばGraphicsStateではGraphicsStateObject）をgetCurrentNode()によって取得するために使われている  


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
eyJoaXN0b3J5IjpbOTkwMTE1MjYxLDQ0NTMyNDI1MCwtMTM0OT
gxODczNiw3ODg2NzcwNDEsLTY0MTUxMTQ1LC0xNzY2MjUyMDgz
LC00MTA5NjA2NzQsMTQ5MDMxMjQwMywtMTUxMzU4MjM0NywtND
E4MTc3NjU1LC0xNTg4MTczODAzLDU4MTgyMzEsMzkwOTA3ODAz
LC0xNTIwNDEzMDc3LC0zNDY0MDIzMjksNTQxNTY0ODE2LDIxOT
QyOTExNCwxNjMyOTI5MTIyLDMyODc2ODA2NSwxMzEwMDA0MDI4
XX0=
-->