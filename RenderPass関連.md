# RenderPass関連

Mogwaiで使われるパスのベースクラス関連  
（Falcorプロジェクト内ではないほうの）RenderPassesディレクトリ内にあるプロジェクトでこのサブクラスが作られている  

## RenderPass

### RednerData


## RenderPassReflection
MogwaiでのパスUI情報を設定、保持しておくためのクラス  

addInput()、addOutput()でインプット、アウトプットの名前とその概要文を設定する  
また、その返り値であるFieldから、入出力するリソースのデータも設定できる  



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
eyJoaXN0b3J5IjpbLTU2MTA3MzgyMiwtMTIyMTQ2MjQzNSwxND
U2OTQwNDY5LC0xNzk4ODgwOTIwLC0xNDgxNzcyOTgxLDEzODAz
NTUzNDQsNDQzMTEwODc2LDE3MDU4OTQyMzYsMjM4NTI1MDAsNz
Y4ODQ4ODM1LC0yNjcwMzgzMDksNzMwOTk4MTE2XX0=
-->