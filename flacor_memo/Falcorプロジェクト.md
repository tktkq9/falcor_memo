# Falcorプロジェクト
場合によってはビルドでC2220エラーがでるので以下の方法で対処
1.  プロジェクトのプロパティを開く
2.  `構成プロパティ`  >  `C/C++`  >  `全般`を開く
3.  `警告をエラーとして扱う`を`いいえ（/WX-）`に変更

参考 : https://qiita.com/ledsun/items/a3bd7691b86389073c7a

これ単体では動かず（dll作成用プロジェクト）、このdllをほかのプロジェクトで使い描画処理を実装している  
サンプルフォルダのテンプレートを見ればわかるように、Renderer.hのIRendererインターフェースを継承したクラスを用意し、Winmain関数を作り、Sample.h, cppで実装してあるstatic run()関数を実行することにより描画できる  

# メイン


## Renderer.h
描画処理のみを抽出したインターフェース（IRenderer）と、そのためのデータ管理やそれ以外の処理のインターフェース（IFramework）の2つが入っている  
サンプルプロジェクトにあるテンプレートはIRendererを継承し描画処理のみを実装し、そのほかの処理はIFrameworkを継承したSample.h, cppに任せてある  

## Sample.h, cpp
描画データと描画以外の処理データ構造のインターフェース（IFramework）を継承して一通りの描画以外の処理をできるようにするもの、またはそのサンプル実装  
IRendererを継承したクラスとSampleConfigを生成し、Sample.h, cppで実装してあるstatic run()関数に渡すことにより描画処理を行っている  
run()実行時にグローバル変数gpFrameworkにこのインスタンスを格納、これを使ってrun()で作ったSampleインスタンスにアクセスでき、描画部分以外の処理ができる  

### IRenderer関連のコード
run() → runInternal()でいろいろ初期化し、最後にこの処理をして描画している
    
        // Load and run
        mpRenderer->onLoad(getRenderContext());
        pBar = nullptr;

        mFrameRate.reset();
        mpWindow->msgLoop();

        mpRenderer->onShutdown();
        if (gpDevice) gpDevice->flushAndSync();
        mpRenderer = nullptr;

mpRendererが渡したIRenderer、mpWindow->msgLoop()は
windowのwhileループとそこでのSample::renderFrame()  の実行  
残りのpBar は描画する前に表示されるプログレスバー、mFrameRateはフレームレート管理、gpDevice->flushAndSync();は最後に残ったコマンドキューが終わるまで待つシャットダウン処理  
IRendererだけを見るとonLoad()1回のみ → onFrameRender()ループ（とonGuiRender()の実行） → ループから抜けたらonShutdown()の流れ  
残りのイベント系列は Window.h, cpp 側のコールバック処理で呼ばれる

## Window.h, cpp
ウィンドウ処理用  
dx12 では HWND に対応

Window クラス自体は作成（ここで ApiCallbacks の関数と GLFWwindow のイベントを関連付け）、解放処理と msgLoop()処理、その他ちょっとした関数のみ  
ウィンドウの作成には GLFW API の GLFWwindow で簡略化されている  
これはOpenGL や Vulkan などの API だが、

    // Don't include GL/GLES headers
    #define GLFW_INCLUDE_NONE
とあるように、おそらくVulkanとは関係なくウィンドウ作成のために使われているだけっぽい（実際このウィンドウ作成はこのプロジェクトのdx12バージョンでもこれが使われている）  

create()呼び出しの時にコールバックを mpCallbacks に割り当て、  
主にキーやマウスやその他イベント処理のための ApiCallbacks クラスで呼び出しを行っている  


## Device.h, cpp
dx12ではID3D12Deviceに対応  
このクラスではID3D12Deviceおよびその周りの作成と設定をまとめている（ファクトリー、スワップチェインとそのFBO、コマンドキュー、RenderContext、フェンス、これらのフレームID管理）  

