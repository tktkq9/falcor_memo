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

addInput()、addOutput()でインプット、アウトプットの名前とその概要文を設定する  
また、その返り値であるFieldから、入出力するリソースのデータも設定できる  

### Field
パスのインプット、アウトプット、内部で使われるリソース情報  
リソース一つにつき1Field  
リソース本体は格納せず  

リソースのタイプごとにある程度決まった設定をする必要があるので、それようの関数を呼んで設定する      
たとえばTexture3DならField::texture3D()といった感じで  
また、リソースによらない設定はそれ用のセッター関数がある  
これらを駆使して、例えば

    reflector.addInput(kSrc, "Multi-sampled texture").format(mFormat).texture2D(0, 0, 0);
といった感じで  

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
eyJoaXN0b3J5IjpbMTU5NzQ3ODE5MywzOTIxODk5MTksLTg4NT
E4Njc5MiwtMTI0MTcxMzQ2OSwxODIxNzc3MjcxLDE2ODQxNjIs
MzUxNTkwMDAyLC0xOTMyNDQwMDk1LDI0Mjk5Nzk3MCwtOTIyOD
I1OTg1LDk1MDM3NDU1LC01NjEwNzM4MjIsLTEyMjE0NjI0MzUs
MTQ1Njk0MDQ2OSwtMTc5ODg4MDkyMCwtMTQ4MTc3Mjk4MSwxMz
gwMzU1MzQ0LDQ0MzExMDg3NiwxNzA1ODk0MjM2LDIzODUyNTAw
XX0=
-->