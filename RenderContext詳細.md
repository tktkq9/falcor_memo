# RenderContext詳細

RenderContext.h, cppと関連するクラスが膨大なのでFalcorプロジェクトのメモとは分けてまとめる  

## RenderContext.h, cpp
TODO    D3D12RenderContextを読む  


### D3D12RenderContext.cpp
TODO   読む  

## ParameterBlock.h, cpp
TODO   読む  
おそらくRenderContextを通してシェーダー用パラメーターの名前と値を設定する用クラス  
実際の保存先とレジスター番号に関するクラスの管理はShaderVar.h, cpp、レジスター番号管理のクラスはProgramReflection.h, cppかな？微妙  

ParameterBlockはslangの機能なので多分これと関係している  

## ShaderVar.h, cpp
 
## ProgramReflection.h, cpp


### UniformShaderVarOffset
コンスタントバッファー用のバイトオフセット値管理クラス  
 uniform shader variableはOpenGLシェーダーのuniformのことで、dx12ではコンスタントバッファーに対応する  

### ResourceShaderVarOffset
シェーダーリソース用のオフセット値管理クラス  
実装はUniformShaderVarOffsetとほぼ変わらず  
mRangeIndexとmArrayIndexの二つのメンバ変数があるが、ヘッダーにも書いてある通り、レジスターなどと対応関係があるわけではないらしいのでこの値を直接操作してはいけない  

### ShaderVarOffset
UniformShaderVarOffsetとResourceShaderVarOffsetが合わさったもの  

### TypedShaderVarOffset
ShaderVarOffsetと後述するReflectionTypeが合わさったもの  

    TypedShaderVarOffset lightOffset = pSomeType["light"]; // get type and offset of texture `light` inside `pSomeType`
    TypedShaderVarOffset materialOffset = pBlock["material"]; // get type and offset of sampler `material` inside block

 In addition, a `TypedShaderVarOffset` can be used to look up offsets for sub-fields/-elements of shader variables with structure or array types:
    
    UniformShaderVarOffset lightPosOffset = lightOffset["position"];
    ResourceShaderVarOffset diffuseMapOffset = materialOffset["diffuseMap"];
        
 Such offsets are always relative to the root type or block where lookup started.
    For example, in the above code `lightPosOffset` would be the offset of the  field `light.position` relative to the enclosing type `pSomeType` and *not*  the offset of the `position` field relative to the immediately enclosing `light` field.

### ReflectionType
KindはGLSL準拠  
https://www.khronos.org/opengl/wiki/Data_Type_(GLSL)  
Kindに対応する、Typeクラス（このあと示す）のベースクラスであり、それぞれのTypeクラスとそのサイズを取得する関数が実装されている  

またResourceRangeというstructがあり、各々のタイプに対するリソースの大きさが格納されているようだが、ヘッダーコメントにもあるように、その情報はParameterBlockから取得したほうがいいらしい  

### ReflectionArrayType
ReflectionTypeのサブクラスの配列タイプ  
ReflectionTypeサブクラスに対応するやつの配列とサイズやストライドを保持する  

### ReflectionStructType
ReflectionTypeのサブクラスのstructタイプ  
ReflectionVar（名前を持ったReflectionTypeとShaderVarOffsetのセット）をメンバ変数にもつクラス名付きstructを形成する  

### ReflectionBasicType
bool, int, floatとそのベクターに関するタイプ  
RowMajorかどうかも設定できる  

### ReflectionResourceType
様々なバッファーに関するタイプ  

        enum class Type
        {
            Texture,
            StructuredBuffer,
            RawBuffer,
            TypedBuffer,
            Sampler,
            ConstantBuffer
        };
          
そのテクスチャーに関する設定（ShaderAccess、 各要素のタイプ、Dimensions、StructuredType（CounterやConsumeなど））の情報が入っている  


### ReflectionInterfaceType



### ReflectionVar
名前を持ったReflectionTypeとShaderVarOffsetのセット  
似たようなクラスとしてTypedShaderVarOffsetがあるが、これを継承しているわけではない  
TODO : TypedShaderVarOffsetとの使い分けはどうしているのか  

## ProgramVersion.h, cpp


## Program.h, cpp
DXR以外のシェーダーのマクロを管理するクラス  
Define（マクロ）を追加、削除、設定し、更新処理も行う  
また、Descからシェーダーの情報も取得できる  

## RtProgram.h, cpp
DXRのシェーダーのマクロを管理するクラス  
Programを継承している  

## Shader.h, cpp
シェーダー本体  

## ComputeContext.h, cpp
TODO    D3D12ComputeContextを読む  


### D3D12ComputeContext.cpp
TODO   読む  



## CopyContext.h, cpp
TODO  D3D12CopyContextを読む  


### D3D12CopyContext.cpp
TODO  読む  

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0OTU1NjU0OTgsLTE4OTY2MDgzNTAsMT
I0NTgxMjU0MSwtMTkxMTk2NTkwMywxNTg1MTA0NTcwLC0xNDQw
NzY1NjI1LDQyNTM0ODQ1OSwtMTMyMTY2ODU5NiwzMTM2NjAyMz
UsLTExMDYzNjc3NDUsMTk5Nzk3NTE0NywtMjEwNzkxOTg5Niwt
ODE4NTAxOTU4LC0xMTE4MDEzMTAzLDk5NTA2NDEwOSwyMDYyND
kxNzUwLC05NjIyMDc0ODgsLTc1MTU1NzU4MiwxOTUzNzUyMTI0
LDk3NzY1NTQ3N119
-->