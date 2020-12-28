# FlacorプロジェクトRenderPasses関連
TODO : NVAPIの理解　GraphicsStateObject
TODO : Programのslang処理の理解  Program  

## BaseGraphicsPassおよび関連するクラス

## BaseGraphicsPass
これ自体はGraphicsVars、GraphicsState、GraphicsProgramを生成と保持し、間接的にこれらのメンバ変数を実行するだけのクラス  
（サブクラスで利用されている）ParameterBlockSharedPtrによってShaderVarにアクセスできるようgetRootVar()が実装されている  
drawやdispatch処理さえないので、完全にサブクラス作成前提のインターフェース的な役割  


## GraphicsState
RenderContextのdrawInstancedなどに渡される2つのうちの1つでパイプラインステートオブジェクトのようなもの  
ルートシグネチャー、モデルのリソース、パイプライン、複数のビューポート、シザー、FBOなどGraphicsVars以外のものを設定、管理  
ブレンドやデプスステンシルやカリングなどの設定管理はGraphicsStateObjectの方に押し付けている  

また、パイプラインステートオブジェクトのラッパークラスであるGraphicsStateObjecの作成も担当しており、mpGsoGraphによる設定変更確認をもとにパイプラインオブジェクトの作成or再利用判定も行っている  

大体は直接このクラスを通して設定するが、ProgramKernelsとRootSignatureはgetGSO()によってGraphicsVarsからもらってくる  

### GraphicsStateObject
パイプラインステートオブジェクトのラッパークラス  
作成に必要なデータがこのクラスのDescにすべて格納され、このデータによりパイプラインステートオブジェクトがapiInit()により作成される（このコードはD3D12GraphicsStateObject.cppの方で定義されている）  

ちなみにapiInit()中に作成されているnvApi...はNVAPI用の処理  
TODO: NVAPIの理解  

### StateGraph
その名の通り状態遷移図  
汎用的に使えるが、GraphicsState、ComputeState、RtProgramといったパイプラインステートオブジェクト系でしか使われていない  

GraphicsState、ComputeState、RtProgramでの使われ方としては、NodeType（パイプラインステートオブジェクトのクラスが割り当てられる）の格納、検索、再利用  
描画に必要なデータ（例えばルートシグネチャーやFBOなど）をもとに各データに対応する状態遷移をwalk()を使って検索、なければ作成  
そして同じデータ割り当てに対応する状態遷移を見つけた場合は、設定したNodeTypeオブジェクト（例えばGraphicsStateではGraphicsStateObject）をgetCurrentNode()によって取得、つまりNodeTypeオブジェクトの再利用を行えるようにするために使われている  

### GraphicsProgram
GraphicsState用のProgramクラスのサブクラス  
中身はそのままProgramクラスを作るためのラッパーcreate関数系とpybind11のやつのみ  

