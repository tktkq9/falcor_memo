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
https://docs.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resolvesubresource

リゾルブはMSAAを適用したいときに必要らしい  
MSAAはシングルサンプルリソースしか受け付けてないのでリソースがマルチサンプルの場合はこのパスをかます  
（MSAA以外では必要にならない？）  
参考文献 : [](https://zerogram.info/?p=1746)

### 実装詳細
渡されるRenderDataのrenderData[kSrc]->asTexture()とrenderData[kDst]->asTexture()を受け取って、RenderContextのresolveResource()に渡すだけ  


<!--stackedit_data:
eyJoaXN0b3J5IjpbMzU1OTIwMzQ1LC0xMjIxNDYyNDM1LDE0NT
Y5NDA0NjksLTE3OTg4ODA5MjAsLTE0ODE3NzI5ODEsMTM4MDM1
NTM0NCw0NDMxMTA4NzYsMTcwNTg5NDIzNiwyMzg1MjUwMCw3Nj
g4NDg4MzUsLTI2NzAzODMwOSw3MzA5OTgxMTZdfQ==
-->