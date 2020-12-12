# Samplesのプロジェクト


IRendererを継承したクラス（と使うシェーダー）のみを作成し、Sampleクラスに渡して描画するタイプの実装サンプル  
このクラス単体で描画できるが、Mogwaiでは使えない（Mogwai用のサンプル実装はRenderPassesのプロジェクトを参照）  

新しくこれ用のプロジェクトを作りたい場合は、Source/Samples/make_new_project.batを実行  

また、この実装をもとにした（IRendererを少し使いやすくしたラップクラスによって実装されている）、DXRチュートリアルもある  
[A Gentle Introduction To DirectX Raytracing](http://cwyman.org/code/dxrTutors/dxr_tutors.md.html)  
[github](https://github.com/NVIDIAGameWorks/GettingStartedWithRTXRayTracing)  

参考文献：[NVIDIA Falcor を使ってみる](https://shikihuiku.github.io/post/falcor_getting_started/  )

## ProjectTemplate
クリアしたレンダーターゲットを描画するだけ  
WinMain()とonFrameRender()の実装のみ  

WinMain()はこのSamples系プロジェクトのエントリー関数  
ProjectTemplateを作成し、Sample;;run()に渡してSampleクラスを作成  
これだけでSampleクラスが描画に必要なdx12系初期化やその他初期化をして、msgloop()によってon...()を実行し続けるようになる  
終了処理も全部やってくれる  

onFrameRender()はmsgl

## ShaderToy

## ModelViewer

## HelloDXR


<!--stackedit_data:
eyJoaXN0b3J5IjpbMTE5Nzc1NjY3MywtMTQ3NDU2NDg4MiwtMT
Q3NDcwMDIyLC0xNjQ0NTQ3MDU4LC0xNTg2Njk0NDQ5LDE3Nzg5
MTk1OTcsMzAyMjA1ODg3LDk0NjQ3ODI5Myw3NTY1NzI3ODldfQ
==
-->