# Falcorプロジェクト
場合によってはビルドでC2220エラーがでるので以下の方法で対処
1.  プロジェクトのプロパティを開く
2.  `構成プロパティ`  >  `C/C++`  >  `全般`を開く
3.  `警告をエラーとして扱う`を`いいえ（/WX-）`に変更

参考 : https://qiita.com/ledsun/items/a3bd7691b86389073c7a

これ単体では動かず（多分dll作成用プロジェクト）、このdllを使い描画処理を実装しているっぽい  
サンプルフォルダのテンプレートを見ればわかるように、Renderer.hのIRendererインターフェースを継承したクラスを用意し、Winmain関数を作り、Sample.h, cppで実装してあるstatic run()関数を実行することにより描画できる  

# メイン

## Sample.h, cpp
描画データと描画以外の処理データ構造のインターフェース（IFramework）を継承して一通りの描画以外の処理をできるようにするもの、またはそのサンプル実装  
IRendererを継承したクラスとSampleConfigを生成し、Sample.h, cppで実装してあるstatic run()関数に渡すことにより描画処理を行っている  


## Renderer.h
描画処理のみを抽出したインターフェース（IRenderer）とそのデータやそれ以外の処理のインターフェース（IFramework）  
サンプルプロジェクトにあるテンプレートはIRendererを継承し描画処理のみを実装し、そのほかの処理はIFrameworkを継承したSample.h, cppに任せてある  

## Framework.h, cpp
Frameworkという名前だがIFrameworkがあるわけではない（Renderer.hにある）  
シェーダー用（ほかでも使う？）のEnumがまとめられている  
また、clampやalignmentなどのちょっとした汎用関数もある  
cppのほうではComparisonFuncに対するpybind11処理のみ  
https://github.com/NVIDIAGameWorks/Falcor/blob/master/Docs/Usage/Render-Passes.md

# サブ

## ScriptBindings.h, cpp
おそらくpybind11のヘルパー  
あとでpythonとpybind11を調べてから見る  

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIxMTY4MzY1NDMsMTExMjExNDI0NCw0Mj
QzMzYyMTQsLTQyMDg0MDkzMSwyNzk5OTQxNjUsLTEwNzU0MzQz
MDMsLTEyMzc4MTIzNjgsMjI3MDc3NzM4LC0xNTEzNzMzNzg2LC
0xNjE2NDAzOTcyLC0xNTE5MzA1OTM3LC0xMDAyNjQzODQsODkx
MDEyMDQ4XX0=
-->