
# DXRのための低レイヤー処理関連（コマンドリストやコンパイルやハンドルやリソース周り）
DXRを使っているため、ほかの処理と違う手続きとかが必要となる  
DXRについては[NVIDIAGameWorks / DxrTutorials](https://github.com/NVIDIAGameWorks/DxrTutorials)（低レベルレイヤーから実装していくチュートリアル、三角形と影を出すくらい）とか、[NVIDIAGameWorks/GettingStartedWithRTXRayTracing](https://github.com/NVIDIAGameWorks/GettingStartedWithRTXRayTracing)（Falcorとそれを使いやすいようにしたラップクラスを用いてAOとかGIとか実装する。一方低レベルには触れない）とか、[Rey Tracing Gems : CHAPTER 3 Introduction to DirectX Raytracing](https://www.realtimerendering.com/raytracinggems/)（DXRの低レベル部分をざっくり説明）とか  



Falcorではおおよそ以下の部分の一部でDXR用実装が必要となる  
[Falcorプロジェクト内のBasePasses関連](https://github.com/tktkq9/falcor_memo/blob/main/BasePasses%E9%96%A2%E9%80%A3.md)
[ProgramVars, ParameterBlock, ProgramReflection関連](https://github.com/tktkq9/falcor_memo/blob/main/ProgramVars%2C%20ParameterBlock%2C%20ProgramReflection%E9%96%A2%E9%80%A3.md)、  
[ProgramVersion, Program, Shaderファイル関連](https://github.com/tktkq9/falcor_memo/blob/main/ProgramVersion%2C%20Program%2C%20Shader%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E9%96%A2%E9%80%A3.md)  
ただし、GraphicsStateといったものなどが無かったりする（代わりにGraphicsStateObjectに該当するRtStateObjectはRtProgramで管理、作成される。RenderContextでもraytrace(()でGraphicsStateの代わりにRtProgramが渡されているように、RtProgramはGraphicsStateも兼ねている）   

## Sceneでのraytrace()など
TLAS、BLASはここで作成される  
そしてここで作成されキャッシュされたTLASなどが、Sceneでのraytrace()の時にsetRaytracingShaderData()関数により設定される  
そして、RenderContextのraytrace()が呼ばれ、そこでほかのDXR用コマンドリスト処理が行われる  

このような実装から、おそらくDXR用パスを作るときはRenderContextからではなくSceneクラスからraytrace()を呼ばなければいけないと思われる  

TLAS、BLASの作成、更新はSceneのupdate()が呼ばれた時と、ここで呼ばれているsetRaytracingShaderData()が呼ばれた時のみ  

## RenderContextでのraytrace()
DXR用のコマンドリスト実行部分  
Sceneのraytrace()関数から呼ばれる  
上にも書いたが、GraphicsStateの代わりにRtProgramが渡され、ここからRtStateObjectを取得する  
そのあと、RtProgramVarsのapply()によってシェーダーテーブルに変数のハンドルが格納され、変数とシェーダーのコマンドリスト設定が行われる  
あとは[NVIDIAGameWorks / DxrTutorials](https://github.com/NVIDIAGameWorks/DxrTutorials)のonFrameRender()と同じ処理、つまりDXRで必要なテンプレディスパッチ処置をしてレイトレ実行  

## RtProgram
Programのサブクラス、DXR用Programクラス  
ラスタライズとは違うエントリーポイント（RayGen、Miss、HitGroup）を使うので、そこら辺の情報を格納するためのDescはDXR用に新しく作り直されている（ただしProgramのDescもprivateメンバ変数として保持し、使える部分は再利用している）  

Programと違いGraphicsStateの機能も兼ねていて、GraphicsStateObjectに該当するRtStateObjectはこのクラスで作成されている  
getRtsoがGraphicsStateのgetGSO()に対応する  
（DXRで必要なのはTLAS、BLAS、UAVで、ラスタライズで必要なVaoやFboなどはいらないからわざわざGraphicsStateを作る必要はなかったのかも）  

それ以外ではlocalRootSignature、DXR用エントリーポイントによるRtEntryPointGroupKernels作成とかのProgramのDXR差分の実装となっている  
（mMaxPayloadSizeとmMaxAttributesSizeは自動設定されないっぽいのでそこは事前に自分で計算して渡す必要があるようだ）  

あとsetScene()でシーンを設定する処理があるので、TLAS、BLAS用に使うのかなと思ったが、使われている形跡がないし、TLAS、BLASはSceneクラスで作成更新コマンドリスト設定が行われるのでこれは多分消し忘れ  

### RtEntryPointGroupKernels
EntryPointGroupKernelsのサブクラス、DXR用EntryPointGroupKernels  
EntryPointGroupKernelsの他にexportName、localRootSignature、maxPayloadSize、maxAttributeSizeが格納されただけのもの  

ラスタライズの場合と同様にProgramKernelsに格納される  
RtProgramのgetRtso()で、  

    auto pProgramKernels = pProgramVersion->getKernels(pVars);
の時にRtProgramのcreateEntryPointGroupKernels()が呼ばれて作成される  

## RtStateObject
GraphicsStateObjectのDXR版  
create()でDXR用のグラフィックステートオブジェクトmApiHandleが作成される  
create()の呼び出しはRtProgramのgetRtso()で呼ばれる  

DXR用のパイプラインステートオブジェクトの生成は 
[dxrtutorials/Tutorials/04-RtPipelineState/](https://github.com/NVIDIAGameWorks/DxrTutorials/tree/master/Tutorials/04-RtPipelineState)参照（特にcreateRtPipelineState()関数部分）  
このcreate()とそこで使われているRtStateObjectHelper（ほぼこちらで作成される）でこれに相当する処理が行われている  

### RtStateObjectHelper
RtStateObjectのcreate()関数でのみ使われ、完了したら破棄される、RtStateObject作成ヘルパークラス  
[dxrtutorials/Tutorials/04-RtPipelineState/](https://github.com/NVIDIAGameWorks/DxrTutorials/tree/master/Tutorials/04-RtPipelineState)の各サブオブジェクトの作成部分と、createRtPipelineState()での

    d3d_call(mpDevice->CreateStateObject(&desc, IID_PPV_ARGS(&mpPipelineState)));
    
より上の部分の処理に相当し、各サブオブジェクトに必要なデータを渡しサブオブジェクトラッパークラスRtStateSubobjectBaseを個別に作成と配列への格納、それによるD3D12_STATE_OBJECT_DESCの作成を担当する  

各々データのサブオブジェクトの作られ方は違うので、それぞれRtStateSubobjectBaseを継承した別Structを作り、そのコンストラクトでサブオブジェクトを作成  
これがadd...()の時に呼ばれて、配列に格納されていく  

変更がない場合はDescの再利用できるようにmDirty管理もされてる  

### MAKE_SMART_COM_PTR

    MAKE_SMART_COM_PTR(ID3D12StateObjectProperties);
    
こんな表記があるが、これは3D12StateObjectPropertiesのCOMオブジェクトポインタークラスID3D12StateObjectPropertiesPtrを作るためのもの  
ここ以外でもちょいちょい使われている  

## ShaderTable
DXR特有のシェーダーテーブルクラス  
RtProgramVarsやD3D12RenderContext.cppのraytrace()で使われる  
シェーダーテーブルとは、各エントリーポイントに対しDXRシェーダー内でどのインデックスで呼び出すかを決めるためのインデックステーブル（またはエントリーポイント関数ポインター配列）みたいなもの  
必要最低限の実装をしている[DxrTutorials/Tutorials/05-ShaderTable/](https://github.com/NVIDIAGameWorks/DxrTutorials/tree/master/Tutorials/05-ShaderTable)とか、複数のエントリーポイントを考慮した[DxrTutorials / Tutorials / 13 - SecondRayType /](https://github.com/NVIDIAGameWorks/DxrTutorials/tree/master/Tutorials/13-SecondRayType)とか参照  
そして、Falcorでのテーブルレイアウトは以下のようになっている  

       +------------+---------+---------+-----+--------+---------+--------+-----+--------+--------+-----+--------+-----+--------+--------+-----+--------+
       |            |         |         | ... |        |         |        | ... |        |        | ... |        | ... |        |        | ... |        |
       |   RayGen   |   Ray0  |   Ray1  | ... |  RayN  |   Ray0  |  Ray1  | ... |  RayN  |  Ray0  | ... |  RayN  | ... |  Ray0  |  Ray0  | ... |  RayN  |
       |   Entry    |   Miss  |   Miss  | ... |  Miss  |   Hit   |   Hit  | ... |  Hit   |  Hit   | ... |  Hit   | ... |  Hit   |  Hit   | ... |  Hit   |
       |            |         |         | ... |        |  Mesh0  |  Mesh0 | ... |  Mesh0 |  Mesh1 | ... |  Mesh1 | ... | MeshN  |  MeshN | ... |  MeshN |
       +------------+---------+---------+-----+--------+---------+--------+-----+--------+--------+-----+--------+-----+--------+--------+-----+--------+

create()では何もせず、update()でシェーダーテーブル作成を行う  
行っていることは[DxrTutorials/Tutorials/05-ShaderTable/](https://github.com/NVIDIAGameWorks/DxrTutorials/tree/master/Tutorials/05-ShaderTable)とかのcreateShaderTable()における、エントリーポイントの数と各エントリーのサイズ、オフセットの計算だけを行っている    
そこから任意のルートシグネチャーとエントリーに対し自動で作成するように実装されている  
実際にシェーダーのアイデンティファーやルートシグネチャーをバッファーに割り当てる処理（とGPUへのアップロード処理flushBuffer()の実行）はRtProgramVarsのapply()とその中で呼ばれているapplyVarsToTable()で行っている  

## RtProgramVars
DXR用ProgramVars  
ShaderTableのテーブルレイアウトに沿った実装がなされている  

create()（とその中で呼ばれているinit()）でRayGen（おそらく1個、複数作れる実装になってはいるが）、Miss（N_ray個）、HitGroupのEntryPointGroupVars（N_ray * N_mash個）のEntryPointGroupVarsを作成、保持される  

残りはapply()とそこで呼ばれている関数のみとなっている（これはRenderContext::raytrace()の時に呼ばれる）  
まずShaderTableがなければ作成し、そのアップデートが必要ならアップデートする  
そこで各SubTableType（RayGen、Miss、HitGroup）ごとにapplyVarsToTable()が呼ばれ、シェーダーテーブルバッファーの設定が行われる  
最後に[ProgramVarsのapply()](https://github.com/tktkq9/falcor_memo/blob/main/ProgramVars%2C%20ParameterBlock%2C%20ProgramReflection%E9%96%A2%E9%80%A3.md#apply%E9%96%A2%E6%95%B0)の代わりに、グローバルルートシグネチャーに対するapplyProgramVarsCommon()を呼びだし、コマンドリストにルートシグネチャーとシェーダー変数のハンドルとそれに関する諸々を設定する（forGraphics = falseになっていることに注意）  

applyVarsToTable()の詳細は、  
SubTableTypeに対応するエントリーポイントすべてに対し、シェーダーテーブルバッファーpRecordを取得し、applyRtProgramVars()でその設定を行い、バージョン管理l用変数astObservedChangeEpochの更新を行う  

applyRtProgramVars()では、  
シェーダーテーブル設定特有の処理である、  
pRecordへのエントリーポイントのアイデンティファーの設定と、  
そのローカルルートシグネチャー変数に対応するハンドルの設定が行われる  
DXRではラスタライズと違って、シェーダー変数のハンドルはコマンドリストではなくShaderTableに設定される（つまりここでいうpRecordに設定される）  
この処理を行うにあたり、疑似コマンドリストクラスRtVarsCmdList（とそれを持つ疑似ContextクラスRtVarsContext）を使うことによって、applyProgramVarsCommon()（通常はルートシグネチャーと変数ハンドルをコマンドリストに設定する関数）でコマンドリストの代わりにpRecordにハンドルを設定するような実装となっている  

### RtVarsContext
ローカルルートシグネチャーに対応する変数をシェーダーテーブルに設定するための疑似Cotextクラス  
ラスタライズ用の処理applyProgramVarsCommon()で無理やりシェーダーテーブルに設定できるようにするために、このような~~回りくどい~~スマートな実装となっている<font color=#FFFFFF>さすがにコードにその説明コメント書いてほしいなああああああああ</font>  
RtProgramVars以外では使われていない（ローカルルートシグネチャー関連の設定はRtProgramVarsでしか行われていないっぽいので）  

ここで行っていることはRtVarsCmdListを生成し保持することと、  
バリアー処理はこのContextではなくDeviceクラスの方のRenderContextに任せるようにすることのみ  
このコンテキストではコマンドキューを実行することはないのでこのようにバリアーは他にまかせている  

### RtVarsCmdList
ローカルルートシグネチャーに対応する変数をシェーダーテーブルに設定するための疑似コマンドリストクラス  
行っていることはコマンドリストの設定ではなく、  
シェーダーテーブルのバッファーmpRootBaseへ変数ハンドルを設定している  
applyProgramVarsCommon()の変更なしに動くよう、通常コマンドリスト設定関数をオーバーライドし、シェーダーテーブルへの設定に差し替えている（あと、呼ばれないであろう処理はすべてshould_not_get_here()でアサートしてある）
もちろんRtVarsContextでのみ使われる  

# TALS、BLASについて
上でも書いたが、TALS、BLASはSceneクラスで作成更新設定管理されている  

TODO : SceneのTALS、BLASについて詳細  


<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQwMDQyNTE3XX0=
-->