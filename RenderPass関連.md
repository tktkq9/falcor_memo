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
https://docs.microsoft.com/ja-jp/previous-versions/direct-x/ee419733(v=vs.85)?redirectedfrom=MSDN
https://docs.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resolvesubresource

リゾルブはMSAAを適用したいときに必要らしい  
MSAAはシングルサンプルリソースしか受け付けてないのでリソースがマルチサンプルの場合はこのパスをかます  
（MSAA以外では必要にならない？）  

実装は渡されるRenderDataの


<!--stackedit_data:
eyJoaXN0b3J5IjpbOTkyMjY3NTMxLDE0NTY5NDA0NjksLTE3OT
g4ODA5MjAsLTE0ODE3NzI5ODEsMTM4MDM1NTM0NCw0NDMxMTA4
NzYsMTcwNTg5NDIzNiwyMzg1MjUwMCw3Njg4NDg4MzUsLTI2Nz
AzODMwOSw3MzA5OTgxMTZdfQ==
-->