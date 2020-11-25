# Falcorプロジェクト
場合によってはビルドでC2220エラーがでるので以下の方法で対処
1.  プロジェクトのプロパティを開く
2.  `構成プロパティ`  >  `C/C++`  >  `全般`を開く
3.  `警告をエラーとして扱う`を`いいえ（/WX-）`に変更

参考 : https://qiita.com/ledsun/items/a3bd7691b86389073c7a

# Core/

## Sample.h, cpp

### a

## Renderer.h
描画処理のインターフェース（IRenderer）とその構造のインターフェース（）

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5MjQwMDMxNTgsLTE1MTM3MzM3ODYsLT
E2MTY0MDM5NzIsLTE1MTkzMDU5MzcsLTEwMDI2NDM4NCw4OTEw
MTIwNDhdfQ==
-->