---
transition: view-transition
---

# Python 3.14: π 的 Python

经过两年，我们来看 Python 3.14 的绝赞变化

<div class="grid grid-cols-3 gap-10 pt-4 -mb-6">

<v-clicks>

<v-click>

<div height="600px">

1. free threading official release
2. sub-interpreter support

</div>

</v-click>

<v-click>

<div height="600px">

1. JIT 大幅迭代，Tier2 优化演进颇多
2. tail call interpreter 奠定更近一步的基础

</div>


</v-click>

<v-click>

<div height="600px">

1. Remote Debugging 官方支持

</div>

</v-click>

</v-clicks>

</div>

---
transition: view-transition
---

# Python 3.14: π 的 Python

OK 我将选择两个我最喜欢的部分来给大家领略一下 Python 3.14 的魅力

<div class="grid grid-cols-3 gap-10 pt-4 -mb-6">

<v-clicks>

<v-click>

<div height="600px">

1. tail call interpreter

</div>

</v-click>

<v-click>

<div height="600px">

&&

</div>

</v-click>

<v-click>

<div height="600px">

1. Remote Debugging

</div>

</v-click>

</v-clicks>

</div>

---
transition: view-transition
---

# Python 3.14: π 的 Python

先聊 tail call interpreter 之前我们先来看一段代码（某种意义上是 Python 解释器核心部分一个简化版）

<div class="grid grid-cols-3 gap-10 pt-4 -mb-6">

<v-clicks>

<div>
<v-clicks>

```c{monaco}
#include <stdio.h>

void basic_computed_goto(int operation) {
    static void* jump_table[] = {
        &&op_add,   
        &&op_sub,   
        &&op_mul,   
        &&op_div,   
        &&op_mod,   
        &&op_default
    };
    
    int a = 10, b = 3;
    int result;
    
    if (operation < 0 || operation > 4) {
        operation = 5;
    }
    
    printf("Operation %d: a=%d, b=%d -> ", operation, a, b);
    
    goto *jump_table[operation];
    
op_add:
    result = a + b;
    printf("ADD: %d\n", result);
    return;
    
op_sub:
    result = a - b;
    printf("SUB: %d\n", result);
    return;
    
op_mul:
    result = a * b;
    printf("MUL: %d\n", result);
    return;
    
op_div:
    if (b != 0) {
        result = a / b;
        printf("DIV: %d\n", result);
    } else {
        printf("DIV: Error (division by zero)\n");
    }
    return;
    
op_mod:
    if (b != 0) {
        result = a % b;
        printf("MOD: %d\n", result);
    } else {
        printf("MOD: Error (division by zero)\n");
    }
    return;
    
op_default:
    printf("Unknown operation\n");
    return;
}
```

</v-clicks>
</div>

<div>
这段代码的缺点是什么？
</div>

<div>

1. Computed Goto 作为 clang 和 gcc 特化的功能，那么其余平台受益的可能性较小
2. 目前 Computed Goto 其实并不成熟，在同一个编译器不同的版本可能会有不同的问题
3. 超大型的 switch case 会导致编译器对于 switch case 的优化不够好

</div>

</v-clicks>

</div>

---
transition: view-transition
---

# Python 3.14: π 的 Python

编译器有问题的 case

---
transition: view-transition
---
# Python 3.14: π 的 Python

所以怎么办？

<v-clicks>

是否能让代码的优化范围更明确？

兼容性更好？

</v-clicks>

---
transition: view-transition
---

# Python 3.14: π 的 Python

tail call 帮我们的忙

<v-clicks>

<div class="grid grid-cols-3 gap-10 pt-4 -mb-6">

<v-clicks>

<div>
<v-click>

```c{monaco}
#include <stdio.h>
__attribute__((preserve_none)) void g(int x);
__attribute__((noinline, preserve_none)) void g(int x){
    printf("Value: %d\n", x);
}

__attribute__((preserve_none)) void f(int x) {
    [[clang::musttail]] return g(x);
}

int main() {
    f(42);
    return 0;
}

```

