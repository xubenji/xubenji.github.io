llvmlite是llvm的一个简化版本，他使用python来生成LLVM IR，首先我们熟悉一下整个编译过程。
1，拿到llvmir：
    
```
import llvmlite.ir as ir

# 创建一个int64类型的变量
int64 = ir.IntType(64)

# 你也可以使用这个来创建一个浮点类型的变量，不过浮点类型的变量就不可以传参了
double = ir.DoubleType()

# 创建一个module，每一个module就相当于一个。c文件一样，
# 其实不一样，我这么说是为了方便理解
# 到时候你就拿这个module去编译的
module = ir.Module(name=__file__)

# 有了模块我们需要创建一个函数，func就是我们创建出来的函数
fnty = ir.FunctionType(int64, (int64, int64))
func = ir.Function(module, fnty, name="fpadd")

# 这个func是一个对象，现在我们给他添加一个basic block
# 我们得到了builder，下面我就是拿builder做一些操作了
block = func.append_basic_block(name="entry")
builder = ir.IRBuilder(block)

# 给这个block添加一个add指令
a, b = func.args
result = builder.add(a, b, name="res")

# 将这个指令return
builder.ret(result)

# 现在我们打印出来看看这个
print(module)
```
打印出来的结果：

```
; ModuleID = "getIR.py"
target triple = "unknown-unknown-unknown"
target datalayout = ""

define i64 @"fpadd"(i64 %".1", i64 %".2")
{
entry:
  %"res" = add i64 %".1", %".2"
  ret i64 %"res"
}
```

至此我们创建了一个带加法指令的函数在llvm ir中

2，编译module
ir已经被创建好了，我们需要去编译他，废话不多说直接上代码。
首先，我们补上一些import，（注意 from这种必须第一行）

```
from __future__ import print_function
from ctypes import CFUNCTYPE, c_int64
import llvmlite.binding as llvm
```

然后再将下面的代码补到print(module)之后

```
llvm_ir = str(module)

# 从这里开始到一直到
# func_ptr = engine.get_function_address("fpadd")
# 你可以看作是一个整体，把编译好的module变成字符串，然后再送入引擎编译
# 下边的init代码你最好直接复制好了，一个非常标准的模版
# 实际上就是一些类传来传去，我们只需要关注上半部分就行了
# All these initializations are required for code generation!
llvm.initialize()
llvm.initialize_native_target()
llvm.initialize_native_asmprinter()  # yes, even this one

def create_execution_engine():
    """
    Create an ExecutionEngine suitable for JIT code generation on
    the host CPU.  The engine is reusable for an arbitrary number of
    modules.
    """
    # Create a target machine representing the host
    target = llvm.Target.from_default_triple()
    target_machine = target.create_target_machine()
    # And an execution engine with an empty backing module
    backing_mod = llvm.parse_assembly("")
    engine = llvm.create_mcjit_compiler(backing_mod, target_machine)
    return engine

def compile_ir(engine, llvm_ir):
    """
    Compile the LLVM IR string with the given engine.
    The compiled module object is returned.
    """
    # Create a LLVM module object from the IR
    mod = llvm.parse_assembly(llvm_ir)
    mod.verify()
    # Now add the module and make sure it is ready for execution
    engine.add_module(mod)
    engine.finalize_object()
    engine.run_static_constructors()
    return mod

# 前面的准备工作做好了，这里就是开始编译
# 编译好了以后就是拿engine做操作，和mod无关了
engine = create_execution_engine()
mod = compile_ir(engine, llvm_ir)



# Look up the function pointer (a Python int)
# 拿到fpadd的函数地址
func_ptr = engine.get_function_address("fpadd")

# Run the function via ctypes
# 这个地方未来可能需要管一下，中间的两个函数其实是不需要管的，
# 不管你的module长的啥样子，中间的两个函数create_execution_engine
# 和compile_ir都不会有任何变化，
# 所以俺推荐你直接复制粘贴中间的这一堆东西行了
cfunc = CFUNCTYPE(c_int64, c_int64, c_int64)(func_ptr)
res = cfunc(1, 3)
print("fpadd(...) =", res)

```

然后俺编译一下得到了如下输出：

```
root@localhost:~/vex2tfa/tests# python3 getir.py
; ModuleID = "getir.py"
target triple = "unknown-unknown-unknown"
target datalayout = ""

define i64 @"fpadd"(i64 %".1", i64 %".2")
{
entry:
  %"res" = add i64 %".1", %".2"
  ret i64 %"res"
}

fpadd(...) = 4

```

3, 添加其他函数
我们已经看到了这个基本的流程，从开始创建module，再到创建函数，在到创建basic block，最后创建instruction。这个都是基于官方文档复习，俺在这里没有做太多的更改，仅仅熟悉了一个流程，仅仅知道这个流程是不够的，不然怎么能够成长为高级程序员呢，下面我们需要做一些其他的操作，添加一个新的函数，然后在这个旧的函数中去call这个新的函数。
我们在module = ir.Module(name=__file__)这一行下边添加以下内容。

```
# 创建一个名字为add2的函数，（int64, int64)表示这个函数接收两个类型为int64的参数
func2 = ir.Function(module, ir.FunctionType(int64, (int64, int64)),  "add2{}".format(64))
#创建一个名字为entry2的basic block
block2 = func2.append_basic_block(name="entry2")
# 将这个创建好的basic block送入builder
builder2 = ir.IRBuilder(block2)
# 拿到这个函数的两个参数
c, d = func2.args
# 添加一个减法指令
result2 = builder2.sub(d, c, name="res2")
# 将这个减出来的数字返回
builder2.ret(result2)

```

这个时候我们就创建了一个名为add2的带一个减法指令的函数
我们怎么去call这个函数呢？
我们需要在result = builder.add(a, b, name="res")这一行下添加一个一个call指令

```
result_of_call = builder.call(func2, [result, b])
```
然后修改返回值

```
builder.ret(result) 改为 builder.ret(result_of_call)

```
改好了以后我们打印一下输出：

```
root@localhost:~/vex2tfa/tests# python3 getir.py
; ModuleID = "getir.py"
target triple = "unknown-unknown-unknown"
target datalayout = ""

define i64 @"add264"(i64 %".1", i64 %".2")
{
entry2:
  %"res2" = sub i64 %".2", %".1"
  ret i64 %"res2"
}

define i64 @"fpadd"(i64 %".1", i64 %".2")
{
entry:
  %"res" = add i64 %".1", %".2"
  %".4" = call i64 @"add264"(i64 %"res", i64 %".2")
  ret i64 %".4"
}

fpadd(...) = -1
```

我们发现结果变成了-1， 那是因为
1 + 3 = 4
3 - 4 = -1
好啦，如果创建函数和调用已经创建的函数我们基本上已经会了，下一期俺将介绍如何在IR中创建if_else语句
喜欢交流的人可以加秋秋裙：375678777
