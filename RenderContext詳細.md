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

init : blit用の初期化のみ  
prepareForDraw : ルートシグネチャーとパイプラインステートやビューポートなど、変数設定と描画命令以外の処理を行う  
draw系 : バリアはってdeaw系呼び出し  
set


## ComputeContext.h, cpp
TODO    D3D12ComputeContextを読む  


### D3D12ComputeContext.cpp
TODO   読む  



## CopyContext.h, cpp
TODO  D3D12CopyContextを読む  


### D3D12CopyContext.cpp
TODO  読む  

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQxMDM5OTAzMiwyMTMwMzAxOTIyLDE5MD
cyNTExMzUsMjk2NTQ3NjM2LC02NzA2NzQ4MTAsMTM1NzUxMzMz
OSwtMTc0NjU5NjI1MiwtMTg5NjYwODM1MCwxMjQ1ODEyNTQxLC
0xOTExOTY1OTAzLDE1ODUxMDQ1NzAsLTE0NDA3NjU2MjUsNDI1
MzQ4NDU5LC0xMzIxNjY4NTk2LDMxMzY2MDIzNSwtMTEwNjM2Nz
c0NSwxOTk3OTc1MTQ3LC0yMTA3OTE5ODk2LC04MTg1MDE5NTgs
LTExMTgwMTMxMDNdfQ==
-->