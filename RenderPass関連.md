# RenderPass関連

Mogwaiで使われるパスのベースクラス関連  
（Falcorプロジェクト内ではないほうの）RenderPassesディレクトリ内にあるプロジェクトでこのサブクラスが作られている  

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
そして、その返り値であるFieldから、入出力するリソースのデータも設定できる（しなければならない）  

これらの関数で設定したFiled配列の管理を行う  

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

## ResolvePass
RenderPassのサブクラス  
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
eyJoaXN0b3J5IjpbMTc0ODMwNjIxNCwtODQyNjk2MTkwLDE3NT
E5NDY4OTksMzkyMTg5OTE5LC04ODUxODY3OTIsLTEyNDE3MTM0
NjksMTgyMTc3NzI3MSwxNjg0MTYyLDM1MTU5MDAwMiwtMTkzMj
Q0MDA5NSwyNDI5OTc5NzAsLTkyMjgyNTk4NSw5NTAzNzQ1NSwt
NTYxMDczODIyLC0xMjIxNDYyNDM1LDE0NTY5NDA0NjksLTE3OT
g4ODA5MjAsLTE0ODE3NzI5ODEsMTM4MDM1NTM0NCw0NDMxMTA4
NzZdfQ==
-->