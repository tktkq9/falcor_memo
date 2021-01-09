# RenderContext関連
コマンドリストの処理を担当するもの  
これに[ProgramVarsサブクラス（ディスクリプターハンドル）](https://github.com/tktkq9/falcor_memo/blob/master/ProgramVars%2C%20ParameterBlock%2C%20ProgramReflection%E9%96%A2%E9%80%A3.md)とか[GraphicsState系のやつ（パイプラインステート）とかProgramKernels（ルートシグネチャー）](https://github.com/tktkq9/falcor_memo/blob/main/ProgramVersion%2C%20Program%2C%20Shader%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E9%96%A2%E9%80%A3.md)とかを渡してコマンドラインの設定とか実行とかさせて描画とかさせる  


RenderContextがComputeContextのサブクラスで描画用  
ComputeContextがCopyContextのサブクラスでコンピュート用  
CopyContextがコピー用  

そして、これらで設定されたコマンドリストをクローズしてキューに渡して実行してフェンスかけてアロケーターとリストリセットするの担当がLowLevelContextDataとなっている  


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

ComputeContext、RenderContextでも使われる共通処理として
- bindDescriptorHeaps : ディスクリプターヒープのコマンドリスト設定
- d3d12ResourceBarrier : リソース全般のバリア設定

あとはコピー系の処理のみ  

## LowLevelContextData
複数のコマンドリストを生成、取得、実行する部分を担当する  
Contextからそれらを行うために、Context系のメンバ変数となっている  
（ちなみにContext以外でも使われているのでContext専用ではない）  

        enum class CommandQueueType
        {
            Copy,
            Compute,
            Direct,
            Count
        };
ごとに別々のLowLevelContextData（つまりCommandQueue）が作られるっぽい  


<!--stackedit_data:
eyJoaXN0b3J5IjpbMTI0MTY4OTc1NCwtOTg4Nzk4OTExLDgxMj
Y1Mzc1MCwtNjg4Njg4MzU0LDM0OTQ5MzQyOSwtMTE2MTc3Njc1
NiwxMzE2MDAwNTIxLDEzOTcwNDM0ODQsMTA2Mjg4MTcxNCwtND
Q4NTA1MTI4LDE4MTk4MzQ4ODIsLTEzMjA3NTc4MiwtMTMyMzE5
MzA5NiwxMTAwODY5MTQyLDI4MzQ1MDY5OSw5NDQ1MTUwOTMsLT
I4MDUzMTQ2NiwxNTQ2MDIwNTA4LDk1NjkyNzExMiw1NzUwOTE4
OTVdfQ==
-->