またDescriptorPoolによるディスクリプターヒープやクエリーヒープ？の管理も行っている  

    poolDesc.setDescCount(DescriptorPool::Type::TextureSrv, 1000000).setDescCount(DescriptorPool::Type::Sampler, 2048).setShaderVisible(true);
アホみたいな量を一気に確保している  

また描画においては、dx12のコマンド関連はRenderContextにほとんどを任せ、pSwapChain->Present()とそのフェンス処理による画面描画を担当  
コマンドリストの格納とコマンドキューの実行については  

    mpRenderContext->flush();
でRenderContextの方で実行している？  

### その他
experimentalFeatures は現状使われてないっぽい  

### D3D12Device.cpp
TODO    読む  

Deviceのdx12部分の実装  

## RenderContext.h, cpp
TODO    D3D12RenderContextを読む  

ComputeContextのサブクラス（ComputeContextはCopyContextのサブクラス）  
clearDsvやらdrawIndexedやらraytraceやらと、描画処理や描画シェーダー周りのコマンドリスト設定とそのDispatch()を担当している  
だいたいの処理はAPIに依存するので、dx12の場合、だいたいの関数はD3D12RenderContext.cppの方で実装されている  

コマンドの実行はflush()で、LowLevelContextApiDataにより行われる  


### D3D12RenderContext.cpp
TODO   読む  

RenderContextのdx12部分の実装  

## ComputeContext.h, cpp
TODO    D3D12ComputeContextを読む  

ComputeContextはCopyContextのサブクラス  
コンピュートシェーダー周りのコマンドリスト設定とそのDispatch()を担当している  
だいたいの処理はAPIに依存するので、dx12の場合、だいたいの関数はD3D12ComputeContext.cppの方で実装されている  

コマンドの実行はflush()で、LowLevelContextApiDataにより行われる  

### D3D12ComputeContext.cpp
TODO   読む  

ComputeContextのdx12部分の実装  


## CopyContext.h, cpp
TODO  D3D12CopyContextを読む  

リソース間のコピーだけではなく、リソースのバリアのコマンドリスト設定も行う  
だいたいの処理はAPIに依存するので、dx12の場合、だいたいの関数はD3D12CopyContext.cppの方で実装されている  
ここでLowLevelContextApiDataが作成される  

コマンドの実行はflush()で、LowLevelContextApiDataにより行われる  

### D3D12CopyContext.cpp
TODO  読む  

CopyContextのdx12部分の実装  

## LowLevelContextApiData.h, D3D12LowLevelContextData.cpp
コマンドリストとアロケーターの作成、コマンドリストとキューとアロケーターの管理、キューの実行を担当  
flush()でExecuteCommandLists()され、コマンドリストに新しいアロケーターが設定される  

ExecuteCommandLists()によるウェイトはここでは行わず、実行したものはFencedPoolにひたすら積まれていく  
ただし、すでに実行が終わっているアロケーターは再利用される  

このようにExecuteCommandLists()ごとに同期はしない設計になっているので、cpu側 or gpu側でウェイトしたい場合はFencedPoolに渡したmpFenceでsyncCpu() or syncGpu()を呼ぶ必要がある  

### D3D12ApiData.h
コマンドアロケーターFencedPoolのラッパークラス  
それだけ  

### FencedPool.h
GpuFenceの現在のシグナル値を参考に、テンプレートオブジェクトの実行が終わっていれば再利用し、そうでなければ新たに作成する  
新たなオブジェクトを作成する場合は、プール作成時に渡されたNewObjectFuncType newFuncの実行により作成される  

今のところこのプールはコマンドアロケーターでのみ使用されている  


## GpuFence,h, D3D12GpuFence.cpp
フェンスの作成と値管理、フェンス値によるCPUとGPUの同期処理を担当  
基本API依存の処理なので、dx12の場合、関数はD3D12GpuFence.cppに定義されている  

