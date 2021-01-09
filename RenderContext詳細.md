# RenderContext関連
コマンドリストの処理を担当するもの  
これにProgramVarsサブクラス（ディスクリプターハンドル）とかGraphicsState系のやつ（パイプラインステート）とかProgramKernels（ルートシグネチャー）とかを渡してコマンドラインの設定とか実行とかさせて描画とかさせる  
<font color=#bfbfbf>（もっともディスクリプターハンドルの設定はRenderContextに渡したProgramVarsにRenderContextを渡して設定するというあっちゃこっちゃいった処理をするので渡すという表現は微妙だけど）</font>


CopyContextがコピー用  
ComputeContextがCopyContextのサブクラスでコンピュート用  
RenderContextがComputeContextのサブクラスで描画用  

## RenderContext.h, cpp
TODO    D3D12RenderContextを読む  
コマンドリストを保持し、  
描画系のコマンドライン設定と実行を担当する  

これにGraphicsState（ルートシグネチャー、パイプラインステート）やGraphicsVars（変数、ディスクリプター）を



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
eyJoaXN0b3J5IjpbLTcyODk0NDcxMCwyOTY1NDc2MzYsLTY3MD
Y3NDgxMCwxMzU3NTEzMzM5LC0xNzQ2NTk2MjUyLC0xODk2NjA4
MzUwLDEyNDU4MTI1NDEsLTE5MTE5NjU5MDMsMTU4NTEwNDU3MC
wtMTQ0MDc2NTYyNSw0MjUzNDg0NTksLTEzMjE2Njg1OTYsMzEz
NjYwMjM1LC0xMTA2MzY3NzQ1LDE5OTc5NzUxNDcsLTIxMDc5MT
k4OTYsLTgxODUwMTk1OCwtMTExODAxMzEwMyw5OTUwNjQxMDks
MjA2MjQ5MTc1MF19
-->