ざっくりProgramクラスを説明すると、シェーダーを管理し、そこからシェーダーブロブやルートシグネチャーを保持するProgramKernelsを取得するためのもの  
詳しくは[ProgramVersion, Program, Shaderファイル関連 （TODO : リンク）](https://github.com/tktkq9/falcor_memo/tree/main)  

### GraphicsVars
TODO : 詳しく読む
シェーダーに渡す変数、バッファーなどを管理  
RenderContextのdrawInstancedなどに渡される2つのうちの1つ  



## ProgramVars, ParameterBlock

### ProgramVars
ParameterBlockのサブクラス  
ほぼほぼParameterBlockのラッパークラス  
ParameterBlockに対応するProgramReflectionとEntryPointGroupVars（これもまたParameterBlockのサブクラス）の配列を持つParameterBlockのようなもの  

これのサブクラスがGraphicsVarsやComputeVarsであり、これらはほぼほぼProgramVarsを作るためのcreate()関数を持つProgramVarsラッパークラスみたいなものとなっている  
ただしProgramVarsと違い特徴的なものとして、ルートシグネチャーと設定したシェーダー変数をコマンドリストに適用するapply()関数も実装されている  
<font color=#bfbfbf>ルートシグネチャーかコンテキストクラスにこれを渡すapply()関数を作るんじゃなく、こっちに作っちゃうんだーって感がある</font>  

### ParameterBlock
#### prepareDescriptorSets
まずはprepareResources()によってParameterBlockReflectionに対応するリソースそれぞれの処理、つまりバリアー処理が必要ならそれを、バッファーの大きさチェックと確保が必要ならそれを行う  
また

ParameterBlockReflectionに設定されているDescriptorSetInfoの数だけディスクリプターヒープと対応付け、つまりDescriptorSetの配列mSetsの中身の作成DescriptorSet::create()を行い、それがDefaultConstantBufferに対応する場合はbindIntoDescriptorSet()も行われる  

bindIntoDescriptorSet()はDefaultConstantBufferのView、つまりバッファーとハンドルの対応付け情報をなければ作成し、作ったDescriptorSetに設定する  
そのバッファーとハンドルの対応付けの際、現在の変数構造情報の更新とそのバッファーサイズ取得をupdateSpecialization()でSlangAPIを用いて行い、（サイズが足りていないか）バッファーがない場合はバッファーの（再）作成も行われる  

#### DescriptorSet
渡されたLayoutにディスクリプターヒープのハンドル  
create()の時にディスクリプターヒープにLayoutに沿ってハンドルとそのrangeを割り当て、そのハンドル情報を保存する  

### ShaderVar

## ProgramReflection

## ProgramVersion, Program, Shaderファイル関連
シェーダーの中身とそれをdx12用のシェーダーブロブやルートシグネチャーにするためのなんやかんや  

いろんなところで保持されているProgramにDefineしていって、ProgramVarsとProgramReflectionを渡すことによってシェーダーブロブやルートシグネチャーが作成され、それを保持したProgramKernels返され、このProgramKernelsを使って、GraphicsStateとかでパイプラインステートオブジェクトが作成されるという感じ  

### ProgramKernelsクラス
一つのslangシェーダーのコンパイルされたものと、それをdx12側で扱うための情報を保持するためのクラス  
情報は各エントリーポイントのブロブ（Shaderクラス）の配列とかProgramReflectionとか  

またその特性からRootSignatureを作成する情報もすべてあるので、ここでシェーダーブロブに対するRootSignatureが作成、保持される  

ちなみにこれはProgramクラスで作成、ProgramVersionで管理されている  
そのためProgramReflection以外の大体の情報はProgramVersionとProgramから作成され渡される  

### ProgramVersionクラス
Slangのコンパイルに必要な情報を一通り渡され保持し、この情報によってコンパイルされたものをProgramから取得し、対応するProgramKernelsの作成を行うためのクラス  

コンパイルと作成はgetKernels(ProgramVars const* pVars)で行われ、今まで作成したProgramKernelsは配列として保持し再利用できるようになっている  

slangシェーダー（Program）のインターフェースなどをProgramVarsのspecializationArgsによってシェーダーが変わったかどうか判定し、ProgramKernelsを（Version）管理するためのクラスであることからProgramVersionと名付けられているのだろうか  
ちなみにこれはProgramクラスで作成、管理されているので、最新バージョンのProgramってだけかもしれない（多分こっち）  

### Programクラス
slangシェーダーファイルとその中身の情報の保持、コンパイル処理、管理用クラス  
エントリーポイントやシェーダーバージョン、シェーダーファイル、そしてこのシェーダーに設定したDefineListを管理する  
DefineListが変更されたかどうかも監視し、再コンパイルが必要かどうかの情報も管理する  

ProgramKernelsに対してこちらはslangシェーダー寄りのクラス  
そのためslangヘッダーを用いたSlangに関するSession作成やらProfile設定やらコンパイルやらのための処理もこちらで一通り実装されている  
で、preprocessAndCreateProgramKernels()でそのコンパイルが行われ、対応するProgramKernelsが返される  

ちなみにSlangプロジェクトのサンプルでもコンパイルしたりその作成されたものを取得したりする部分はProgramという構造体でカプセル化されているので、多分そこからこのクラス名になったのだと思われる  

TODO : Programのslang処理の理解  

### Shaderクラス
Shaderという名を持つが、シェーダー本体はProgramクラスの方であり、こちらはどちらかというとエントリーポイントのブロブクラス  

ちなみにDefineListもこのクラス内に定義されているが、これは主にProgramのコンパイルで使われ、さらにShaderクラスでは使われてない。なぜここにあるのか  

あとShaderファイルにはComPtrテンプレートクラスもあるが、これもシェーダーのみのためのものではなくどちらかというとProgramクラスでよく使われている。なぜこあ  

もしかしたらこれらはSlang適用時の残留物であろうか（SlangはFalcorができた後に作成され、それなりに後のバージョンで適用された）  
こいつら全員Programファイルに吸収された方がいいと思う  

# BaseGraphicsPassのサブクラス

## FullScreenPass
画面に自動で四角形を描画し、ピクセルシェーダーのみを扱う場合のパス  

    using SharedPtr = ParameterBlockSharedPtr<FullScreenPass>;
によって以下のようにシェーダーの変数を

    mpMainPass["ToyCB"]["iResolution"] = float2(width, height);
    mpMainPass["ToyCB"]["iGlobalTime"] = (float)gpFramework->getGlobalClock().getTime();  
のように設定できるようになっている    

頂点シェーダー（とviewportMaskの立っているビットの数だけレンダーターゲットを作成するためのジオメトリシェーダー）とその頂点バッファーなどの作成と設定はFullScreenPassのコンストラクタで自動で行われる  
複数回呼ばれるのを想定して、頂点バッファーはgFullScreenDataによって再利用されている  

故に頂点とジオメトリシェーダーはすでに用意されている  
頂点シェーダー（FullScreenPass.vs.slang）は頂点とuvをそのまま流すだけ  
ジオメトリシェーダー（FullScreenPass.gs.slang）はviewportMaskの立っているビット分のレンダーターゲットを作成するためのもの（参考文献：[ジオメトリシェーダを使用した複数画面描画](https://sites.google.com/site/monshonosuana/directxno-hanashi-1/directx-107)、[DirectX12 でシングルパスキューブマップ描画](https://blog.techlab-xe.net/directx12-render-cubemap-singlepass/)）  


<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQxMzc3ODgxMCw5Mzc1OTczOTEsLTI5MT
A2OTUxMywtMTAxNTU2NjA1MiwxNjUzNjQzODgsLTU2MTk2NjU2
NCwzMzE2MzUyNzAsLTE0MTQ2OTkzOTUsLTY1Nzg3MjM3LDEzMz
I3Njk2NTYsLTc2NDAzNDYyMSw2MTA0MjIyODAsMTUzNDM3MjY2
Nyw2MTkzOTgwMDMsLTE0MzUxNTExOTQsMTEwNjcyNTEsMTAxMz
kyOTc2OCwtOTkxMzM4NzQyLC0yMTI5NDM4OTYxLC00MzEwMzAz
NDBdfQ==
-->