</v-click>
</div>

<div>
对应的汇编
</div>

<div>

```assembly{monaco}
0000000000001140 <g>:
    1140:	55                   	push   %rbp
    1141:	48 89 e5             	mov    %rsp,%rbp
    1144:	48 83 ec 10          	sub    $0x10,%rsp
    1148:	44 89 65 fc          	mov    %r12d,-0x4(%rbp)
    114c:	8b 75 fc             	mov    -0x4(%rbp),%esi
    114f:	48 8d 3d ae 0e 00 00 	lea    0xeae(%rip),%rdi        # 2004 <_IO_stdin_used+0x4>
    1156:	31 c0                	xor    %eax,%eax
    1158:	e8 d3 fe ff ff       	call   1030 <printf@plt>
    115d:	48 83 c4 10          	add    $0x10,%rsp
    1161:	5d                   	pop    %rbp
    1162:	c3                   	ret
    1163:	66 66 66 66 2e 0f 1f 	data16 data16 data16 cs nopw 0x0(%rax,%rax,1)
    116a:	84 00 00 00 00 00 

0000000000001170 <f>:
    1170:	55                   	push   %rbp
    1171:	48 89 e5             	mov    %rsp,%rbp
    1174:	44 89 65 fc          	mov    %r12d,-0x4(%rbp)
    1178:	44 8b 65 fc          	mov    -0x4(%rbp),%r12d
    117c:	5d                   	pop    %rbp
    117d:	e9 be ff ff ff       	jmp    1140 <g>
    1182:	66 66 66 66 66 2e 0f 	data16 data16 data16 data16 cs nopw 0x0(%rax,%rax,1)
    1189:	1f 84 00 00 00 00 00 

```

</div>

</v-clicks>

</div>


</v-clicks>

---
transition: view-transition
---

# Python 3.14: π 的 Python

先让我们暂时回到 3.12 

我们都知道 3.12 中，PEP 669 极为低成本的监控方式，可以让我们监控包括不仅限于如下的事件

1. PY_START: Start of a Python function (occurs immediately after the call, the callee’s frame will be on the stack)
2. PY_RESUME: Resumption of a Python function (for generator and coroutine functions), except for throw() calls.
3. PY_THROW: A Python function is resumed by a throw() call.
4. PY_RETURN: Return from a Python function (occurs immediately before the return, the callee’s frame will be on the stack).
5. PY_YIELD: Yield from a Python function (occurs immediately before the yield, the callee’s frame will be on the stack).
6. PY_UNWIND: Exit from a Python function during exception unwinding.
7. CALL: A call in Python code (event occurs before the call).
8. C_RETURN: Return from any callable, except Python functions (event occurs after the return).
9. C_RAISE: Exception raised from any callable, except Python functions (event occurs after the exit).
10. RAISE: An exception is raised, except those that cause a STOP_ITERATION event.
11. EXCEPTION_HANDLED: An exception is handled.
12. LINE: An instruction is about to be executed that has a different line number from the preceding instruction.
13. INSTRUCTION – A VM instruction is about to be executed.
14. JUMP – An unconditional jump in the control flow graph is made.
15. BRANCH – A conditional branch is taken (or not).
16. STOP_ITERATION – An artificial StopIteration is raised; see the STOP_ITERATION event。

---
transition: view-transition
---

# Python 3.14: π 的 Python

那么我们怎么去对已有的代码进行分析呢？

```python
import os
import sys
import tempfile

# Create a temporary script
with tempfile.NamedTemporaryFile(mode='w', suffix='.py', delete=False) as f:
    script_path = f.name
    f.write(f"import my_debugger; my_debugger.connect({os.getpid()})")
try:
    # Execute in process with PID 1234
    print("Behold! An offering:")
    sys.remote_exec(1234, script_path)
finally:
    os.unlink(script_path)
```

