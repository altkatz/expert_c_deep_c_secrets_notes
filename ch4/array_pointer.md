A live example(platform Mac OS X 10.0 x86_64) for Ch4 4.3.2
* array.c

```
int a[10];
```

* main.c

```
extern int a[];
int main(){
  a[0]=10;
}
```

* compile: `clang -g -o main array.c main.c` run './main' no segmentfault !

* disas `lldb ./main`

```
(lldb) disassemble -n main
main`main at main.c:2:
main[0x100000f80]:  pushq  %rbp
main[0x100000f81]:  movq   %rsp, %rbp
main[0x100000f84]:  movl   $0x0, %eax
main[0x100000f89]:  leaq   0x70(%rip), %rcx
main[0x100000f90]:  movl   $0xa, (%rcx)
main[0x100000f96]:  popq   %rbp
main[0x100000f97]:  retq
```


Now, change `extern int a[];` in main.c to `extern int *`, and compile and run, Bang! Segment fault! lets disas in lldb:

```
(lldb) disassemble -n main
main`main at main.c:2:
main[0x100000f80]:  pushq  %rbp
main[0x100000f81]:  movq   %rsp, %rbp
main[0x100000f84]:  movl   $0x0, %eax
main[0x100000f89]:  leaq   0x70(%rip), %rcx
main[0x100000f90]:  movq   (%rcx), %rcx
main[0x100000f93]:  movl   $0xa, (%rcx)
main[0x100000f99]:  popq   %rbp
main[0x100000f9a]:  retq
```

Notice the extra instruction "movq   (%rcx), %rcx" at 0x100000f90: that's what happened when we mistreated array as pointer: the array `a` 's compile time constant address(0x100001000, equals to rip(0x100000f90)+0x70) is mistakenly treated as the pointer variable's address, to get the pointer's value, an extra dereference is made.
