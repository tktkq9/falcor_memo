# RenderContext詳細

RenderContext.h, cppと関連するクラスが膨大なのでFalcorプロジェクトのメモとは分けてまとめる  

## RenderContext.h, cpp
TODO    D3D12RenderContextを読む  


### D3D12RenderContext.cpp
TODO   読む  

## ParameterBlock.h, cpp
TODO   読む  
おそらくRenderContextを通してシェーダー用パラメーターの名前と値を設定する用クラス  
実際の保存先とレジスター番号に関するクラスの管理はShaderVar.h, cpp、レジスター番号管理のクラスはProgramReflection.h, cppかな？  

## ShaderVar.h, cpp
 
## ProgramReflection.h, cpp

各クラスやその変数名などはGLSL準拠になっている  
 参照：# GLSL-to-HLSL reference  
 https://docs.microsoft.com/ja-jp/windows/uwp/gaming/glsl-to-hlsl-reference

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


## Program.h, cpp
シェーダーを管理するクラス  

## RtProgram.h, cpp
DXRのシェーダーを管理するクラス  

## ComputeContext.h, cpp
TODO    D3D12ComputeContextを読む  


### D3D12ComputeContext.cpp
TODO   読む  



## CopyContext.h, cpp
TODO  D3D12CopyContextを読む  


### D3D12CopyContext.cpp
TODO  読む  

<!--stackedit_data:
eyJoaXN0b3J5IjpbMjM3ODM4OTQzLC0xMzIxNjY4NTk2LDMxMz
Y2MDIzNSwtMTEwNjM2Nzc0NSwxOTk3OTc1MTQ3LC0yMTA3OTE5
ODk2LC04MTg1MDE5NTgsLTExMTgwMTMxMDMsOTk1MDY0MTA5LD
IwNjI0OTE3NTAsLTk2MjIwNzQ4OCwtNzUxNTU3NTgyLDE5NTM3
NTIxMjQsOTc3NjU1NDc3LDEwNDg4OTU0ODAsLTEwNDE5MjE2Mj
UsNDAyMzgxNDY2LC0xMTM0NzY5NjUyLDQ1MjM3MjQ4OCw2NDE4
NjAwODZdfQ==
-->