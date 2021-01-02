# RenderContext関連
コマンドリストの処理を担当するもの  
これにProgramVarsサブクラス（ディスクリプターハンドル）とかGraphicsState系のやつ（パイプラインステート）とかProgramKernels（ルートシグネチャー）とかを渡してコマンドラインの設定とか実行とかさせて描画とかさせる  
（もっとも）

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
eyJoaXN0b3J5IjpbNDk4NjY3NjgyLDEzNTc1MTMzMzksLTE3ND
Y1OTYyNTIsLTE4OTY2MDgzNTAsMTI0NTgxMjU0MSwtMTkxMTk2
NTkwMywxNTg1MTA0NTcwLC0xNDQwNzY1NjI1LDQyNTM0ODQ1OS
wtMTMyMTY2ODU5NiwzMTM2NjAyMzUsLTExMDYzNjc3NDUsMTk5
Nzk3NTE0NywtMjEwNzkxOTg5NiwtODE4NTAxOTU4LC0xMTE4MD
EzMTAzLDk5NTA2NDEwOSwyMDYyNDkxNzUwLC05NjIyMDc0ODgs
LTc1MTU1NzU4Ml19
-->