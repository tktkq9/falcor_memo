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
配列のサイズやストライドを保持する  

### ReflectionStructType

### ReflectionBasicType

### ReflectionResourceType
### ReflectionInterfaceType

### ReflectionVar
名前を持ったReflectionTypeとShaderVarOffsetのセット  
TODO : TypedShaderVarOffsetとの使い分けはどうしているのか  


## ComputeContext.h, cpp
TODO    D3D12ComputeContextを読む  


### D3D12ComputeContext.cpp
TODO   読む  



## CopyContext.h, cpp
TODO  D3D12CopyContextを読む  


### D3D12CopyContext.cpp
TODO  読む  

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTExMDM0ODU4NSwxMDQ4ODk1NDgwLC0xMD
QxOTIxNjI1LDQwMjM4MTQ2NiwtMTEzNDc2OTY1Miw0NTIzNzI0
ODgsNjQxODYwMDg2LDE1ODEzOTkxNzEsMjAyNTMxMDA1MSwtMT
AzNzA4MTg2OSwtNTAzMTAzNTA5LC03OTM0NjgyMCwxNjAzNTY2
MDc0LDY3MzQxNTA4MiwxNTA1NTg1ODU1LC01NDE1Mzk0MjAsNj
A1MjE1NzU5LDkxNjI2NDczMiw5MDc3ODI4NjcsLTIzNDM0Mzgz
Nl19
-->