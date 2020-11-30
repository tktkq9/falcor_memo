# Falcorプロジェクト
場合によってはビルドでC2220エラーがでるので以下の方法で対処
1.  プロジェクトのプロパティを開く
2.  `構成プロパティ`  >  `C/C++`  >  `全般`を開く
3.  `警告をエラーとして扱う`を`いいえ（/WX-）`に変更

参考 : https://qiita.com/ledsun/items/a3bd7691b86389073c7a

これ単体では動かず（dll作成用プロジェクト）、このdllをほかのプロジェクトで使い描画処理を実装している  
サンプルフォルダのテンプレートを見ればわかるように、Renderer.hのIRendererインターフェースを継承したクラスを用意し、Winmain関数を作り、Sample.h, cppで実装してあるstatic run()関数を実行することにより描画できる  

# メイン


## Renderer.h
描画処理のみを抽出したインターフェース（IRenderer）と、そのためのデータ管理やそれ以外の処理のインターフェース（IFramework）の2つが入っている  
サンプルプロジェクトにあるテンプレートはIRendererを継承し描画処理のみを実装し、そのほかの処理はIFrameworkを継承したSample.h, cppに任せてある  

## Sample.h, cpp
描画データと描画以外の処理データ構造のインターフェース（IFramework）を継承して一通りの描画以外の処理をできるようにするもの、またはそのサンプル実装  
IRendererを継承したクラスとSampleConfigを生成し、Sample.h, cppで実装してあるstatic run()関数に渡すことにより描画処理を行っている  
run()実行時にグローバル変数gpFrameworkにこのインスタンスを格納、これを使ってrun()で作ったSampleインスタンスにアクセスでき、描画部分以外の処理ができる  

### IRenderer関連のコード
run() → runInternal()でいろいろ初期化し、最後にこの処理をして描画している
    
        // Load and run
        mpRenderer->onLoad(getRenderContext());
        pBar = nullptr;

        mFrameRate.reset();
        mpWindow->msgLoop();

        mpRenderer->onShutdown();
        if (gpDevice) gpDevice->flushAndSync();
        mpRenderer = nullptr;

mpRendererが渡したIRenderer、mpWindow->msgLoop()は
windowのwhileループとそこでのSample::renderFrame()  の実行  
残りのpBar は描画する前に表示されるプログレスバー、mFrameRateはフレームレート管理、gpDevice->flushAndSync();は最後に残ったコマンドキューが終わるまで待つシャットダウン処理  
IRendererだけを見るとonLoad()1回のみ → onFrameRender()ループ（とonGuiRender()の実行） → ループから抜けたらonShutdown()の流れ  
残りのイベント系列は Window.h, cpp 側のコールバック処理で呼ばれる

## Window.h, cpp
window処理用  
windowの作成には GLFW API の GLFWwindow で簡略化されている  
これはOpenGL や Vulkan などの API だが、おそらくそれとは関係なくwindow作成のために使われているっぽい、多分（実際この window はDirectXでも使われている）  

create()呼び出しの時にコールバックを mpCallbacks に割り当て、  
主にキーやマウスやその他イベント処理のためのクラス ApiCallbacks で呼び出しを行っている  
Window クラス自体は作成、解放処理と msgLoop()処理、 あとは  ApiCallbacks, GLFWwindow 間のバインドを行っている  

## Device.h, cpp


## FalcorConfig.h
ファルカーノコンフィグ  
プロファイルとログ出力の設定のみ  

# サブ

## Framework.h, cpp
Frameworkという名前だがIFrameworkがあるわけではない（IFrameworkはRenderer.hにある。なぞ）  
シェーダー用（ほかでも使う？）のEnumがまとめられている  
また、clampやalignmentなどのちょっとした汎用関数もある  
cppの方はComparisonFuncに対するpybind11処理のみ  


## ScriptBindings.h, cpp, Scripting.h, cpp, Dictionary.h
おそらくpybind11のヘルパー  
見るのはぴぃちょんとぴぃびんど11を勉強した後で  
pybind11周りの説明 : https://github.com/NVIDIAGameWorks/Falcor/blob/master/Docs/Usage/Render-Passes.md  
https://buildersbox.corp-sansan.com/entry/2019/12/09/110000  

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTE2MTQyOTEyLDkxNTA0MjY2MSw4NTA3Mj
AyMCwtMTk1OTE5Njk2MSwxMjI5MzU5MTg4LDE1MjE0OTY3MDUs
LTE2ODkxNzI2MTYsLTY4OTU0MTkxOCwxNzE3MjA3ODQ2LDI5MD
IxMDkwNywxNzg5NTMzMDQyLC0xOTAxMDM1MzY4LC0yMDY3NDgz
ODYwLDE0MDE0MTc0OTEsLTE1Nzk5NDA0NDYsMTQyNzQwMzc0NS
wxMjYxMzgzMTIsLTUxNjE3NTU2NSwxMjMzNjk2OTY1LC0xNTI0
ODkxMTM3XX0=
-->