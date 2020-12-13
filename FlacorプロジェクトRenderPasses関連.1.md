# FlacorプロジェクトRenderPasses関連

## BaseGraphicsPassおよび関連するクラス

## BaseGraphicsPass
これ自体はGraphicsVarsとGraphicsStateを保持し、間接的にこれらのメンバ変数を実行するだけのクラス  これらに処理を渡すだけ  
drawやdispatch処理さえないので、完全にサブクラス作成前提のインターフェース的な役割  


## GraphicsProgram

## GraphicsState

# BaseGraphicsPassのサブクラス

## FullScreenPass
ピクセルシェーダーのみあうかう
    using SharedPtr = ParameterBlockSharedPtr<FullScreenPass>;
によって以下のようにシェーダーの変数を設定できるようにしている  

    mpMainPass["ToyCB"]["iResolution"] = float2(width, height);
    mpMainPass["ToyCB"]["iGlobalTime"] = (float)gpFramework->getGlobalClock().getTime();


<!--stackedit_data:
eyJoaXN0b3J5IjpbMTY1MzgzNDU4OSwxMzEwMDA0MDI4LDE5Mz
QxODM1ODEsLTIwMDA2MzQ5MzEsMTY4Nzc5Njc5MiwtMjA5MTgw
MjM2OV19
-->