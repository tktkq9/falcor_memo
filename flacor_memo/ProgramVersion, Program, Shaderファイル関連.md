# ProgramVersion, Program, Shaderファイル関連
TODO : Programのslang処理の理解  Program  

シェーダーの中身とそれをdx12用のシェーダーブロブやルートシグネチャーにするためのなんやかんや  
低レイヤー処理関連  

いろんなところで保持されているProgramにDefineしていって、ProgramVarsとProgramReflectionを渡すことによってシェーダーブロブやルートシグネチャーが作成され、それを保持したProgramKernels返され、このProgramKernelsを使って、GraphicsStateとかでパイプラインステートオブジェクトが作成されるという感じ  

 
## ProgramKernelsクラス
一つのslangシェーダーのコンパイルされたものと、それをdx12側で扱うための情報を保持するためのクラス  
情報は各エントリーポイントのブロブ（Shaderクラス）の配列とかProgramReflectionとか  

またその特性からRootSignatureを作成する情報もすべてあるので、ここでシェーダーブロブに対するRootSignatureが作成、保持される  

ちなみにこれはProgramクラスで作成、ProgramVersionで管理されている  
そのためProgramReflection以外の大体の情報はProgramVersionとProgramから作成され渡される  
Va
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

ちなみに[Slangプロジェクトのサンプル](https://github.com/shader-slang/slang/tree/master/examples/model-viewer)でもコンパイルしたりその作成されたものを取得したりする部分はProgramという構造体でカプセル化されているので、多分そこからこのクラス名になったのだと思われる  

TODO : Programのslang処理の理解  

## Shaderクラス
Shaderという名を持つが、シェーダー本体はProgramクラスの方であり、こちらはどちらかというとエントリーポイントのブロブクラス  

ちなみにDefineListもこのクラス内に定義されているが、これは主にProgramのコンパイルで使われ、さらにShaderクラスでは使われてない。なぜここにあるのか  

あとShaderファイルにはComPtrテンプレートクラスもあるが、これもシェーダーのみのためのものではなくどちらかというとProgramクラスでよく使われている。なぜこあ  

もしかしたらこれらはSlang適用時の残留物であろうか（SlangはFalcorができた後に作成され、それなりに後のバージョンで適用された）  
こいつら全員Programファイルに吸収された方がいいと思う  

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTYxMjI1OTQyM119
-->