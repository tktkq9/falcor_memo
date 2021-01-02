# RenderContext関連
コマンドリストの処理を担当するもの  
これにProgramVarsサブクラス（ディスクリプターハンドル）とかGraphicsState系のやつ（パイプラインステート）とかProgramKernels（ルートシグネチャー）とかを渡してコマンドラインの設定とか実行とかさせて描画とかさせる  

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
eyJoaXN0b3J5IjpbLTEyMDc5NDQ2NjIsMTM1NzUxMzMzOSwtMT
c0NjU5NjI1MiwtMTg5NjYwODM1MCwxMjQ1ODEyNTQxLC0xOTEx
OTY1OTAzLDE1ODUxMDQ1NzAsLTE0NDA3NjU2MjUsNDI1MzQ4ND
U5LC0xMzIxNjY4NTk2LDMxMzY2MDIzNSwtMTEwNjM2Nzc0NSwx
OTk3OTc1MTQ3LC0yMTA3OTE5ODk2LC04MTg1MDE5NTgsLTExMT
gwMTMxMDMsOTk1MDY0MTA5LDIwNjI0OTE3NTAsLTk2MjIwNzQ4
OCwtNzUxNTU3NTgyXX0=
-->