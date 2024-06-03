# lesson7 Writing an LLVM PASS

docs : <https://llvm.org/docs/WritingAnLLVMNewPMPass.html>

<https://www.cs.cornell.edu/\~asampson/blog/llvm.html>


task : replace add with mul and mul with add

```python
#include "llvm/IR/PassManager.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/User.h"
#include "llvm/Passes/PassPlugin.h"
#include "llvm/Passes/PassBuilder.h"

using namespace llvm ;
namespace {

    struct SkeletonPass : public PassInfoMixin<SkeletonPass>{
        PreservedAnalyses run(Module &M, ModuleAnalysisManager &AM){
            bool modified =false ;
            for (auto &F : M){
                // errs()<<"I saw a function called "<<F.getName()<<"!\n";
                // errs()<<"Function body:\n"<<F<<"\n";
                for (auto&B : F){
                    for (auto&I : B){
                        errs()<<I<<"\n";
                        
                    }
                }

            }
            for(auto &F : M ){
                for (auto &B : F){
                    for (auto &I : B){
                        if (I.getOpcode()==Instruction::Add)
                        {
                            auto* op =dyn_cast<BinaryOperator>(&I);
                            IRBuilder<> builder(op);
                            // op->swapOperands();
                            Value *lhs =op->getOperand(0);
                            Value *rhs =op->getOperand(1);
                            Value *mul = builder.CreateMul(lhs,rhs);
                            for (auto& U: op->uses()){
                                User* user =U.getUser();
                                user->setOperand(U.getOperandNo(),mul) ;
                            }
                            modified=true ;
                        }
                        if(I.getOpcode()==Instruction::Mul){
                            auto* op = dyn_cast<BinaryOperator>(&I);
                            IRBuilder<> builder(op);
                            // op->swapOperands();
                            Value *lhs =op->getOperand(0);
                            Value *rhs =op->getOperand(1);
                            Value *add = builder.CreateAdd(lhs,rhs);
                            for (auto& U: op->uses()){
                                User* user =U.getUser();
                                user->setOperand(U.getOperandNo(),add) ;
                            }
                            modified=true ;

                        }
                    }
                }
            }
            errs()<<"after swapping:\n";
            for (auto &F:M){
                for (auto &B : F){
                    for (auto &I : B){
                        errs()<<I<<"!\n";
                    }
                }
            }
            return modified? PreservedAnalyses::none() :  PreservedAnalyses::all();

        } 
    };
}

extern "C" LLVM_ATTRIBUTE_WEAK ::llvm::PassPluginLibraryInfo
llvmGetPassPluginInfo() {
    return {
        .APIVersion = LLVM_PLUGIN_API_VERSION,
        .PluginName = "Skeleton pass",
        .PluginVersion = "v0.1",
        .RegisterPassBuilderCallbacks = [](PassBuilder &PB) {
            PB.registerPipelineStartEPCallback(
                [](ModulePassManager &MPM, OptimizationLevel Level) {
                    MPM.addPass(SkeletonPass());
                });
        }
    };
}

// if (auto * op =dyn_cast<BinaryOperator>(&I){
//                             IRBuilder<> builder(op);
//                             Value* lhs = op->getOperand(0);
//                             Value* rhs = op->getOperand(1);
//                             Value* mul = builder.CreateMul(lhs,rhs);

//                             for (auto& U : op->uses()){
//                                 User * user =U.getUser();
//                                 User->setOperand(U.getOperandNo(),mul);

//                             }
//                              modified=true;



                            
//                         })
```


```python
(llm_C_venv) adithyalbhat@Adithyas-MacBook-Pro llvm-pass-skeleton % clang -fpass-plugin=`echo build/skeleton/SkeletonPass.*` -c prog1.c
  %1 = alloca i32, align 4
  %2 = alloca i32, align 4
  %3 = alloca i32, align 4
  %4 = alloca i32, align 4
  %5 = alloca i32, align 4
  store i32 0, ptr %1, align 4
  store i32 8, ptr %2, align 4
  store i32 3, ptr %3, align 4
  %6 = load i32, ptr %2, align 4
  %7 = load i32, ptr %3, align 4
  %8 = add nsw i32 %6, %7
  store i32 %8, ptr %4, align 4
  %9 = load i32, ptr %4, align 4
  %10 = mul nsw i32 %9, 2
  store i32 %10, ptr %5, align 4
  %11 = load i32, ptr %5, align 4
  %12 = call i32 (ptr, ...) @printf(ptr noundef @.str, i32 noundef %11)
  ret i32 0
after swapping:
  %1 = alloca i32, align 4!
  %2 = alloca i32, align 4!
  %3 = alloca i32, align 4!
  %4 = alloca i32, align 4!
  %5 = alloca i32, align 4!
  store i32 0, ptr %1, align 4!
  store i32 8, ptr %2, align 4!
  store i32 3, ptr %3, align 4!
  %6 = load i32, ptr %2, align 4!
  %7 = load i32, ptr %3, align 4!
  %8 = mul i32 %6, %7!
  %9 = add nsw i32 %6, %7!
  store i32 %8, ptr %4, align 4!
  %10 = load i32, ptr %4, align 4!
  %11 = add i32 %10, 2!
  %12 = mul nsw i32 %10, 2!
  store i32 %11, ptr %5, align 4!
  %13 = load i32, ptr %5, align 4!
  %14 = call i32 (ptr, ...) @printf(ptr noundef @.str, i32 noundef %13)!
  ret i32 0!
```


