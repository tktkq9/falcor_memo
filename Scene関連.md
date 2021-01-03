# Scene関連

TODO : Flacorプロジェクト下のSceneディレクトリにあるやつ全部読む  

## Importer
ファイルからインポートするときの複数のImpoterをまとめるラッパークラス  
Impoter本体はregisterImporter()関数によって登録され、この関数はREGISTER_IMPORTERのdefineによって呼び出される  

現状登録されているImpoterはAssimpImporter、PythonImporter、SceneImporter  

### AssimpImporter
[assimp](https://github.com/assimp/assimp)というAPIによるモデルファイルインポートクラス  
使い方は[assimpのサイト](https://assimp-docs.readthedocs.io/en/latest/)とか[]()とか[OpenGLによる実装例](http://ogldev.atspace.co.uk/www/tutorial38/tutorial38.html)

<!--stackedit_data:
eyJoaXN0b3J5IjpbODEzNjA4NjQ0LC02NDY0MTg5OCwtMTgwOT
M4NDc0LC0xNTkzNDUxMDIwXX0=
-->