# Scene関連

TODO : Flacorプロジェクト下のSceneディレクトリにあるやつ全部読む  

## Importer
ファイルからインポートするときの複数のImpoterをまとめるラッパークラス  
Impoter本体はregisterImporter()関数によって登録され、この関数はREGISTER_IMPORTERのdefineによって呼び出される  

現状登録されているImpoterはAssimpImporter、PythonImporter、SceneImporter  

### AssimpImporter
[Assimp]()というAPIによるモデルファイルインポートクラス  

<!--stackedit_data:
eyJoaXN0b3J5IjpbNTYwNjI3MzA3LC02NDY0MTg5OCwtMTgwOT
M4NDc0LC0xNTkzNDUxMDIwXX0=
-->