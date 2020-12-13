# FlacorプロジェクトRenderPasses関連

## BaseGraphicsPassおよび関連するクラス

## BaseGraphicsPass
これ自体はGraphicsVarsとGraphicsStateを保持し、間接的にこれらのメンバ変数を実行するだけのクラス  これらに処理を渡すだけ  
（サブクラスで利用されている）ParameterBlockSharedPtrによってShaderVarにアクセスできるようgetRootVar()が実装されている  
drawやdispatch処理さえないので、完全にサブクラス作成前提のインターフェース的な役割  


## GraphicsProgram

## GraphicsState

## ShaderVar


# BaseGraphicsPassのサブクラス

## FullScreenPass
ピクセルシェーダーのみ扱う場合のパス  

    using SharedPtr = ParameterBlockSharedPtr<FullScreenPass>;
によって以下のようにシェーダーの変数を

    mpMainPass["ToyCB"]["iResolution"] = float2(width, height);
    mpMainPass["ToyCB"]["iGlobalTime"] = (float)gpFramework->getGlobalClock().getTime();  
のように設定できるようになっている    

頂点シェーダー（とviewportMaskの立っているビットの数だけ描画するためのジオメトリシェーダー）とその頂点バッファーなどはFullScreenPassのコンストラクタ側で作成される  
複数回呼ばれるのを想定して、頂点バッファーはgFullScreenDataによって再利用されている  

頂点とジオメトリシェーダーはFullScreenPass.vs.slangとFullScreenPass.gs.slang  
頂点シェーダーは頂点とuvをそのまま流すだけ  
ジオメトリシェーダーはviewportMaskの立っているビット分のレンダーターゲットを作成（参考文献：[ジオメトリシェーダを使用した複数画面描画](https://sites.google.com/site/monshonosuana/directxno-hanashi-1/directx-107)）  


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTExMzAzODEzMDgsNTQxNTY0ODE2LDIxOT
QyOTExNCwxNjMyOTI5MTIyLDMyODc2ODA2NSwxMzEwMDA0MDI4
LDE5MzQxODM1ODEsLTIwMDA2MzQ5MzEsMTY4Nzc5Njc5MiwtMj
A5MTgwMjM2OV19
-->