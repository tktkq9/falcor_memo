# Falcorプロジェクト
場合によってはビルドでC2220エラーがでるので以下の方法で対処
1.  プロジェクトのプロパティを開く
2.  `構成プロパティ`  >  `C/C++`  >  `全般`を開く
3.  `警告をエラーとして扱う`を`いいえ（/WX-）`に変更

参考 : https://qiita.com/ledsun/items/a3bd7691b86389073c7a

# Core/

## Sample.h, cpp
描画処理データ構造のインターフェース（IFramework）を継承して一通りの描画処理をできるようにするもの、またはそのサンプル実装
IRendererを継承したクラスとSampleConfigを生成し、static run()関数に渡すことにより

### a

## Renderer.h
描画処理のインターフェース（IRenderer）と描画データ構造やそれ以外の処理のインターフェース（IFramework）  
SampleプロジェクトにあるテンプレートはIRendererを継承し基本描画処理のみをじｓ

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyMzc4MTIzNjgsMTUzMTgwMjk5LDIyNz
A3NzczOCwtMTUxMzczMzc4NiwtMTYxNjQwMzk3MiwtMTUxOTMw
NTkzNywtMTAwMjY0Mzg0LDg5MTAxMjA0OF19
-->