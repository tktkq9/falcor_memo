# RenderContext関連
コマンドリストの処理を担当するもの  
これにProgramVarsサブクラス（ディスクリプターハンドル）とかGraphicsState系のやつ（パイプラインステート）とかProgramKernels（ルートシグネチャー）とかを渡してコマンドラインの設定実行処理させて描画とかさせる

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
eyJoaXN0b3J5IjpbMTk0NzA0ODE5NiwxMzU3NTEzMzM5LC0xNz
Q2NTk2MjUyLC0xODk2NjA4MzUwLDEyNDU4MTI1NDEsLTE5MTE5
NjU5MDMsMTU4NTEwNDU3MCwtMTQ0MDc2NTYyNSw0MjUzNDg0NT
ksLTEzMjE2Njg1OTYsMzEzNjYwMjM1LC0xMTA2MzY3NzQ1LDE5
OTc5NzUxNDcsLTIxMDc5MTk4OTYsLTgxODUwMTk1OCwtMTExOD
AxMzEwMyw5OTUwNjQxMDksMjA2MjQ5MTc1MCwtOTYyMjA3NDg4
LC03NTE1NTc1ODJdfQ==
-->