# Scene関連

TODO : Flacorプロジェクト下のSceneディレクトリにあるやつ全部読む  

Falcorドキュメントでもざっくりまとめられている  
[Scene Creation](https://github.com/NVIDIAGameWorks/Falcor/blob/4.2-release/Docs/Usage/Scene-Creation.md)
[Scenes](https://github.com/NVIDIAGameWorks/Falcor/blob/4.2-release/Docs/Usage/Scenes.md)
[Scene Formats](https://github.com/NVIDIAGameWorks/Falcor/blob/4.2-release/Docs/Usage/Scene-Formats.md)




## Importer
ファイルからインポートするときの複数のImpoterをまとめるラッパークラス  
Impoter本体はregisterImporter()関数によって登録され、この関数はREGISTER_IMPORTERのdefineによって呼び出される  

現状登録されているImpoterはAssimpImporter、SceneImporter、PythonImporter  

### AssimpImporter
[assimp](https://github.com/assimp/assimp)というAPIによるモデルファイルインポートクラス  
これだけで大体のものが読み込める  
（ただしドキュメントにもあるようにレイトレに関するマテリアル情報とかは不十分っぽい）

使い方は[assimpサイトのドキュメント](https://assimp-docs.readthedocs.io/en/latest/)とか[OpenGL学習サイトのAssimpページ](https://learnopengl.com/Model-Loading/Assimp)とか[OpenGLによる実装例](http://ogldev.atspace.co.uk/www/tutorial38/tutorial38.html)とか  

### SceneImporter
fsceneファイル用インポーター  

fscene = Falcor scene  
Falcorに最初から入っているシーンや[ORCA](https://developer.nvidia.com/orca)で使われている  

### SceneImporter
pysceneファイル用インポーター  

pysceneもまたFalcorのオリジナル  
さまざまなモデルやライトやカメラなどの読み込みや設定をPythonで記述したもの  
使い方は[Python Scene Files](https://github.com/NVIDIAGameWorks/Falcor/blob/4.2-release/Docs/Usage/Scene-Formats.md#python-scene-files)  

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTM0ODk1MzIzM119
-->