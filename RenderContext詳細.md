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
ShaderVarOffsetと後述するReflectionType


 参照：# GLSL-to-HLSL reference  
 https://docs.microsoft.com/ja-jp/windows/uwp/gaming/glsl-to-hlsl-reference

## ComputeContext.h, cpp
TODO    D3D12ComputeContextを読む  


### D3D12ComputeContext.cpp
TODO   読む  



## CopyContext.h, cpp
TODO  D3D12CopyContextを読む  


### D3D12CopyContext.cpp
TODO  読む  

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTYwMzU2NjA3NCw2NzY1ODU0MjksNjczND
E1MDgyLDE1MDU1ODU4NTUsLTU0MTUzOTQyMCw2MDUyMTU3NTks
OTE2MjY0NzMyLDkwNzc4Mjg2NywtMjM0MzQzODM2LDE2NTQ1Mj
I2MDRdfQ==
-->