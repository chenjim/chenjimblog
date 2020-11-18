
> 本地首发地址 <https://h89.cn/archives/189.html>  
> 最新更新地址 <https://gitee.com/chenjim/chenjimblog>  
> 原文地址  <https://www.romainguy.dev/posts/2024/speeding-up-isblank/> 


最近在优化 Jetpack Compose 运行时的部分时，偶然发现了一个看似无害的 API [isBlank()](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/is-blank.html) 。如果调用的字符串为空或仅由空格字符组成，则此 API 将返回 true 。  
但它真的无害吗？让我们看一下 JVM 实现，以更好地了解它的作用：  
```kotlin
public actual fun CharSequence.isBlank(): Boolean =
    length == 0 || indices.all { this[it].isWhitespace() }
```
简单明了，开门见山。不幸的是，字节码讲述了一个非常不同的故事：  
```asm
const-string v0, "s" // string@0080
invoke-static {v6, v0}, Lkotlin/text/CharsKt__CharKt;.checkNotNullParameter:(Ljava/lang/Object;Ljava/lang/String;)V // method@002a
invoke-interface {v6}, Ljava/lang/CharSequence;.length:()I // method@0006
move-result v0
const/4 v1, #int 1 // #1
if-eqz v0, 0069 // +005f
new-instance v0, Lkotlin/ranges/IntRange; // type@001c
invoke-interface {v6}, Ljava/lang/CharSequence;.length:()I // method@0006
move-result v2
add-int/lit8 v2, v2, #int -1 // #ff
const/4 v3, #int 0 // #0
invoke-direct {v0, v3, v2}, Lkotlin/ranges/IntRange;.<init>:(II)V // method@0026
instance-of v2, v0, Ljava/util/Collection; // type@0017
if-eqz v2, 0026 // +000c
move-object v2, v0
check-cast v2, Ljava/util/Collection; // type@0017
invoke-interface {v2}, Ljava/util/Collection;.isEmpty:()Z // method@001b
move-result v2
if-eqz v2, 0026 // +0003
goto 0064 // +003f
invoke-virtual {v0}, Lkotlin/ranges/IntProgression;.iterator:()Ljava/util/Iterator; // method@001e
move-result-object v0
move-object v2, v0
check-cast v2, Lkotlin/ranges/IntProgressionIterator; // type@001b
iget-boolean v2, v2, Lkotlin/ranges/IntProgressionIterator;.hasNext:Z // field@0007
if-eqz v2, 0064 // +0035
move-object v2, v0
check-cast v2, Lkotlin/ranges/IntProgressionIterator; // type@001b
iget v4, v2, Lkotlin/ranges/IntProgressionIterator;.next:I // field@0008
iget v5, v2, Lkotlin/ranges/IntProgressionIterator;.finalElement:I // field@0006
if-ne v4, v5, 0047 // +000f
iget-boolean v5, v2, Lkotlin/ranges/IntProgressionIterator;.hasNext:Z // field@0007
if-eqz v5, 0041 // +0005
iput-boolean v3, v2, Lkotlin/ranges/IntProgressionIterator;.hasNext:Z // field@0007
goto 004c // +000c
new-instance v6, Ljava/util/NoSuchElementException; // type@0019
invoke-direct {v6}, Ljava/util/NoSuchElementException;.<init>:()V // method@001c
throw v6
iget v5, v2, Lkotlin/ranges/IntProgressionIterator;.step:I // field@0009
add-int/2addr v5, v4
iput v5, v2, Lkotlin/ranges/IntProgressionIterator;.next:I // field@0008
invoke-interface {v6, v4}, Ljava/lang/CharSequence;.charAt:(I)C // method@0005
move-result v2
invoke-static {v2}, Ljava/lang/Character;.isWhitespace:(C)Z // method@0008
move-result v4
if-nez v4, 005f // +000b
invoke-static {v2}, Ljava/lang/Character;.isSpaceChar:(C)Z // method@0007
move-result v2
if-eqz v2, 005d // +0003
goto 005f // +0003
const/4 v2, #int 0 // #0
goto 0060 // +0002
const/4 v2, #int 1 // #1
if-nez v2, 002a // -0036
const/4 v6, #int 0 // #0
goto 0065 // +0002
const/4 v6, #int 1 // #1
if-eqz v6, 0068 // +0003
goto 0069 // +0002
const/4 v1, #int 0 // #0
return v1
```
该实现遇到了几个问题，包括分配新对象： IntRange 和 Iterator。如果我们再次查看 Kotlin 实现，我们会注意到 CharSequence.indices 调用 all()，IntRange 通常在编译常 for 规循环时会被丢弃。在这种情况下，对使用泛型定义的 API 的调用 all() 会导致上面的荒谬代码，该代码经过许多抽象只是为了迭代字符列表。  
我们可以使用传统的 for 循环来代替漂亮的单行：  
```kotlin
private inline fun CharSequence.fastIsBlank(): Boolean {
    for (i in 0..<length) {
        if (!this[i].isWhitespace()) {
            return false
        }
    }
    return true
}
```
代码仍然很容易理解，但生成的字节码效率更高：  
```asm
invoke-interface {v6}, Ljava/lang/CharSequence;.length:()I // method@0007
move-result v0
const/4 v1, #int 1 // #1
sub-int/2addr v0, v1
const/4 v2, #int 0 // #0
const/4 v3, #int 0 // #0
if-ge v3, v0, 0024 // +001c
invoke-interface {v6, v3}, Ljava/lang/CharSequence;.charAt:(I)C // method@0006
move-result v4
invoke-static {v4}, Ljava/lang/Character;.isWhitespace:(C)Z // method@0009
move-result v5
if-nez v5, 0021 // +000f
const/16 v5, #int 160 // #a0
if-eq v4, v5, 0021 // +000b
const/16 v5, #int 8199 // #2007
if-eq v4, v5, 0021 // +0007
const/16 v5, #int 8239 // #202f
if-eq v4, v5, 0021 // +0003
return v2
add-int/lit8 v3, v3, #int 1 // #01
goto 0008 // -001b
return v1
```
如果我们使用 1,000 个字符串的列表对这两种实现进行基准测试，我们会发现我们的for循环版本快了 60% 并且零分配：  
```
37,998 ns  2001 allocs  isBlank
14,943 ns     0 allocs  isBlankForLoop
```
我们甚至可以进一步改进它。[如果仔细观](https://cs.android.com/android/platform/superproject/main/+/main:libcore/ojluni/src/main/java/java/lang/Character.java;l=11259?q=isSpaceChar)察字节码，您会发现 KotlinisWhitespace() 调用了Character.isSpaceChar() 和 Character.isWhitespace()，这两个函数执行类似的工作（实现 [几乎](https://cs.android.com/android/platform/superproject/main/+/main:libcore/ojluni/src/main/java/java/lang/Character.java;l=11130;bpv=1;bpt=1?q=isSpaceChar)相同）。  
为了进一步优化我们的函数，我们可以只调用 Character.isWhitespace() 并执行额外的检查Character.isSpaceChar()（相反会更复杂）：     
```kotlin
private inline fun CharSequence.fastIsBlank(): Boolean {
    for (i in 0..<length) {
        val c = this[i]
        if (!Character.isWhitespace(c) &&
            c != '\u00a0' && c != '\u2007' && c != '\u202f') {
            return false
        }
    }
    return true
}
```
现在的基准是：  
```
37,998 ns  2001 allocs  isBlank
14,943 ns     0 allocs  isBlankForLoop
13,519 ns     0 allocs  isBlankForLoopOneCall
``` 
最新版本比原始实现快了近 65%，而且我们也不会触发垃圾收集器！

我将在提前编译后为您节省最终 ARM 汇编的打印输出，只需知道我们从原始的 161 条指令减少到 53 条指令即可。这还不包括我们通过不调用而保存的指令 Character.isSpaceChar()。

那么这有关系吗？如果您在用户发送表单时验证文本字段，则可能不会有太大影响。如果您尝试解析大量字符串，仅 GC 节省就可能值得关注这一点。但更重要的是，这表明即使是看似简单的一行代码，也会在您意想不到的地方对性能产生不成比例的影响。

---
相关文章  

[Kotlin 微优化 1 ](https://www.romainguy.dev/posts/2024/micro-optimizations-in-kotlin-1/)  
[Kotlin 微优化 2 ](https://www.romainguy.dev/posts/2024/micro-optimizations-in-kotlin-2/)  
[Kotlin 微优化 3 ](https://www.romainguy.dev/posts/2024/micro-optimizations-in-kotlin-3/)  



