# RenderContext関連
コマンドリストの処理を担当するもの  
これにProgramVarsサブクラス（ディスクリプターハンドル）とかGraphicsState系のやつ（パイプラインステート）とかProgramKernels（ルートシグネチャー）とかを渡してコマンドラインの設定とか実行とかさせて描画とかさせる  

RenderContextがComputeContextのサブクラスで描画用  
ComputeContextがCopyContextのサブクラスでコンピュート用  
CopyContextがコピー用  

## RenderContext
ComputeContextのサブクラス  
コマンドリストを保持し、描画系のコマンドリスト設定を担当する  
（もちろんComputeContextのサブクラスなのでComputeContextとCopyContextの機能も使える）  
リソースで考えるとFBO、RTV、DSV担当  

これにGraphicsState（ルートシグネチャー、パイプラインステート、シザービューポートなど変数以外の部分）やGraphicsVars（ディスクリプター、ハンドルなど変数部分）を渡して、描画に関連するマンドリスト処理を行う  

### D3D12RenderContext.cpp
RenderContextのdx12の場合の処理  

内容は以下のようなものとなっている  
- init : blit用の初期化のみ  
- prepareForDraw : ルートシグネチャーとパイプラインステートやビューポート、ハンドル設定など描画以外のコマンドリスト設定を行う
- set, clear系 : prepareForDrawとかで行っている設定を個別に行う用、prepareForDrawでもset系は使われている  
- draw系 : prepareForDraw呼んで（必要ならバリアはって）deaw系のコマンドリスト設定  
- blit : blit実行  
- resolve系 : resolve呼び出し  

prepareForDrawの変数のハンドル設定だけは直接ここで行われるのではなく、  ProgramVarsのapply()関数呼び出し -> D3D12DescriptorSetの関数呼び出しによってコマンドリスト設定、という流れになっている  

## ComputeContext
CopyContextのサブクラス  
コマンドリストを保持し、dispatch、dispatchIndirectのコマンドライン設定を担当する  
（もちろんCopyContextのサブクラスなのでCopyContextの機能も使える）  
リソースで考えるとUAV担当   

これにComputeState（ルートシグネチャー、パイプラインステート、シザービューポートなど変数以外の部分）やComputeVars（ディスクリプター、ハンドルの変数部分）を渡して、描画に関連するマンドリスト処理を行う  


### D3D12ComputeContext.cpp
ComputeContextのdx12の場合の処理  

内容は以下のようなものとなっている  
- init : ID3D12CommandSignatureが無ければ作成のみ。ID3D12CommandSignatureはdispatchIndirect用
- prepareForDispatch: ルートシグネチャーとパイプラインステート、ハンドル設定などdispatch以外のコマンドリスト設定を行う
- set, clear系 : prepareForDrawとかで行っている設定を個別に行う用、prepareForDrawでもset系は使われている  
- dispatch : prepareForDispatch読んでdispatchのコマンドリスト設定  
- dispatchIndirect : prepareForDispatch読んでバリアはってdispatchIndirectのコマンドリスト設定。ここでinitで作ったpDispatchCommandSigが利用される  
- 
RenderContextと同じく、prepareForDrawの変数のハンドル設定だけは直接ここで行われるのではなく、  ProgramVarsのapply()関数呼び出し -> D3D12DescriptorSetの関数呼び出しによってコマンドリスト設定、という流れになっている  

## CopyContext
コマンドリストを保持し、コピー系（そしてコピーが必要なテクスチャーのアップデート系もここ）のコマンドリスト設定担当する  

あと、ComputeContext、RenderContextで使われる共通処理の実装もここ  

### D3D12CopyContext.cpp
CopyContextのdx12の場合の処理  

ComputeContext、RenderContextで使われる共通処理として
- bindDescriptorHeaps : ディスクリプターヒープのコマンドリスト設定
- d3d12ResourceBarrier : リソース全般のバリア設定
- 


## LowLevelContextData

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTQxMjUxOTIxLC02ODg2ODgzNTQsMzQ5ND
kzNDI5LC0xMTYxNzc2NzU2LDEzMTYwMDA1MjEsMTM5NzA0MzQ4
NCwxMDYyODgxNzE0LC00NDg1MDUxMjgsMTgxOTgzNDg4MiwtMT
MyMDc1NzgyLC0xMzIzMTkzMDk2LDExMDA4NjkxNDIsMjgzNDUw
Njk5LDk0NDUxNTA5MywtMjgwNTMxNDY2LDE1NDYwMjA1MDgsOT
U2OTI3MTEyLDU3NTA5MTg5NSwtMTIzMDM0NjQ5LC0yMDc4NTk0
MDAzXX0=
-->