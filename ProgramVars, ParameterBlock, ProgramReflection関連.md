
# ProgramVars, ParameterBlock, ProgramReflection関連

Falcorではシェーダー言語としてSlangを使っており、レジスター番号やスペース番号が自動で設定され、その番号などはSlangAPIから取得することになる  
そのためこの変数名と番号の情報の管理し、それらをコマンドリストに自動で設定できるようにしなければいけない  
また、変数を番号指定で変更するのはナンセンスなので、番号情報を外部から隠蔽し、構造体名や変数名などの名前指定のみで変数変更ができるようにもしたい  
（あとSlang特有の機能であるParameterBlockとかインターフェースとかもどうにかしたい）  
こういった機能がこのProgramVars, ShaderVar, ParameterBlock, ProgramReflectionとそれに関連するクラスで実装されている  

それぞれざっと説明すると
- ProgramVarsが外部用、  
- ProgramReflectionが変数名と番号との対応付け情報、  
- ShaderVarがProgramReflectionをもとにParameterBlockから変数参照を持ってくるための橋渡し、  
- ParameterBlockが変数実体

といった持ち回りになっている。タブンネ  

ちなみに一つのProgramVars（ParameterBlockのサブクラスなのでParameterBlockでもある）が（一つのPassに対応する）シェーダーの変数すべてを担当しているのでこれ一つでいいはず。複数使うパターンもあるのかもしれない、知らんけど  

ちなみにこれらが作られる順は  
 1. シェーダーファイルからProgram（シェーダーとそのSlangAPIの処理に関するクラス）が作られ、その時にProgramReflectionが作られる
 2. このProgramReflectionをもとにProgramVarsが作られる。ちなみにProgramVarsはParameterBlockのサブクラスなので要するにParameterBlock作られる  
 3. 終わり  

ShaderVarはProgramVarsから変数アクセスしようとしたときにその都度作られ、使い終わったら破棄される  
だいたいこんな感じ  

以下それぞれざっくり詳細  

## ProgramVars
ParameterBlockのサブクラス  
これのサブクラスとしてGraphicsVarsやComputeVarsがあり、SharedPtr がParameterBlockSharedPtrとなっていることからも分かるように、変数情報などに対する外部用クラスとなっている  

ParameterBlockとの違いはProgramReflectionとEntryPointGroupVars（これもまたParameterBlockのサブクラス）の配列も持つ  
だがcppでもほぼ利用されていないことから、これはおそらく外部で必要になったとき用の変数  

これのサブクラスであるGraphicsVarsやComputeVarsはProgramVarsを作るためのcreate()関数を持つProgramVarsラッパークラスみたいなものとなっている  
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
渡されたLayoutに対応するディスクリプターヒープのハンドル作成管理クラス  
create()でLayoutに沿ってディスクリプターヒープにrange分のハンドルを割り当て、そのハンドル情報を保持する  

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


### ParameterBlockReflection
リソースとコンスタントバッファーとParameterBlockのReflectionStructType、  
レジスター番号、スペース番号、コマンドリストにセットするディスクリプターの番号、ルートシグネチャーの番号の入れ物クラス  

ProgramReflectionを作るときに上の情報も一緒に計算し、ParameterBlockReflectionに入れていくパターン、  
ReflectionTypeから作るパターン  
TypeLayoutReflectionからReflectionTypeと一緒に作っていくパターン  
がある  

### EntryPointGroupReflection
ParameterBlockReflectionのサブクラスでcreate()とコンストラクタが違うのみ  
エントリーポイントで使われているグローバル変数に対するParameterBlockReflection  

エントリーポイントが使う変数はすべて同じものをもっているか、またはすべての変数をもっているエントリーポイントが少なくとも一つは存在している前提で実装されているらしいので、  
一番UniformParameterCount多いエントリーポイントのやつを選んで、それのParameterBlockReflectionを作成している  

#### ParameterBlockReflectionFinalizer
ParameterBlockReflectionが持っているmDescriptorSets（DescriptorSetInfoの配列  ）、mRootDescriptorRangeIndices、mParameterBlockSubObjectRangeIndicesを作成するための構造体  
事前に計算しておいたDefaultConstantBufferBindingInfo、ResourceRangeBindingInfo、ResourceRangeを使って作成していく  
ParameterBlockReflectionやProgramReflectionで一通りParameterBlockReflectionの作成や初期化が終わったタイミングでこの構造体のfinalize()関数が呼ばれる  

mDescriptorSetsはスペースとタイプごとのDescriptorSetInfoの配列である  
DescriptorSetInfoはシェーダーに対するレジスター、スペース、数、タイプ（これはSlangの方で自動で設定される）に対する、CPU側の変数をコマンドリスト設定するための情報である  
また、Layoutに設定したmRangesとリソースやsubObjectResourceとの対応関係情報も持っている  



### ProgramReflection
シェーダーグローバルスコープ変数とエントリーポイント、その入出力変数など、シェーダー全体に対応するクラス  
ファイル名にもなっているように、ProgramReflectionファイルの他のクラスの大本  
これにより、様々なシェーダー変数やエントリーポイントなどをその名前から取得できるようにする  

グローバルスコープで定義した変数の集まりはグローバルStructのようなものなので、ReflectionStructTypeのタイプとして扱っている  
これと対応するParameterBlockReflectionを作ってmpDefaultBlockに保持している  

また、それとは別にエントリーポイントグループごとのEntryPointGroupReflectionも作成され、mEntryPointGroupsに格納される  
これはProgramReflectionファイルにあるクラスでは使われず、外部で使用される  

さらに全てのエントリーポイントで使われているすべての入出力変数の情報も作成され、mPsOut、mVertAttr、mVertAttrBySemanticに格納される  
これにより、入出力変数の情報も名前検索できるようになる  

このクラスはcreate()関数によって作成され、ProgramVersionに対応する[ShaderReflection](https://github.com/shader-slang/slang/blob/master/docs/api-users-guide.md#reflection-information)によってグローバルなシェーダー変数を取得しメンバ変数を作成していく  
このShaderReflectionはSlangAPIの構造体であり、ここからシェーダーのグローバル変数やエントリーポイントなどが取得できる  

<!--stackedit_data:
eyJoaXN0b3J5IjpbNDY2MzM1OTY3LDE1MjIyNDM5MDksLTEzNz
Y0MDMyNTUsMTIwMTI3MzUwNSwtNTI2NDEyMTczLDcwOTM5MDI1
MCwyMTc4MTQzMTgsOTEzOTA5NTE2LDE0NDI5OTkzNiwxNzQxNT
k5OTAwLDg3NDE1NjQ4MywtMTUxNzI3OTQ0LDEyOTg1NzY1MTcs
LTEwMTQzMTY4NDgsLTY5NTExNDI1OCwtMTMyNjMzMTM4MywxND
MyMTQ0MDQwLC0xMDM1MTM4MTI5LDExNTkwMDAwMjIsMTkwNDQ1
NjM1NF19
-->