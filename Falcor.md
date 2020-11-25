# Falcorプロジェクト
場合によってはビルドでC2220エラーがでるので以下の方法で対処
1.  プロジェクトのプロパティを開く
2.  `構成プロパティ`  >  `C/C++`  >  `全般`を開く
3.  `警告をエラーとして扱う`を`いいえ（/WX-）`に変更

参考 : https://qiita.com/ledsun/items/a3bd7691b86389073c7a

これ単体では動かず（dll作成用プロジェクト）、このdllをほかのプロジェクトで使い描画処理を実装している  
サンプルフォルダのテンプレートを見ればわかるように、Renderer.hのIRendererインターフェースを継承したクラスを用意し、Winmain関数を作り、Sample.h, cppで実装してあるstatic run()関数を実行することにより描画できる  

# メイン

## Sample.h, cpp
描画データと描画以外の処理データ構造のインターフェース（IFramework）を継承して一通りの描画以外の処理をできるようにするもの、またはそのサンプル実装  
IRendererを継承したクラスとSampleConfigを生成し、Sample.h, cppで実装してあるstatic run()関数に渡すことにより描画処理を行っている  
run()実行時にgpFrameworkにこのインスタンスが


## Renderer.h
描画処理のみを抽出したインターフェース（IRenderer）とそのデータやそれ以外の処理のインターフェース（IFramework）  
サンプルプロジェクトにあるテンプレートはIRendererを継承し描画処理のみを実装し、そのほかの処理はIFrameworkを継承したSample.h, cppに任せてある  

## FalcorConfig.h
ファルカーノコンフィグ  
プロファイルとログ出力の設定のみ  

# サブ

## Framework.h, cpp
Frameworkという名前だがIFrameworkがあるわけではない（IFrameworkはRenderer.hにある。なぞ）  
シェーダー用（ほかでも使う？）のEnumがまとめられている  
また、clampやalignmentなどのちょっとした汎用関数もある  
cppの方はComparisonFuncに対するpybind11処理のみ  
pybind11周りの説明 : https://github.com/NVIDIAGameWorks/Falcor/blob/master/Docs/Usage/Render-Passes.md


## ScriptBindings.h, cpp
おそらくpybind11のヘルパー  
見るのはぴぃちょんとぴぃびんど11を勉強した後で  
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE2NDQ0NzI3MTIsMTI2MTM4MzEyLC01MT
YxNzU1NjUsMTIzMzY5Njk2NSwtMTUyNDg5MTEzNywxMTEyMTE0
MjQ0LDQyNDMzNjIxNCwtNDIwODQwOTMxLDI3OTk5NDE2NSwtMT
A3NTQzNDMwMywtMTIzNzgxMjM2OCwyMjcwNzc3MzgsLTE1MTM3
MzM3ODYsLTE2MTY0MDM5NzIsLTE1MTkzMDU5MzcsLTEwMDI2ND
M4NCw4OTEwMTIwNDhdfQ==
-->