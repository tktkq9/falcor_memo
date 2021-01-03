# Scene関連

TODO : Flacorプロジェクト下のSceneディレクトリにあるやつ全部読む  

## Importer
ファイルからインポートするときの複数のImpoterをまとめるラッパークラス  
Impoter本体はregisterImporter()関数によって登録され、この関数はREGISTER_IMPORTERのdefineによって呼び出される  

現状登録されているImpoterはAssimpImporter、PythonImporter、SceneImporter  

### AssimpImporter
[assimp](https://github.com/assimp/assimp)というAPIによるモデルファイルインポートクラス  
これだけで大体のものが読み込める  
（ただしドキュメントにもあるようにレイトレに関するマテリアル情報とかは不十分っぽい）

使い方は[assimpのサイト](https://assimp-docs.readthedocs.io/en/latest/)とか[OpenGL学習サイトのAssimpページ](https://learnopengl.com/Model-Loading/Assimp)とか[OpenGLによる実装例](http://ogldev.atspace.co.uk/www/tutorial38/tutorial38.html)とか  

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTY0MDgyNjg0LC02NDY0MTg5OCwtMTgwOT
M4NDc0LC0xNTkzNDUxMDIwXX0=
-->