TODO  
描画のためのコマンドキューに対するフェンス値はLowLevelContextApiDataで作成されたフェンスによって管理されていると思われるが、ほかでもGpuFence::create()が呼ばれているので違いを後で調べる（おそらく初期化時の処理用だけだと思う多分）  


## DescriptorPool.h, cpp
TODO  より詳しく読む    

dx12ではID3D12DescriptorHeapに対応  
Deviceで作成される  
DescriptorSetに渡されて、リソースとの紐づけが行われる  

## DescriptorSet.h, cpp, D3D12DescriptorSet.cpp
TODO  より詳しく読む    

FalcorD3D12.hで定義されているように、各リソースのハンドルとして扱われている  

    
    using RtvHandle = std::shared_ptr<DescriptorSet>;
    using DsvHandle = std::shared_ptr<DescriptorSet>;
    using SrvHandle = std::shared_ptr<DescriptorSet>;
    using SamplerHandle = std::shared_ptr<DescriptorSet>;
    using UavHandle = std::shared_ptr<DescriptorSet>;
    using CbvHandle = std::shared_ptr<DescriptorSet>;  

ハンドルとあるようにディスクリプターヒープのCPU, GPUハンドルはこのクラスから取得できるようになっている  
また、setCpuHandle()によってCopyDescriptorsSimple()を行いほかの場所で作ったDescriptorSetに対応するリソースをコピーすることもできる  

また、このハンドルに対するシェーダーでのレジスターやスペースの利用範囲、各シェーダーからのVisibility情報としてLayoutも保持している  

TODO : createUavDescriptor()とかのように、layout.addRange(DescriptorSet::Type::TextureUav, 0, 1);とbaseRegIndex = 0で必ず実装される。あくまでこれは範囲であり、シェーダーで実際に設定するレジスター番号とは違うもの？要確認  


## ResourceView.h, cpp, D3D12ResourceViews.cpp
TODO  より詳しく読む    

ResourceViewベースクラスとそのサブクラスがまとめられている  
またテンプレートクラスはApiHandleTypeとあるように、各リソースに対応するDescriptorSet、つまり...Handleが設定されている  

create()でリソースとその情報を渡し、DESC を生成、gpDeviceからハンドル生成、Create...View()によりハンドルとリソースの対応付けを行う  
また、これによって生成されたハンドルクラスDescriptorSetを保持し、そのラップクラスとしてハンドルを管理する  

また、リソースに関する情報もResourceViewInfoクラスとしてcreate()時に作成され、メンバ変数に保持されている  


## QueryHeap.h, D3D12QueryHeap.cpp

？？？？？？:thinking::thinking::thinking::thinking::thinking::thinking::thinking:？？？？？？？？  
  https://microsoft.github.io/DirectX-Specs/d3d/CountersAndQueries.html   
  https://docs.microsoft.com/ja-jp/windows/win32/direct3d12/performance-measurement  
TODO    あとで勉強する  


## FalcorConfig.h
ファルカーノコンフィグ  
プロファイルとログ出力の設定のための#defineのみ  

# サブ

## Framework.h, cpp
Frameworkという名前だがIFrameworkがあるわけではない（IFrameworkはRenderer.hにある。なぞ）  
シェーダー用（ほかでも使う？）のEnumがまとめられている  
また、clampやalignmentなどのちょっとした汎用関数もある  
cppの方はComparisonFuncに対するpybind11処理のみ  


## ScriptBindings.h, cpp, Scripting.h, cpp, Dictionary.h
TODO  
おそらくpybind11のヘルパー  
見るのはぴぃちょんとぴぃびんど11を勉強した後で  
pybind11周りの説明 : https://github.com/NVIDIAGameWorks/Falcor/blob/master/Docs/Usage/Render-Passes.md  
https://buildersbox.corp-sansan.com/entry/2019/12/09/110000  

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTExMzQyNTY4MDVdfQ==
-->