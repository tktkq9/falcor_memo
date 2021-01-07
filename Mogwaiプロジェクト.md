# Mogwaiプロジェクト
TODO : もう少し詳しく読む  

RenderGraphによるグラフのUIの表示と操作、及びその描画を行う  

pythonファイルからグラフを生成し、そのグラフの操作や変更、保存ができる  

グラフ操作など色々なUIはimguiで実装されており、主にFalcorプロジェクトのRenderGraphUIでその処理が実装されている  

一方Mogwaiプロジェクトは複数のRenderGraphを管理し、複数のUIを管理し、繋ぎ込みを行う持ち回りになっているっぽい  

## AppData

filesystemに関する部分の処理をまとめているクラス  

また、最近開いたファイルリストも管理する  

## Mogwai

Mogwai用IRendererの実装、及びMogwaiプロジェクトのエントリーポイント  

Mogwaiクラスというものはない  

RenderGraphを用いて描画処理を行う  

またmain（or Winmain）関数もcppに実装されいることから、Mogwaiプロジェクトはこれがエントリーポイントとなっており、  

Samplesディレクトリにあるプロジェクトと同じようにSample::run()関数によってRendererが実行される  

### Extension

### MogwaiScriptingファイル
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTkwNDMyNTA4OF19
-->