# RenderPass、RenderGraph関連
TODO : RenderGraph周りはざっくりとしか読んでないのでいずれちゃんと読む  

Mogwaiで使われるパスのベースクラス関連  
（Falcorプロジェクト内ではないほうの）RenderPassesディレクトリ内にあるプロジェクトでこのサブクラスが作られている  

# RenderPass周り

## RenderPass
様々なパスのアブストラクトクラス  
基本的なメンバ変数と、コンストラクタで渡されたそれらをそのままメンバ変数に割り当てる実装のみで、残りの関数はすべてアブストラクト実装  
サブクラス作成の注意点は以下の通り  

Render passes are expected to implement a static create() function that returns a shared pointer to a new object, or throws an exception if creation failed. The constructor should be private to force creation of shared pointers.

Render passes are inserted in a render graph, which is executed at runtime. Each render pass declares its I/O requirements in the reflect() function, and as part of the render graph compilation their compile() function is called. At runtime, execute() is called each frame to generate the pass outputs.

### CompileData
Mogwai初期化時に渡されるもの  
TODO : まだどんなデータかわからん

### RednerData
RenderPass::execute()で渡されるデータ  
グローバルなリソースや変数を管理する  
あと、RenderPassReflectionで指定されてないときのデフォルトな設定とか  

## RenderPassReflection
MogwaiでのパスUI情報を設定、保持しておくためのクラス  

addInput()、addOutput()、addInputOutput()でインプット、アウトプットのリソースの名前とその概要文を設定する  
一方addInternal()は内部用リソースの設定  
そして、その返り値であるFieldから、入出力するリソース情報も設定できる（しなければならない）  

RenderPassReflectionはこのFieldの作成をサポートし、作成したFiled配列の管理を行う  

### Field
パスのインプット、アウトプット、内部で使われるリソース情報  
リソース一つにつき1Field  
リソース本体は格納せず  

リソースのタイプごとにある程度決まった設定をする必要があるので、それようの関数を呼んで設定する      
たとえばTexture3DならField::texture3D()といった感じで  
また、リソースによらない設定はそれ用のセッター関数がある  
これらにより、例えば

    field.format(mFormat).texture2D(0, 0, 0);
といった感じで設定する  

## RenderPassHelpers  
細々とした関数  

ChannelListからDefineもらうのと  
ChannelListによってパスのインプットアウトプット作るやつ  

## ResourceCache  
Fieldに対するリソース本体の作成、格納用クラス  

最初にResourceDataにFieldとリソース以外の情報を更新または作成しmResourceDataとmNameToIndexにその情報を入れ、  
allocateResources()でmResourceDataのリソースを一気に作成する  

# RenderGraphのUIじゃない方周り
MogwaiのGraphEditerで表示されるパスとそれが線でつながっているやつ全体に関するやつら  

これを使ってRenderPassの集まりを処理し、複数のパスによる描画とか行う  

## RenderGraph  
レンダーパスグラフのクラス  
グラフ上のパスとエッジを管理する 
またそれぞれのパスのコンパイルや様々なパス処理の大元もここである  

コンパイルはRenderGraphCompiler、  
それぞれのパス処理はRenderGraphExeが担当しており、    
これらを通してコンパイル、パス処理が行われる  

## RenderGraphCompiler  
RenderGraphExeを作成するためのクラス  

RenderGraph側で

    mpExe = RenderGraphCompiler::compile(*this, pContext, mCompilerDeps);
によってRenderGraphExeが作成されている  
また、その時に渡したmCompilerDepsからデフォルトリソース設定情報や外部リソースの情報が格納される  

この関数内で行われているcompilePasses(pContext)によってRenderPassのcompile()とreflect()が呼ばれ、各パスが作成される  
またResourcesCacheのリソースもallocateResources()によりここで作成される 

## RenderGraphExe  
RenderGraphに設定した全てのRenderPassの実行を担当する（compileとreflect以外全て） 

mExecutionListの順序で各パスの処理が行われる  
このmExecutionListはRenderGraphCompilerによってRenderGraphから作成される  

## RenderPassLibrary  
全てのRederPassの情報を格納し、そこから作成するためのクラス  

シングルトンであり、instance()を通して呼び出される  

createPass()によってこの関数に指定したパスを作成し返す 

# RenderGraphのUI周り

## RenderPassUI



## RenderGraphIR
中間表現（intermediate representation）  
RenderGraph作成用pythonコードを記述するためのヘルパークラス  

RenderGraphImportExport用のセーブデータ作成や、  
RenderGraphUIでのパスやエッジの追加や削除や更新とその他設定に利用される  

作ったコードはgetIR()して実行コードstringを取得し、セーブしたり  
そのstringを加工してScriptingクラスによって実行したりしている  

実行コードを使えるようにするためにRenderGraphの方で RenderGraphの関数とRenderGraphIR側で定義した名前とのpybind11処理をしている  

## RenderGraphImportExport  
RenderGraphの状態をRenderGraphIRによってセーブデータ読み込み書き込み  




# その他

## ResolvePass
唯一Falcorプロジェクト内で実装されているRenderPassのサブクラス  
SrcをDstにResolveするためのPass  
SrcとDstは、リソース タイプと次元が同じである必要があり、コピー元とコピー先のフォーマットには互換性が必要  
[ID3D11DeviceContext::ResolveSubresource 日本語](https://docs.microsoft.com/ja-jp/previous-versions/direct-x/ee419733(v=vs.85))
[# ID3D12GraphicsCommandList::ResolveSubresource method (d3d12.h) 英語](https://docs.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resolvesubresource)

リゾルブはMSAAを適用したいときに必要らしい  
MSAAはシングルサンプルリソースしか受け付けてないのでリソースがマルチサンプルの場合はこのパスをかます  
（MSAA以外では必要にならない？）  
参考文献 : [# VS2015でDirectX12プログラミング](https://zerogram.info/?p=1746)とか

### 実装詳細
渡されるRenderDataのrenderData[kSrc]->asTexture()とrenderData[kDst]->asTexture()を受け取って、RenderContextのresolveResource()に渡すだけ  


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTcwMjk5NzA3NSw2NjIxNjQ1NzEsMTY3OT
c4MzMsNjAyNDU1ODI4LDgwMTc3MDYyOSwtMTUxNzQ2MDgwNCw3
Nzc3OTY3MjksLTE5MjM5OTYyNTEsMTU0NjQ4OTM0NywxODI0Mz
MwODcsODA0NTM5NDEwLDc0NTEwMDQyMiw4NDAyODM4ODYsMTk3
NzQ1ODI1MywtODU3MjY3ODI2LC0xMDc2NzEwMzcwLC04NDI2OT
YxOTAsMTc1MTk0Njg5OSwzOTIxODk5MTksLTg4NTE4Njc5Ml19

-->