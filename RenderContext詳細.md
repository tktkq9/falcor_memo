# RenderContext関連
コマンドリストの処理を担当するもの  
これにProgramVarsサブクラス（ディスクリプターハンドル）とかGraphicsState系のやつ（パイプラインステート）とかProgramKernels（ルートシグネチャー）とかを渡してコマンドラインの設定とか実行とかさせて描画とかさせる  
（もっともディスクリプターハンドルの設定はRenderContextに渡したProgramVarsにそのRenderContextを渡して設定するというあっちゃこっちゃいった処理をするので渡すというひょうげんは）

CopyContextがコピー用  
ComputeContextがCopyContextのサブクラスでコンピュート用  
RenderContextがComputeContextのサブクラスで描画用  

## RenderContext.h, cpp
TODO    D3D12RenderContextを読む  


### D3D12RenderContext.cpp
TODO   読む  


## ComputeContext.h, cpp
TODO    D3D12ComputeContextを読む  


### D3D12ComputeContext.cpp
TODO   読む  



## CopyContext.h, cpp
TODO  D3D12CopyContextを読む  


### D3D12CopyContext.cpp
TODO  読む  

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTY3MDY3NDgxMCw2Mjk2NTcyODMsMTM1Nz
UxMzMzOSwtMTc0NjU5NjI1MiwtMTg5NjYwODM1MCwxMjQ1ODEy
NTQxLC0xOTExOTY1OTAzLDE1ODUxMDQ1NzAsLTE0NDA3NjU2Mj
UsNDI1MzQ4NDU5LC0xMzIxNjY4NTk2LDMxMzY2MDIzNSwtMTEw
NjM2Nzc0NSwxOTk3OTc1MTQ3LC0yMTA3OTE5ODk2LC04MTg1MD
E5NTgsLTExMTgwMTMxMDMsOTk1MDY0MTA5LDIwNjI0OTE3NTAs
LTk2MjIwNzQ4OF19
-->