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
MSAAはシングルサンプリングしか受け付けてないのでリソースがマルチサンプリングの場合はこのパスをかます  
（）


<!--stackedit_data:
eyJoaXN0b3J5IjpbNDg4MTUwODQzLC0xNzk4ODgwOTIwLC0xND
gxNzcyOTgxLDEzODAzNTUzNDQsNDQzMTEwODc2LDE3MDU4OTQy
MzYsMjM4NTI1MDAsNzY4ODQ4ODM1LC0yNjcwMzgzMDksNzMwOT
k4MTE2XX0=
-->