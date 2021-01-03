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

リゾルブが必要な


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE3OTg4ODA5MjAsLTE0ODE3NzI5ODEsMT
M4MDM1NTM0NCw0NDMxMTA4NzYsMTcwNTg5NDIzNiwyMzg1MjUw
MCw3Njg4NDg4MzUsLTI2NzAzODMwOSw3MzA5OTgxMTZdfQ==
-->