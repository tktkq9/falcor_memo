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

これにGraphicsState（ルートシグネチャー、パイプラインステート、シザービューポートなど変数以外の部分）やGraphicsVars（ディスクリプター、ハンドルなど変数部分）を渡して、描画などのコマンドリスト処理を行う  

### D3D12RenderContext.cpp
RenderContextのdx12の場合の処理  

prepareForDraw : ルートシグネチャーとパイプラインステートと、

## ComputeContext.h, cpp
TODO    D3D12ComputeContextを読む  


### D3D12ComputeContext.cpp
TODO   読む  



## CopyContext.h, cpp
TODO  D3D12CopyContextを読む  


### D3D12CopyContext.cpp
TODO  読む  

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTA1MTUwODg2MSwxOTA3MjUxMTM1LDI5Nj
U0NzYzNiwtNjcwNjc0ODEwLDEzNTc1MTMzMzksLTE3NDY1OTYy
NTIsLTE4OTY2MDgzNTAsMTI0NTgxMjU0MSwtMTkxMTk2NTkwMy
wxNTg1MTA0NTcwLC0xNDQwNzY1NjI1LDQyNTM0ODQ1OSwtMTMy
MTY2ODU5NiwzMTM2NjAyMzUsLTExMDYzNjc3NDUsMTk5Nzk3NT
E0NywtMjEwNzkxOTg5NiwtODE4NTAxOTU4LC0xMTE4MDEzMTAz
LDk5NTA2NDEwOV19
-->