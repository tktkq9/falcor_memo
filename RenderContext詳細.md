# RenderContext関連
コマンドリストの処理を担当するもの  
これにProgramVarsサブクラス（ディスクリプターハンドル）とかGraphicsState系のやつ（パイプラインステート）とかProgramKernels（ルートシグネチャー）とかを渡してコマンドラインの設定とか実行とかさせて描画とかさせる  

RenderContextがComputeContextのサブクラスで描画用  
ComputeContextがCopyContextのサブクラスでコンピュート用  
CopyContextがコピー用  

## RenderContext.h, cpp
ComputeContextのサブクラス  
コマンドリストを保持し、描画系のコマンドライン設定と実行を担当する  
（もちろんComputeContextのサブクラスなのでComputeContextとCopyContextの機能も使える）  
リソースで考えるとFBO、RTV、DSV担当  

これにGraphicsState（ルートシグネチャー、パイプラインステート、シザービューポートなど変数以外の部分）やGraphicsVars（ディスクリプター、ハンドルなど変数部分）を渡して、描画に関連するマンドリスト処理を行う  

### D3D12RenderContext.cpp
RenderContextのdx12の場合の処理  

内容は以下のようなものとなっている  
- init : blit用の初期化のみ  
- prepareForDraw : ルートシグネチャーとパイプラインステートやビューポート、ハンドル設定など描画以外のこｍ処理を行う
- set, clear系 : prepareForDrawとかで行っている設定を個別に行う用、prepareForDrawでもset系は使われている  
- draw系 : バリアはってdeaw系呼び出し  
- blit : blit実行  
- resolve系 : resolve呼び出し  

prepareForDrawの変数のハンドル設定だけは直接ここで行われるのではなく、  ProgramVarsのapply()関数呼び出し -> D3D12DescriptorSetの関数呼び出しによってコマンドリスト設定、という流れになっている  

## ComputeContext.h, cpp
CopyContextのサブクラス  
コマンドリストを保持し、描画系のコマンドライン設定と実行を担当する  
（もちろんCopyContextのサブクラスなのでCopyContextの機能も使える）  
リソースで考えるとUAV担当   

これにComputeState（ルートシグネチャー、パイプラインステート、シザービューポートなど変数以外の部分）やComputeVars（ディスクリプター、ハンドルの変数部分）を渡して、描画に関連するマンドリスト処理を行う  


### D3D12ComputeContext.cpp
TODO   読む  



## CopyContext.h, cpp
TODO  D3D12CopyContextを読む  


### D3D12CopyContext.cpp
TODO  読む  

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTEwMDg2OTE0MiwxMTA5NjQ5OTgwLDI4Mz
Q1MDY5OSw5NDQ1MTUwOTMsLTI4MDUzMTQ2NiwxNTQ2MDIwNTA4
LDk1NjkyNzExMiw1NzUwOTE4OTUsLTEyMzAzNDY0OSwtMjA3OD
U5NDAwMywxOTA3MjUxMTM1LDI5NjU0NzYzNiwtNjcwNjc0ODEw
LDEzNTc1MTMzMzksLTE3NDY1OTYyNTIsLTE4OTY2MDgzNTAsMT
I0NTgxMjU0MSwtMTkxMTk2NTkwMywxNTg1MTA0NTcwLC0xNDQw
NzY1NjI1XX0=
-->