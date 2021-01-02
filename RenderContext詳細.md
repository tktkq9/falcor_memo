# RenderContext関連
コマンドリストの処理を担当するもの  
これにProgramVarsサブクラス（ディスクリプターハンドル）とかGraphicsState系のやつ（パイプラインステート）とかProgramKernels（ルートシグネチャー）とかを渡してコマンドラインの設定とか実行とかさせて描画とかさせる  
<font color=#bfbfbf>（もっともディスクリプターハンドルの設定はRenderContextに渡したProgramVarsにそのRenderContextを渡して設定するというあっちゃこっちゃいった処理をするので渡すという表現は微妙だけど）</font>


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
eyJoaXN0b3J5IjpbLTEwMjQ1NzE0MjEsLTY3MDY3NDgxMCwxMz
U3NTEzMzM5LC0xNzQ2NTk2MjUyLC0xODk2NjA4MzUwLDEyNDU4
MTI1NDEsLTE5MTE5NjU5MDMsMTU4NTEwNDU3MCwtMTQ0MDc2NT
YyNSw0MjUzNDg0NTksLTEzMjE2Njg1OTYsMzEzNjYwMjM1LC0x
MTA2MzY3NzQ1LDE5OTc5NzUxNDcsLTIxMDc5MTk4OTYsLTgxOD
UwMTk1OCwtMTExODAxMzEwMyw5OTUwNjQxMDksMjA2MjQ5MTc1
MCwtOTYyMjA3NDg4XX0=
-->