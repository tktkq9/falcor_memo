# Samplesのプロジェクト


IRendererを継承したクラス（と使うシェーダー）のみを作成し、Sampleクラスに渡して描画するタイプの実装サンプル  
このクラス単体で描画できるが、Mogwaiでは使えない（Mogwai用のサンプル実装はRenderPassesのプロジェクトを参照）  

新しくこれ用のプロジェクトを作りたい場合は、Source/Samples/make_new_project.batを実行  

また、この実装をもとにした（IRendererを少し使いやすくしたラップクラスによって実装されている）、DXRチュートリアルもある  
[A Gentle Introduction To DirectX Raytracing](http://cwyman.org/code/dxrTutors/dxr_tutors.md.html)  
[github](https://github.com/NVIDIAGameWorks/GettingStartedWithRTXRayTracing)  

参考文献：[NVIDIA Falcor を使ってみる](https://shikihuiku.github.io/post/falcor_getting_started/  )

## ProjectTemplate
単色でクリアしたレンダーターゲットを描画するだけ  
WinMain()とonFrameRender()とonGuiRender()の実装のみ  

WinMain()はこのSamples系プロジェクトのエントリー関数  
ProjectTemplateとSampleConfig（ウィンドウとデバイスの設定）を作成し、Sample;;run()に渡してSampleクラスを作成  
これだけでSampleクラスが描画に必要なdx12系初期化やその他初期化をして、ゲームやリアルタイム描画でよく使われているmsgloop()をおこないマイフレーム描画をし続けるようになる  
ある程度のキー操作も準備されており（左上にでるサブウィンドウにヘルプがある）、終了処理もやってくれる  

onFrameRender()はmsgloop()で呼ばれる描画関数  
Sampleクラスが用意した、描画に関するリソースやらコマンドリスト実行やらを管理するRenderContextクラスが渡されるので、これに実行したコマンドなどを入れて描画設定を行う  
このプロジェクトでは

    pRenderContext->clearFbo(pTargetFbo.get(), clearColor, 1.0f, 0, FboAttachmentType::All);
    
によってレンダーターゲットをクリアしているだけ  
FboはFrameBuffaerObject。この名前はOpenGLで使われている名前で、dx12ではスワップチェインに設定しているレンダーターゲットバッファー的なやつに対応  

onGuiRender()は左上にでるやつ用  
imguiによって実装されており、渡されてくるGuiクラスはimguiを実装しやすくしたラップクラス  
実装内容としてはgpFramework->renderGlobalUI(pGui);が実行したときの「Click Here」のより上の部分で、あとのコードは「Click Here」の実装のみ  

## ShaderToy
WinMain()とonLoad()とonFrameRender()の実装のみ  
その他にも実装されているように見えるが実際は何もやってないに等しいので無視していい（はず）  

onLoad()はmsgLoop()前の初期化中に呼ばれる  
FullScreenPassという多分ピクセルシェーダーだけ表示する用レンダーパスにシェーダーファイルを渡して、レンダーパス作成。これを使ってonFrameRender()で描画処理を行う  
RasterizerState, DepthStencilState, BlendState, Samplerを作成しているがこれは意味なし（消しても動いた。おそらくこれをもとに拡張するときに必要なら使う用）  



### Toy.ps.slang
ShaderToyプロジェクトのピクセルシェーダー  
slangは [HLSL ベースの新しいシェーダ言語](http://masafumi.cocolog-nifty.com/masafumis_diary/2018/11/hlsl-slang-8752.html)  
TODO : slangのお勉強  


### FullScreenPass.h, cpp



## ModelViewer

## HelloDXR


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTgzNjUzMzU1MiwtMzY1MTkyMiwtMTA5NT
U1NjMwMywtMTAxNzk5MjkzOSwtMTI5NzgyNjczLDgyNDU3MDE3
MywxMDYwMzM2MDk5LC0xNDc0NzAwMjIsLTE2NDQ1NDcwNTgsLT
E1ODY2OTQ0NDksMTc3ODkxOTU5NywzMDIyMDU4ODcsOTQ2NDc4
MjkzLDc1NjU3Mjc4OV19
-->