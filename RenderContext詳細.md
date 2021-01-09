# RenderContext関連
コマンドリストの処理を担当するもの  
これにProgramVarsサブクラス（ディスクリプターハンドル）とかGraphicsState系のやつ（パイプラインステート）とかProgramKernels（ルートシグネチャー）とかを渡してコマンドラインの設定とか実行とかさせて描画とかさせる  
<font color=#bfbfbf>（もっともディスクリプターハンドルの設定はRenderContextに渡したProgramVarsにRenderContextを渡して設定するというあっちゃこっちゃいった処理をするので渡すという表現は微妙だけど）</font>


CopyContextがコピー用  
ComputeContextがCopyContextのサブクラスでコンピュート用  
RenderContextがComputeContextのサブクラスで描画用  

## RenderContext.h, cpp
ComputeContextのサブクラス  
コマンドリストを保持し、描画系のコマンドライン設定と実行を担当する  
（もちろんComputeContextのサブクラスなのでComputeContextとCopyContextの機能も使える）  

これにGraphicsState（ルートシグネチャー、パイプラインステート、シザービューポートなど変数以外の部分）やGraphicsVars（ディスクリプター、ハンドルなど変数部分）を渡して、描画に関連するマンドリスト処理を行う  

### D3D12RenderContext.cpp
RenderContextのdx12の場合の処理  

内容は以下のようなものとなっている  
- init : blit用の初期化のみ  
- prepareForDraw : ルートシグネチャーとパイプラインステートやビューポートなど描画命令以外の処理を行う
- set, clear系 : prepareForDrawとかで行っている設定を個別に行う用、prepareForDrawでもset系は使われている  
- draw系 : バリアはってdeaw系呼び出し  
- blit : blit実行  
- resolve系 : resolve呼び出し  

prepareForDrawの変数のハンドル設定だけは直接ここで行われるのではなく、  ProgramVarsのapply()関数呼び出し -> D3D12DescriptorSetの関数呼び出しによってコマンドリスト設定、という流れになっている  

## ComputeContext.h, cpp
TODO    D3D12ComputeContextを読む  


### D3D12ComputeContext.cpp
TODO   読む  



## CopyContext.h, cpp
TODO  D3D12CopyContextを読む  


### D3D12CopyContext.cpp
TODO  読む  

<!--stackedit_data:
eyJoaXN0b3J5IjpbOTQ0NTE1MDkzLC0yODA1MzE0NjYsMTU0Nj
AyMDUwOCw5NTY5MjcxMTIsNTc1MDkxODk1LC0xMjMwMzQ2NDks
LTIwNzg1OTQwMDMsMTkwNzI1MTEzNSwyOTY1NDc2MzYsLTY3MD
Y3NDgxMCwxMzU3NTEzMzM5LC0xNzQ2NTk2MjUyLC0xODk2NjA4
MzUwLDEyNDU4MTI1NDEsLTE5MTE5NjU5MDMsMTU4NTEwNDU3MC
wtMTQ0MDc2NTYyNSw0MjUzNDg0NTksLTEzMjE2Njg1OTYsMzEz
NjYwMjM1XX0=
-->