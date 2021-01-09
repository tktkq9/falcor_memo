# RenderContext関連
コマンドリストの処理を担当するもの  
これにProgramVarsサブクラス（ディスクリプターハンドル）とかGraphicsState系のやつ（パイプラインステート）とかProgramKernels（ルートシグネチャー）とかを渡してコマンドラインの設定とか実行とかさせて描画とかさせる  
<font color=#bfbfbf>（もっともディスクリプターハンドルの設定はRenderContextに渡したProgramVarsにRenderContextを渡して設定するというあっちゃこっちゃいった処理をするので渡すという表現は微妙だけど）</font>


CopyContextがコピー用  
ComputeContextがCopyContextのサブクラスでコンピュート用  
RenderContextがComputeContextのサブクラスで描画用  

## RenderContext.h, cpp
コマンドリストを保持し、  
描画系のコマンドライン設定と実行を担当する  

これにGraphicsState（ルートシグネチャー、パイプラインステート）やGraphicsVars（変数、ディスクリプター、ハンドル）を渡して、描画などのコマンドリスト処理を行う  

### D3D12RenderContext.cpp
RenderContextのdx12の場合の処理  



## ComputeContext.h, cpp
TODO    D3D12ComputeContextを読む  


### D3D12ComputeContext.cpp
TODO   読む  



## CopyContext.h, cpp
TODO  D3D12CopyContextを読む  


### D3D12CopyContext.cpp
TODO  読む  

<!--stackedit_data:
eyJoaXN0b3J5IjpbNjA3MjYyNDM4LDE5MDcyNTExMzUsMjk2NT
Q3NjM2LC02NzA2NzQ4MTAsMTM1NzUxMzMzOSwtMTc0NjU5NjI1
MiwtMTg5NjYwODM1MCwxMjQ1ODEyNTQxLC0xOTExOTY1OTAzLD
E1ODUxMDQ1NzAsLTE0NDA3NjU2MjUsNDI1MzQ4NDU5LC0xMzIx
NjY4NTk2LDMxMzY2MDIzNSwtMTEwNjM2Nzc0NSwxOTk3OTc1MT
Q3LC0yMTA3OTE5ODk2LC04MTg1MDE5NTgsLTExMTgwMTMxMDMs
OTk1MDY0MTA5XX0=
-->