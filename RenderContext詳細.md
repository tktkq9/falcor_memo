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

内容は以下のようなものとなっている  
- init : blit用の初期化のみ  
- prepareForDraw : ルートシグネチャーとパイプラインステートやビューポートなど描画命令以外の処理を行う。変数設定だけは直接ここで設定するのではなく、  
- set, clear系 : prepareForDrawとかで行っている設定を個別に行う用、prepareForDrawでもset系は使われている  
- draw系 : バリアはってdeaw系呼び出し  
- blit : blit実行  
- resolve系 : resolve呼び出し  

変数のハンドル設定だけはここではなくProgramVarsのapply()関数で行われる  

## ComputeContext.h, cpp
TODO    D3D12ComputeContextを読む  


### D3D12ComputeContext.cpp
TODO   読む  



## CopyContext.h, cpp
TODO  D3D12CopyContextを読む  


### D3D12CopyContext.cpp
TODO  読む  

<!--stackedit_data:
eyJoaXN0b3J5IjpbNTc1MDkxODk1LDE4NzU0NTYzMTQsLTEyMz
AzNDY0OSwtMjA3ODU5NDAwMywxOTA3MjUxMTM1LDI5NjU0NzYz
NiwtNjcwNjc0ODEwLDEzNTc1MTMzMzksLTE3NDY1OTYyNTIsLT
E4OTY2MDgzNTAsMTI0NTgxMjU0MSwtMTkxMTk2NTkwMywxNTg1
MTA0NTcwLC0xNDQwNzY1NjI1LDQyNTM0ODQ1OSwtMTMyMTY2OD
U5NiwzMTM2NjAyMzUsLTExMDYzNjc3NDUsMTk5Nzk3NTE0Nywt
MjEwNzkxOTg5Nl19
-->