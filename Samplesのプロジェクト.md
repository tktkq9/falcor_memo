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
ProjectTemplateを作成し、Sample;;run()に渡してSampleクラスを作成  
これだけでSampleクラスが描画に必要なdx12系初期化やその他初期化をして、msgloop()によってon...()を実行し続けるようになる  
終了処理も全部やってくれる  

onFrameRender()はmsgloop()で呼ばれる描画関数  
Sampleクラスが用意した、描画に関するリソースやらコマンドリスト実行やらを管理するRenderContextクラスが渡されるので、これに実行したコマンドなどを入れて描画設定を行う  
このプロジェクトでは

    pRenderContext->clearFbo(pTargetFbo.get(), clearColor, 1.0f, 0, FboAttachmentType::All);
    
によってレンダーターゲットをクリアしているだけ  
FboはFrameBuffaerObject。この名前はOpenGLで使われている名前で、dx12ではスワップチェインに設定しているレンダーターゲットバッファー的なやつに対応  

onGuiRender()は実行すると左上にでるやつ  
imguiによって実装されており、渡されてくるGuiクラスはimguiを実装しやすくしたラップクラス  
実装内容としてはgpFramework->renderGlobalUI(pGui);が実行したときの「Click Here」のより上の部分で、あとはClick Hereの実装のみ  

## ShaderToy



## ModelViewer

## HelloDXR


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTYzMTg5NDY1OSw4MjQ1NzAxNzMsMTA2MD
MzNjA5OSwtMTQ3NDcwMDIyLC0xNjQ0NTQ3MDU4LC0xNTg2Njk0
NDQ5LDE3Nzg5MTk1OTcsMzAyMjA1ODg3LDk0NjQ3ODI5Myw3NT
Y1NzI3ODldfQ==
-->