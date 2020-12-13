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
によって以下のようにシェーダーの変数を設定できるようにしている  

    mpMainPass["ToyCB"]["iResolution"] = float2(width, height);
    mpMainPass["ToyCB"]["iGlobalTime"] = (float)gpFramework->getGlobalClock().getTime();

頂点シェーダー（とviewportMaskの立っているビットの数だけ描画するためのジオメトリシェーダー）と画面に描画するための頂点などはFullScreenPassのコンストラクタ側で作成される  
複数回呼ばれるのを想定して、頂点バッファーはgFullScreenDataによって再利用される  
<!--stackedit_data:
eyJoaXN0b3J5IjpbNzgwMDk4MDA5LDIxOTQyOTExNCwxNjMyOT
I5MTIyLDMyODc2ODA2NSwxMzEwMDA0MDI4LDE5MzQxODM1ODEs
LTIwMDA2MzQ5MzEsMTY4Nzc5Njc5MiwtMjA5MTgwMjM2OV19
-->