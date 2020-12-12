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

WinMain()はこのプロジェクトのエントリー関数  


## ShaderToy

## ModelViewer

## HelloDXR


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTM2MzU3OTc2MiwtMTQ3NDcwMDIyLC0xNj
Q0NTQ3MDU4LC0xNTg2Njk0NDQ5LDE3Nzg5MTk1OTcsMzAyMjA1
ODg3LDk0NjQ3ODI5Myw3NTY1NzI3ODldfQ==
-->