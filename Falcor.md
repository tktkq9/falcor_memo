# Falcorプロジェクト
場合によってはビルドでC2220エラーがでるので以下の方法で対処
1.  プロジェクトのプロパティを開く
2.  `構成プロパティ`  >  `C/C++`  >  `全般`を開く
3.  `警告をエラーとして扱う`を`いいえ（/WX-）`に変更

参考 : https://qiita.com/ledsun/items/a3bd7691b86389073c7a

# Core/

## Sample.h, cpp
描画処理データ構造のインターフェース（IFramework）を継承して一通りの描画処理をできるようにする

### a

## Renderer.h
描画処理のインターフェース（IRenderer）とそのデータ構造のインターフェース（IFramework）

<!--stackedit_data:
eyJoaXN0b3J5IjpbMjI3MDc3NzM4LDM5OTA2MjAzOCwtMTUxMz
czMzc4NiwtMTYxNjQwMzk3MiwtMTUxOTMwNTkzNywtMTAwMjY0
Mzg0LDg5MTAxMjA0OF19
-->