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
- prepareForDraw : ルートシグネチャーとパイプラインステートやビューポートなど描画命令以外の処理を行う。変数のハンドル設定だけは直接ここで行われるのではなく、  ProgramVarsのapply()関数 -> D3D12DescriptorSetによってコマンドリスト設定という流れになっている  
- set, clear系 : prepareForDrawとかで行っている設定を個別に行う用、prepareForDrawでもset系は使われている  
- draw系 : バリアはってdeaw系呼び出し  
- blit : blit実行  
- resolve系 : resolve呼び出し  


## ComputeContext.h, cpp
TODO    D3D12ComputeContextを読む  


### D3D12ComputeContext.cpp
TODO   読む  



## CopyContext.h, cpp
TODO  D3D12CopyContextを読む  


### D3D12CopyContext.cpp
TODO  読む  

<!--stackedit_data:
eyJoaXN0b3J5IjpbNTk2NDQyMTQxLDU3NTA5MTg5NSwtMTIzMD
M0NjQ5LC0yMDc4NTk0MDAzLDE5MDcyNTExMzUsMjk2NTQ3NjM2
LC02NzA2NzQ4MTAsMTM1NzUxMzMzOSwtMTc0NjU5NjI1MiwtMT
g5NjYwODM1MCwxMjQ1ODEyNTQxLC0xOTExOTY1OTAzLDE1ODUx
MDQ1NzAsLTE0NDA3NjU2MjUsNDI1MzQ4NDU5LC0xMzIxNjY4NT
k2LDMxMzY2MDIzNSwtMTEwNjM2Nzc0NSwxOTk3OTc1MTQ3LC0y
MTA3OTE5ODk2XX0=
-->