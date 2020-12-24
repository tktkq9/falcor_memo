# FlacorプロジェクトRenderPasses関連

## BaseGraphicsPassおよび関連するクラス

## BaseGraphicsPass
これ自体はGraphicsVars、GraphicsState、GraphicsProgramを生成と保持し、間接的にこれらのメンバ変数を実行するだけのクラス  
（サブクラスで利用されている）ParameterBlockSharedPtrによってShaderVarにアクセスできるようgetRootVar()が実装されている  
drawやdispatch処理さえないので、完全にサブクラス作成前提のインターフェース的な役割  


## GraphicsState
TODO : 詳しく読む
ビューポート、シザー、ルートシグネチャー、モデルのリソース、パイプライン、複数のFBOなどGraphicsVars以外のものを設定、管理  
RenderContextのdrawInstancedなどに渡される2つのうちの1つ  



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
eyJoaXN0b3J5IjpbLTQxMDk2MDY3NCwxNDkwMzEyNDAzLC0xNT
EzNTgyMzQ3LC00MTgxNzc2NTUsLTE1ODgxNzM4MDMsNTgxODIz
MSwzOTA5MDc4MDMsLTE1MjA0MTMwNzcsLTM0NjQwMjMyOSw1ND
E1NjQ4MTYsMjE5NDI5MTE0LDE2MzI5MjkxMjIsMzI4NzY4MDY1
LDEzMTAwMDQwMjgsMTkzNDE4MzU4MSwtMjAwMDYzNDkzMSwxNj
g3Nzk2NzkyLC0yMDkxODAyMzY5XX0=
-->