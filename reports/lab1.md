
# Table of Contents

1.  [功能实现](#orgd8e687b)
2.  [问答题](#org495684d)
3.  [荣誉准则](#org25db33d)

[LearningOs-stage2](20231021014240-learningos_stage2.md)


<a id="orgd8e687b"></a>

# 功能实现

-   通过在trap模块添加代码，监测系统调用次数
-   通过在run<sub>first</sub><sub>app添加时间记录</sub>，来记录开始时间，然后在系统调用sys<sub>info的时候</sub>，计算时间差，得到运行时间
-   在run<sub>first</sub><sub>app和run</sub><sub>next时</sub>，修改运行状态为running


<a id="org495684d"></a>

# 问答题

-   1.正确进入 U 态后，程序的特征还应有：使用 S 态特权指令，访问 S 态寄存器后会报错。 请同学们可以自行测试这些内容 (运行 Rust 三个 bad 测例 (ch2b<sub>bad</sub><sub>\*</sub>.rs) ， 注意在编译时至少需要指定 LOG=ERROR 才能观察到内核的报错信息) ， 描述程序出错行为，同时注意注明你使用的 sbi 及其版本。
    
    > 内核出错的行为:
    > [kernel] PageFault in application, bad addr = 0x0, bad instruction = 0x804003c4, kernel killed it.
    > sbi版本:
    > [rustsbi] RustSBI version 0.3.0-alpha.2, adapting to RISC-V SBI v1.0.0

-   2.深入理解 trap.S 中两个函数 \_<sub>alltraps</sub> 和 \_<sub>restore</sub> 的作用，并回答如下问题:
    -   1.L40：刚进入 \_<sub>restore</sub> 时，a0 代表了什么值。请指出 \_<sub>restore</sub> 的两种使用情景。
        
        > 1.刚进入\_<sub>restore时</sub>，a0代表了之前在\_<sub>alltraps备份的sp内核栈</sub>
        > 
        > 2.\_<sub>restore的两种使用场景</sub>
        > 1).trap<sub>handler</sub> return 比如系统调用完成, 的时候，用来恢复之前的上下文,各种寄存器的值
        > 2).在run<sub>next</sub><sub>app的时候</sub>，复用\_<sub>restore来初始化U状态上下文</sub>
    
    -   2. L43-L48：这几行汇编代码特殊处理了哪些寄存器？这些寄存器的的值对于进入用户态有何意义？请分别解释。
        
            ld t0, 32*8(sp)
            ld t1, 33*8(sp)
            ld t2, 2*8(sp)
            csrw sstatus, t0
            csrw sepc, t1
            csrw sscratch, t2
        
        > 恢复了csr控制寄存器，sstatus,sscratch,sepc
        > 对于用户态的意义:
        > 
        > -   1.切换用户栈
        >     sscratch在\_<sub>alltraps时</sub>->kernel stack,sp->user stack
        > 
        > \_<sub>restore进入用户态后</sub>，csrrw sp,sscratch,sp ，可以将sp恢复为user stack
        > 
        > -   2.sstatus可以在准备从S进入U(开始执行下个app时)时，修改为U状态
        > 
        > sstatus.set<sub>spp</sub>(SPP::User);
        > 
        > -   3.sepc用来设置应用程序入口地址0x80400000,或者syscall后的下一条指令地址,
        >     即trap完后到哪执行？
        > 
        > 上面这几个在代码里设置好了，在\_<sub>restore里进行恢复U态上下文</sub>
        
        -   3. L50-L56：为何跳过了 x2 和 x4？

> sp（x2) 在\_<sub>alltraps时是内核栈</sub>，并且带着保存的上下文通过a0到了 trap<sub>handler</sub>
> 最后通过a0进入了\_<sub>alllrestore,没有保存</sub>，因此也不用ld
> 
> x4 app没有用到，所以不用保存和加载

-   4.L60：该指令之后，sp 和 sscratch 中的值分别有什么意义？
    
        csrrw sp, sscratch, sp
    
    > 将sp恢复为用户栈

-   5.\_<sub>restore</sub>：中发生状态切换在哪一条指令？为何该指令执行之后会进入用户态？
    
    > sret，指令，cpu会将当前特权等级按照sstatus的spp字段设置为U或S，然后跳转到sepc寄存器指向的位置，继续执行


<a id="org25db33d"></a>

# 荣誉准则

-   1.本次实验没有交流对象
-   2.我参考了以下资料
    -   RISC-V版计算机组成与设计，
    -   rCore 简明教程 2023A
    -   RISC-V Reader
    -   stackoverflow关于栈指针的说明
    -   RISC-V官方文档
    -   rust文档
-   3. 我独立完成了本次实验除以上方面之外的所有工作，包括代码与文档。 我清楚地知道，从以上方面获得的信息在一定程度上降低了实验难度，可能会影响起评分。

-   4. 我从未使用过他人的代码，不管是原封不动地复制，还是经过了某些等价转换。 我未曾也不会向他人（含此后各届同学）复制或公开我的实验代码，我有义务妥善保管好它们。 我提交至本实验的评测系统的代码，均无意于破坏或妨碍任何计算机系统的正常运转。 我清楚地知道，以上情况均为本课程纪律所禁止，若违反，对应的实验成绩将按“-100”分计。

