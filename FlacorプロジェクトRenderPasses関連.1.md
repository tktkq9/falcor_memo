# FlacorプロジェクトRenderPasses関連
TODO : NVAPIの理解　GraphicsStateObject
TODO : Programのslang処理の理解  Program  

# BaseGraphicsPassおよび関連するクラス

# BaseGraphicsPass
これ自体はGraphicsVars、GraphicsState、GraphicsProgramを生成と保持し、間接的にこれらのメンバ変数を実行するだけのクラス  
（サブクラスで利用されている）ParameterBlockSharedPtrによってShaderVarにアクセスできるようgetRootVar()が実装されている  
drawやdispatch処理さえないので、完全にサブクラス作成前提のインターフェース的な役割  


# GraphicsState
RenderContextのdrawInstancedなどに渡される2つのうちの1つでパイプラインステートオブジェクトのようなもの  
ルートシグネチャー、モデルのリソース、パイプライン、複数のビューポート、シザー、FBOなどGraphicsVars以外のものを設定、管理  
ブレンドやデプスステンシルやカリングなどの設定管理はGraphicsStateObjectの方に押し付けている  

また、パイプラインステートオブジェクトのラッパークラスであるGraphicsStateObjecの作成も担当しており、mpGsoGraphによる設定変更確認をもとにパイプラインオブジェクトの作成or再利用判定も行っている  

大体は直接このクラスを通して設定するが、ProgramKernelsとRootSignatureはgetGSO()によってGraphicsVarsからもらってくる  

## GraphicsStateObject
パイプラインステートオブジェクトのラッパークラス  
作成に必要なデータがこのクラスのDescにすべて格納され、このデータによりパイプラインステートオブジェクトがapiInit()により作成される（このコードはD3D12GraphicsStateObject.cppの方で定義されている）  

ちなみにapiInit()中に作成されているnvApi...はNVAPI用の処理  
TODO: NVAPIの理解  

## StateGraph
その名の通り状態遷移図  
汎用的に使えるが、GraphicsState、ComputeState、RtProgramといったパイプラインステートオブジェクト系でしか使われていない  

GraphicsState、ComputeState、RtProgramでの使われ方としては、NodeType（パイプラインステートオブジェクトのクラスが割り当てられる）の格納、検索、再利用  
描画に必要なデータ（例えばルートシグネチャーやFBOなど）をもとに各データに対応する状態遷移をwalk()を使って検索、なければ作成  
そして同じデータ割り当てに対応する状態遷移を見つけた場合は、設定したNodeTypeオブジェクト（例えばGraphicsStateではGraphicsStateObject）をgetCurrentNode()によって取得、つまりNodeTypeオブジェクトの再利用を行えるようにするために使われている  

## GraphicsProgram
GraphicsState用のProgramクラスのサブクラス  
中身はそのままProgramクラスを作るためのラッパーcreate関数系とpybind11のやつのみ  