```python
 struct SkeletonPass : public PassInfoMixin<SkeletonPass>{
        PreservedAnalyses run(Module &M, ModuleAnalysisManager &AM){
```

So here PassInfoMixin contains the name of the pass . 

```python
PreservedAnalyses run(Module &M, ModuleAnalysisManager &AM){
```

this runs a new pass manager and not legacy pass manager , new pass manager is available only for select few tasks . 


```python
  for (auto &F : M){
                // errs()<<"I saw a function called "<<F.getName()<<"!\n";
                // errs()<<"Function body:\n"<<F<<"\n";
                for (auto&B : F){
                    for (auto&I : B){
                        errs()<<I<<"\n";
                        
                    }
                }
            }
```

M contains the module , that is LLVM IR instructions are divided into modules and F.getName() contains the name of the function in that module , there can be multiple functions in a module  .

```clike
#include <stdio.h>
int main() {
   int a = 2 * 4;
   int b = 3;
   int c = a + b;
   int d = c * 2;
   printf("%i", d);
   return 0;
}
```

So in the IR of this code , there will be two functions that is main() and printf()

```python
for(auto &F : M ){
                for (auto &B : F){
                    for (auto &I : B){
                        if (I.getOpcode()==Instruction::Add)
                        {
                            auto* op =dyn_cast<BinaryOperator>(&I);
                            IRBuilder<> builder(op);
                            // op->swapOperands();
                            Value *lhs =op->getOperand(0);
                            Value *rhs =op->getOperand(1);
                            Value *mul = builder.CreateMul(lhs,rhs);
                            for (auto& U: op->uses()){
                                User* user =U.getUser();
                                user->setOperand(U.getOperandNo(),mul) ;
                            }
```

here iterating through each module from collection of modules `for(auto &F : M ) :` 

iterating through each function of a module `for (auto &B : F){`

Iterating through each Instruction of that function .`for (auto &I : B)`

check if that instruction has "add" opcode : `if (I.getOpcode()==Instruction::Add)` 

if it has the instruction add then find me the starting address of the instruction : `auto* op =dyn_cast<BinaryOperator>(&I);`

Object used to construct an instruction in IR : `IRBuilder<> builder(op); ` 

Here  the statement is commented out `// op->swapOperands();` but essentially what it does is replace the operands of a binary operator . ex `add a b` will become `add b a` . 

```python
Value *lhs =op->getOperand(0);
Value *rhs =op->getOperand(1);
Value *mul = builder.CreateMul(lhs,rhs);
```


here op→operand(0) will get the 1st operand that is "a" and  op→operand(1) will get the second operand . 


```python
for (auto& U: op->uses()){
  User* user =U.getUser();
  user->setOperand(U.getOperandNo(),mul) ;
                            }
```


```javascript
for (auto& U : op->uses()) {
```

this iterates over all uses of the original Add instruction . 

```javascript
User* user = U.getUser();
```

In LLVM a User object is that object that can use a Value . In this context it is refers to the result of add instruction .

```javascript
user->setOperand(U.getOperandNo(), mul);
```

for the result of that instruction , set the operand in the indices returned by U.getOprandNo() and replace that operand with mul . which contains the new instruction . 

The same is repeated for multiplication instruction too .  

before swapping : 

```python
%8 = add nsw i32 %6, %7
```

After swapping : 

```python
  %8 = mul i32 %6, %7!
  %9 = add nsw i32 %6, %7!
```

here what do observe? the add instruction is technically not removed . its added after mul instruction . But the single static assignment of add instruction  %9 is never used anywhere . A dead code elimination pass has to done over the new code to get a clean version .