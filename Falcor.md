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
run()実行時にグローバル変数gpFrameworkにこのインスタンスを格納、これを使ってrun()で作ったSampleインスタンスにアクセスでき、描画部分以外の処理ができる  

### IRendererかんれ

## Renderer.h
描画処理のみを抽出したインターフェース（IRenderer）と、そのためのデータ管理やそれ以外の処理のインターフェース（IFramework）の2つが入っている  
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


## ScriptBindings.h, cpp
おそらくpybind11のヘルパー  
見るのはぴぃちょんとぴぃびんど11を勉強した後で  
pybind11周りの説明 : https://github.com/NVIDIAGameWorks/Falcor/blob/master/Docs/Usage/Render-Passes.md  
https://buildersbox.corp-sansan.com/entry/2019/12/09/110000  

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwNjc0ODM4NjAsLTI3NjEwMzEwLDE0MD
E0MTc0OTEsLTE1Nzk5NDA0NDYsMTQyNzQwMzc0NSwxMjYxMzgz
MTIsLTUxNjE3NTU2NSwxMjMzNjk2OTY1LC0xNTI0ODkxMTM3LD
ExMTIxMTQyNDQsNDI0MzM2MjE0LC00MjA4NDA5MzEsMjc5OTk0
MTY1LC0xMDc1NDM0MzAzLC0xMjM3ODEyMzY4LDIyNzA3NzczOC
wtMTUxMzczMzc4NiwtMTYxNjQwMzk3MiwtMTUxOTMwNTkzNywt
MTAwMjY0Mzg0XX0=
-->