ざっくりProgramクラスを説明すると、シェーダーを管理し、そこからシェーダーブロブやルートシグネチャーを保持するProgramKernelsを取得するためのもの  
詳しくは[ProgramVersion, Program, Shaderファイル関連 （TODO : リンク）](https://github.com/tktkq9/falcor_memo/tree/main)  

## GraphicsVars
TODO : 詳しく読む
シェーダーに渡す変数、バッファーなどを管理  
RenderContextのdrawInstancedなどに渡される2つのうちの1つ  



# ProgramVars, ParameterBlock, ProgramReflection関連

## ProgramVars
ParameterBlockのサブクラス  
ほぼほぼParameterBlockのラッパークラス  
ParameterBlockに対応するProgramReflectionとEntryPointGroupVars（これもまたParameterBlockのサブクラス）の配列を持つParameterBlockのようなもの  

これのサブクラスがGraphicsVarsやComputeVarsであり、これらはほぼほぼProgramVarsを作るためのcreate()関数を持つProgramVarsラッパークラスみたいなものとなっている  
ただしProgramVarsと違い特徴的なものとして、ルートシグネチャーと設定したシェーダー変数をコマンドリストに適用するapply()関数も実装されている  
<font color=#bfbfbf>ルートシグネチャーかコンテキストクラスにこれを渡すapply()関数を作るんじゃなく、こっちに作るのってどうなの</font>  

### apply()関数
コマンドリストにルートシグネチャーとParameterBlockの変数に対応するハンドルを設定するための関数  
以下はその全体的な流れ  

まず、applyProgramVarsCommon()でコマンドリストにルートシグネチャーをセットし、bindRootSetsCommon()に移る  

bindRootSetsCommon(()ではpVars->prepareDescriptorSets(pContext)にParameterBlockに割り当てた変数構造に対応するリソースのバリアーや確保、ディスクリプターハンドルとの対応付けを行う  
そのディスクリプターハンドルの情報がParameterBlockが持つDescriptorSetの配列mSetsに確保される  

そしてbindParameterBlockSets()、bindParameterBlockRootDescs()に移り、コマンドリストに対してSetGraphicsRootShaderResourceView()などのレジスターとハンドルの対応付けが行われる  
また、ParameterBlockの中に複数のサブParameterBlockことから再帰的に呼ばれていき、ParameterBlockがなくなるまでハンドルが割り当てられていく  

bindParameterBlockRootDescs()はUAV, SRVのセット  
bindParameterBlockSets()はそれ以外のセット  
となっている  

## ParameterBlockSharedPtrクラス
ParameterBlockとShaderVarを通して、シェーダー変数の設定を辞書形式で行えるようにするためのクラス  
例えばFullScreenPassでは

    using SharedPtr = ParameterBlockSharedPtr<FullScreenPass>;
で宣言され（getRootVar()の実装が必要）、ほかのクラスで

    FullScreenPass::SharedPtr       mpMainPass;
と定義することにより

    mpMainPass["ToyCB"]["iResolution"] = float2(width, height);
    mpMainPass["ToyCB"]["iGlobalTime"] = (float)gpFramework->getGlobalClock().getTime();  
のように設定できるようになる  

## ParameterBlock
シェーダーに割り当てるための変数を設定、保持しておくためのクラス  

BindLocationによる変数参照はParameterBlockで行うが、
変数名によって参照する場合はShaderVarクラスで対応する  
正確には変数にアクセスしようとするたびShaderVarを一時的に作成し、ShaderVarで対応するBindLocationを取得し、そのBindLocationをParameterBlockに戻して変数割り当てを行う  

わざわざShaderVarを介する理由は、以下でもまとめてあるが、BindLocationによる変数取得処理を外部から隠すためである  


### prepareDescriptorSets()関数
最初にupdateSpecialization()でParameterBlockReflectionのSlang情報を更新  
そして以下のprepareDescriptorSets()に移る  

まずはcheckForIndirectChanges()でParameterBlockReflectionの中にある複数のサブParameterBlockReflectionに対し、更新が必要かチェック、必要ならその印をつけてあとで更新させるようにする  

次に、prepareResources()によってParameterBlockReflectionに対応するリソースそれぞれの処理、つまりバリアー処理が必要ならそれを、バッファーの大きさチェックと確保が必要ならそれを行う  
また、サブParameterBlockReflectionの更新もここで行う  
ただし、サブではないメインParameterBlockReflectionのDefaultConstantBufferは例外で、後で実行されるbindIntoDescriptorSet()でバッファー更新を行うので、ここでは更新が必要かチェックを入れるのみ  

また場合によってはParameterBlockReflectionの中に
ParameterBlockReflectionに設定されているDescriptorSetInfoの数だけディスクリプターヒープと対応付け、つまりDescriptorSetの配列mSetsの中身の作成DescriptorSet::create()を行い、それがDefaultConstantBufferに対応する場合はbindIntoDescriptorSet()も行われる  

bindIntoDescriptorSet()はDefaultConstantBufferのView、つまりバッファーとハンドルの対応付け情報をなければ作成し、作ったDescriptorSetに設定する  
そのバッファーとハンドルの対応付けの際、現在の変数構造情報の更新とそのバッファーサイズ取得をupdateSpecialization()でSlangAPIを用いて行い、（サイズが足りていないか）バッファーがない場合はバッファーの（再）作成も行われる  
ただし、ここでのbindIntoDescriptorSet()ではすでにprepareDescriptorSets()で

#### DescriptorSet
渡されたLayoutに対応するディスクリプターヒープのハンドル  
create()でLayoutに沿ってディスクリプターヒープとrange分のハンドルとそのを割り当て、そのハンドル情報を保存する  

## ShaderVar
外部用ParameterBlockクラス  
ParameterBlockSharedPtrで  

    mpMainPass["ToyCB"]["iResolution"] = float2(width, height);
    mpMainPass["ToyCB"]["iGlobalTime"] = (float)gpFramework->getGlobalClock().getTime();  
と書けるようoperator実装がされている  

詳細としては、  
ParameterBlockの変数にアクセスのためのオフセット  

    BindLocation =  ParameterBlockReflection::BindLocation ＝ ParameterBlockReflection::TypedShaderVarOffset
を変数名から取得し、そのBindLocationを用いてParameterBlockの対応する変数にアクセスする処理を担っている  
ようするに直感的には扱えないBindLocationを外部（ParameterBlockSharedPtr）から隠すためのクラスでもある  

また、外部用ParameterBlockとして機能するために、オフセットが絡まない関数に対するラッパー関数も一通り実装されている  


## ProgramReflectionファイル
変数名からオフセットを取得する際に利用されるクラス群  
要するに、ShaderVarとParameterBlockの橋渡しを行うためのもの  

TypedShaderVarOffsetが名前からオフセットを取得するためのクラス  

また、シェーダーで定義した変数のレイアウト情報などもProgramReflectionから取得できる  

### ShaderVarOffsetまとめ
基本的に外部で扱われるのはTypedShaderVarOffset  
その他のクラスはTypedShaderVarOffsetを構成するためのものとなっている  

- UniformShaderVarOffset
  - Uniform変数（GLSL用語、HLSLではコンスタントバッファーに対応）のtype/buffer/block用オフセット（TODO : type/buffer/blockとはなにか）  
- ResourceShaderVarOffset
  - リソース用オフセット  
- ShaderVarOffset
  - UniformShaderVarOffsetとResourceShaderVarOffsetをまとめたもの  
- TypedShaderVarOffset 
  - ShaderVarOffsetとReflectionTypeをまとめたもの。以下詳細  

#### TypedShaderVarOffset
ReflectionTypeを持ったShaderVarOffsetのサブクラス  
ShaderVarで変数にアクセスするために作られるオフセットクラス  
TypedShaderVarOffset作成時に渡されるReflectionTypeからの情報によりオフセットが取得、計算される  

オフセットの取得は

    operator[](const std::string& name)
  によって取得され、ReflectionTypeがReflectionStructTypeの時のみ機能する  
（つまり、変数名によるオフセット取得は構造体タイプに対してのみ行われ、ShaderVarは構造体のみしかアクセスできないようになっている？）  
変数名に対応する変数情報はReflectionTypeが持っており、TypedShaderVarOffsetでは、このReflectionTypeからもらってきた変数情報をオフセットに加工する役割を果たしている  

### ReflectionType系まとめ
各タイプに対する変数名に対応する変数情報が格納されたもの  
ParameterBlockが持っているParameterBlockReflectionから取得される  

メンバ変数の一つである[TypeLayoutReflection](https://github.com/shader-slang/slang/blob/master/docs/api-users-guide.md#type-layouts)はSlangAPIの構造体であり、シェーダー側で「宣言」したStruct、ParameterBlock, ConstantBufferなどの情報を取得することができる  
また、その中にある変数のVariableLayoutReflectionもそれぞれ取得することができる  

ReflectionType系クラスの中に複数のReflectionType系クラスがある再帰的な構造となっている  
例えばReflectionStructTypeのなかに基本型（Intとかfloat3x4とか）に対応するReflectionBasicTypeが複数入っているとかそんな感じ  
これを、reflectVariable系関数で再帰的に作成する  

以下概要
- ReflectionType
  - 以下のクラスのインターフェース。reflectType()の中身や、create()がないことからわかるように、このクラス自体のインスタンスが作られることはなくKindに関連するクラスのみが作られる
- ReflectionArrayType
  - 配列に対応するReflectionType。配列の要素数、配列の要素間のサイズ（アラインメントを考慮したサイズで、必ずしも要素のサイズではない）、要素のReflectionType情報を持つ
- ReflectionStructType
  - Structに対応するReflectionType。struct内部の変数やそのReflectionType、オフセットなど様々な情報の管理を行う（詳細は後の方で）
- ReflectionBasicType
  - intやFloat3x4など標準型に対応するReflectionType。その特性から、ReflectionBasicTypeが子ReflectionTypeを持つことはない
- ReflectionResourceType
  - Texture、Sampler、様々なBufferに対応するReflectionType。StructuredBufferとConstantBuffer以外はこれが終端となるが、StructuredBufferの場合はその中身のReflectionTypeも再帰的に作成と保持、ConstantBufferの場合はさらに対応するParameterBlockReflectionも再帰的に作成して保持している
- ReflectionInterfaceType
  - slangで利用可能となったインターフェースに関するReflectionType。ReflectionPathのpDeferredに対応するインターフェースをspecializeしたあとのParameterBlockReflectorを保持する

#### ReflectionStructType
シェーダーのStructに対応するReflectionType  
構造体なので、様々なReflectionTypeがこのクラスに格納される  
またReflectionStructType作成の際にreflectVariable()も実行されReflectionVarの作成と格納も行われる  

ReflectionVarの格納はaddMember()で行われ、
新しいメンバ変数の名前と対応するインデックスのペアがmNameToIndexに追加、  mMembersにReflectionVarの追加とメンバ変数の場所と範囲であるfieldRangeを計算しmResourceRangesに追加する  

findMemberByOffset()で登録しておいたmMembersからoffsetに対応するメンバ変数を検索し、その変数のTypedShaderVarOffsetを返す  

#### reflectType系関数
ReflectionReflection.cppの方で宣言定義されており、ReflectionTypeを作るための関数  
渡される引数の一つであるTypeLayoutReflectionからReflectionTypeがなんのタイプか、サイズはどのくらいか、再帰的に作成する子要素があるかなどが決定される  

ほかの引数であるReflectionPathはResourceRangeBindingInfoのregIndex、regSpaceの計算のためのもの  
ParameterBlockReflectionの変数構造を親のみをもつツリーにしたものを作る時のノードに対応するもの    
これのツリーをReflectionTypeの再帰作成の時に作っていき、ReflectionTypeの作成がリーフに達したときにResourceRangeBindingInfoの情報が計算され、引数の一つであるpBlockに追加される  
（pBlockはそのpBlock->addResourceRange(bindingInfo)する以外では使われない）  
ちなみにpDeferredはslangのspecialization用、つまり何のレジスタータイプが使われるかはspecializationされるまでわからないことからpPrimaryが終わった後に評価するためのもので、reflectSpecializedType()で用意され、reflectInterfaceType()で評価される  
 

### ReflectionVar
シェーダーの変数定義部分の情報をまとめておくためのクラス  
この変数に対するShaderVarOffsetとReflectionTypeのみ  
ReflectionStructTypeかでのみ管理され、ReflectionStructTypeからのオフセット取得の時に利用される  

reflectVariable()により作成され、渡される[VariableLayoutReflection](https://github.com/shader-slang/slang/blob/master/docs/api-users-guide.md#variable-layouts)を用いてオフセットなどのシェーダー情報を取得し格納する    
VariableLayoutReflectionはSlangAPIの構造体の一つであり、これからシェーダーでのあるスコープ内での「定義」された変数のうちの一つの情報を取得することができる    
例えばShaderReflectionというシェーダー本体情報からはグローバルな変数を、StructのTypeLayoutReflectionからはそのStructで定義した変数をそれぞれお取得できる  
この情報からはその変数に対応するTypeLayoutReflectionも得ることができる 

### ProgramReflection
シェーダーグローバルスコープ変数に対応するクラス  
グローバルスコープで定義した変数の集まりはStructと同じなので、ReflectionStructTypeのタイプとして扱っている  

[ShaderReflection](https://github.com/shader-slang/slang/blob/master/docs/api-users-guide.md#reflection-information)によってグローバルなシェーダー変数を取得し作成していく  
このShaderReflectionはSlangAPIの構造体であり、ここからシェーダーのグローバル変数やエントリーポイントなどが取得できる  


# ProgramVersion, Program, Shaderファイル関連
シェーダーの中身とそれをdx12用のシェーダーブロブやルートシグネチャーにするためのなんやかんや  

いろんなところで保持されているProgramにDefineしていって、ProgramVarsとProgramReflectionを渡すことによってシェーダーブロブやルートシグネチャーが作成され、それを保持したProgramKernels返され、このProgramKernelsを使って、GraphicsStateとかでパイプラインステートオブジェクトが作成されるという感じ  

## ProgramKernelsクラス
一つのslangシェーダーのコンパイルされたものと、それをdx12側で扱うための情報を保持するためのクラス  
情報は各エントリーポイントのブロブ（Shaderクラス）の配列とかProgramReflectionとか  

またその特性からRootSignatureを作成する情報もすべてあるので、ここでシェーダーブロブに対するRootSignatureが作成、保持される  

ちなみにこれはProgramクラスで作成、ProgramVersionで管理されている  
そのためProgramReflection以外の大体の情報はProgramVersionとProgramから作成され渡される  

## ProgramVersionクラス
Slangのコンパイルに必要な情報を一通り渡され保持し、この情報によってコンパイルされたものをProgramから取得し、対応するProgramKernelsの作成を行うためのクラス  

コンパイルと作成はgetKernels(ProgramVars const* pVars)で行われ、今まで作成したProgramKernelsは配列として保持し再利用できるようになっている  

slangシェーダー（Program）のインターフェースなどをProgramVarsのspecializationArgsによってシェーダーが変わったかどうか判定し、ProgramKernelsを（Version）管理するためのクラスであることからProgramVersionと名付けられているのだろうか  
ちなみにこれはProgramクラスで作成、管理されているので、最新バージョンのProgramってだけかもしれない（多分こっち）  

## Programクラス
slangシェーダーファイルとその中身の情報の保持、コンパイル処理、管理用クラス  
エントリーポイントやシェーダーバージョン、シェーダーファイル、そしてこのシェーダーに設定したDefineListを管理する  
DefineListが変更されたかどうかも監視し、再コンパイルが必要かどうかの情報も管理する  

ProgramKernelsに対してこちらはslangシェーダー寄りのクラス  
そのためslangヘッダーを用いたSlangに関するSession作成やらProfile設定やらコンパイルやらのための処理もこちらで一通り実装されている  
で、preprocessAndCreateProgramKernels()でそのコンパイルが行われ、対応するProgramKernelsが返される  

ちなみにSlangプロジェクトのサンプルでもコンパイルしたりその作成されたものを取得したりする部分はProgramという構造体でカプセル化されているので、多分そこからこのクラス名になったのだと思われる  

TODO : Programのslang処理の理解  

## Shaderクラス
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
eyJoaXN0b3J5IjpbLTM0MTIwMzQ1MSwyMTIxOTY5ODI4LC03Mz
U3ODIzOTksLTIxNDMyNTE4MTgsMTk5MTcxNjYxNiwzMDkyNTYy
OTksMTY4Nzc2MzMyMCwtMTI5NzMzMjU1NCw0MTIxNTg3MDUsLT
E0ODAyNzMzNDcsLTk2MTI4ODkzMSwtMTg3NzYxNDk5MSwtMTU5
MjE0NDIyLDE2NTU1MDQwMjYsLTE5MTI4ODkxMTksLTE1MTYyMz
g3NTQsLTMzOTczNTQyOSwtMTQ4MzUzMDQ4NSwtMTMzMDUwOTU2
MSwtMjAxNTIxNTEyM119
-->