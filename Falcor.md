# Falcorプロジェクト
場合によってはビルドでC2220エラーがでるので以下の方法で対処
1.  プロジェクトのプロパティを開く
2.  `構成プロパティ`  >  `C/C++`  >  `全般`を開く
3.  `警告をエラーとして扱う`を`いいえ（/WX-）`に変更

参考 : https://qiita.com/ledsun/items/a3bd7691b86389073c7a

# Core/

## Sample.h, cpp
描画データと描画以外の処理データ構造のインターフェース（IFramework）を継承して一通りの描画処理をできるようにするもの、またはそのサンプル実装
IRendererを継承したクラスとSampleConfigを生成し、static run()関数に渡すことにより

### a

## Renderer.h
描画処理のインターフェース（IRenderer）と描画データ構造やそれ以外の処理のインターフェース（IFramework）  
SampleプロジェクトにあるテンプレートはIRendererを継承し描画処理のみを実装し、そのほかの処理はIFrameworkを継承したSample.h, cppに任せてある

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwODMxOTcwMiwtMTIzNzgxMjM2OCwyMj
cwNzc3MzgsLTE1MTM3MzM3ODYsLTE2MTY0MDM5NzIsLTE1MTkz
MDU5MzcsLTEwMDI2NDM4NCw4OTEwMTIwNDhdfQ==
-->