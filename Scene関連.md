# Scene関連

TODO : Flacorプロジェクト下のSceneディレクトリにあるやつ全部読む  

## Importer
ファイルからインポートするときの複数のImpoterをまとめるラッパークラス  
Impoter本体はregisterImporter()関数によって登録され、この関数はREGISTER_IMPORTERのdefineによって呼び出される  

現状登録されているImpoterはAssimpImporter、PythonImporter、SceneImporter  

### AssimpImporter
[assimp](https://github.com/assimp/assimp)というAPIによるモデルファイルインポートクラス  
使い方は[assimpのサイト](https://assimp-docs.readthedocs.io/en/latest/)や

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTI4NDY1ODQxNywtNjQ2NDE4OTgsLTE4MD
kzODQ3NCwtMTU5MzQ1MTAyMF19
-->