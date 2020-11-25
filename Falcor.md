# Falcorプロジェクト
場合によってはビルドでC2220エラーがでるので以下の方法で対処
1.  プロジェクトのプロパティを開く
2.  `構成プロパティ`  >  `C/C++`  >  `全般`を開く
3.  `警告をエラーとして扱う`を`いいえ（/WX-）`に変更

参考 : https://qiita.com/ledsun/items/a3bd7691b86389073c7a

# Core/

## Sample.h, cpp
描画処理データ構造のインターフェース（IFramework）を継承して一通りの描画処理をでき

### a

## Renderer.h
描画処理のインターフェース（IRenderer）とそのデータ構造のインターフェース（IFramework）

<!--stackedit_data:
eyJoaXN0b3J5IjpbMzk5MDYyMDM4LC0xNTEzNzMzNzg2LC0xNj
E2NDAzOTcyLC0xNTE5MzA1OTM3LC0xMDAyNjQzODQsODkxMDEy
MDQ4XX0=
-->