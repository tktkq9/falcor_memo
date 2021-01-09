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

- init : blit用の初期化のみ  
- prepareForDraw : ルートシグネチャーとパイプラインステートやビューポートなど、変数設定と描画命令以外の処理を行う  
- draw系 : バリアはってdeaw系呼び出し  



## ComputeContext.h, cpp
TODO    D3D12ComputeContextを読む  


### D3D12ComputeContext.cpp
TODO   読む  



## CopyContext.h, cpp
TODO  D3D12CopyContextを読む  


### D3D12CopyContext.cpp
TODO  読む  

<!--stackedit_data:
eyJoaXN0b3J5IjpbNzA5MzIyOTcxLDE0MTAzOTkwMzIsMTkwNz
I1MTEzNSwyOTY1NDc2MzYsLTY3MDY3NDgxMCwxMzU3NTEzMzM5
LC0xNzQ2NTk2MjUyLC0xODk2NjA4MzUwLDEyNDU4MTI1NDEsLT
E5MTE5NjU5MDMsMTU4NTEwNDU3MCwtMTQ0MDc2NTYyNSw0MjUz
NDg0NTksLTEzMjE2Njg1OTYsMzEzNjYwMjM1LC0xMTA2MzY3Nz
Q1LDE5OTc5NzUxNDcsLTIxMDc5MTk4OTYsLTgxODUwMTk1OCwt
MTExODAxMzEwM119
-->