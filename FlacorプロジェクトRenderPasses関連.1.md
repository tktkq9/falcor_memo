# FlacorプロジェクトRenderPasses関連

## BaseGraphicsPassおよび関連するクラス

## BaseGraphicsPass
これ自体はGraphicsVarsとGraphicsStateを保持し、間接的にこれらのメンバ変数を実行するだけのクラス  これらに処理を渡すだけ  
（サブクラスで利用されている）ParameterBlockSharedPtrによってShaderVarにアクセスできるようgetRootVar()が実装されている  
drawやdispatch処理さえないので、完全にサブクラス作成前提のインターフェース的な役割  


## GraphicsProgram

## GraphicsState



# BaseGraphicsPassのサブクラス

## FullScreenPass
ピクセルシェーダーのみ扱う場合のパス  

    using SharedPtr = ParameterBlockSharedPtr<FullScreenPass>;
によって以下のようにシェーダーの変数を設定できるようにしている  

    mpMainPass["ToyCB"]["iResolution"] = float2(width, height);
    mpMainPass["ToyCB"]["iGlobalTime"] = (float)gpFramework->getGlobalClock().getTime();


<!--stackedit_data:
eyJoaXN0b3J5IjpbOTY4MTQ1NTQyLDE2MzI5MjkxMjIsMzI4Nz
Y4MDY1LDEzMTAwMDQwMjgsMTkzNDE4MzU4MSwtMjAwMDYzNDkz
MSwxNjg3Nzk2NzkyLC0yMDkxODAyMzY5XX0=
-->