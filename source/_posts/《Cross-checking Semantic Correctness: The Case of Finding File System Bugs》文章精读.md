---
title: "《Cross-checking Semantic Correctness: The Case of Finding File System Bugs》文章精读"
category: CS&Maths
#id: 57
date: 2023-8-31 20:42:32
tags: 
  - Linux
  - File System
  - Staitc Analysis
  - SOSP
  - 论文
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: article  # 展示在时间线列表中
---

主要思路是统计多个文件系统的实现，计算具体文件系统与多个文件系统总体之间的差异性（直方图/信息熵），从而窥探出其文件系统的具体实现在语义上的差异。
<!--more-->

## 设计
![Overview](/《Cross-checking Semantic Correctness: The Case of Finding File System Bugs》文章精读/image1.png)

1. To enable comparison, the source code of each file system is merged into one large file. This allows interprocedural analysis within a file system.
2. JUXTA collects execution information for each function in the file systems by constructing control-flow graphs (CFGs) and symbolically exploring these graphs from the entry point to the end of each function.
3. Symbols are canonicalized so that equivalent symbols have the same name across file systems. A path database that stores the extracted path information is also created, which allows applications like checkers and specification extractors to use the path information without reexecuting symbolic analysis. All of these facilitate comparison.
4. Symbolic execution is used to explore paths through functions and build a path database. The path info contains path conditions, return values, side effects, etc.
5. Two statistical methods are used to compare paths:
   - Histogram-based: Integer ranges are encoded into histograms and distances between histograms are computed.
   - Entropy-based: Used for **events** like flags or return value checks. Low non-zero entropy indicates a likely bug.（别的的实现均不包含这一event，则说明this event could be wrong implementation）
  
  ![Histogram-based comparison](/《Cross-checking Semantic Correctness: The Case of Finding File System Bugs》文章精读/image2.png)
  
1. Bug reports are ranked to prioritize investigation of likely true positives. Metrics include histogram distance and entropy values.

## 实现

需要构建8个检查器：

1. 返回值检查器：比较不同文件系统对同一VFS接口的返回值，找到返回值有明显差异的文件系统。
2. 副作用检查器：比较同一VFS接口和返回值下的副作用，找到副作用有明显差异的文件系统。
3. 函数调用检查器：比较同一VFS接口下的函数调用，找到调用函数有明显差异的文件系统。
4. 路径条件检查器：比较同一VFS接口下的路径条件，找到路径条件有明显差异的文件系统。
5. 提取文件系统规范：从不同文件系统中提取出共性行为作为潜在规范。
6. 重构跨模块抽象：根据共性行为，可将其提升到VFS层，减少冗余。
7. 推断锁语义：推断每个路径所持有和释放的锁，跟踪受锁保护的字段。
8. 推断外部API语义：基于调用参数和返回值检查的频率分布，找出用法有明显差异的文件系统。

## 问题

1. 依赖于大量实现相同功能的模块

## 实验复现
### 实验环境
- docker ubuntu 14.04
- gcc 4.8
- g++ 4.8

### 复现步骤

#### 下载 juxta
```shell
git clone https://github.com/sslab-gatech/juxta.git
```

#### 下载并测试 Linux
```shell
git clone https://github.com/torvalds/linux.git
cd linux
git checkout v4.0-rc2
cp ../juxta/config/config-x86_64-full-fs-4.0 .config
make; make clean
cd ../juxta
```

#### 编译 clang
在编译 clang 之前在 juxta 的 `Makefile` 中增加`COMPILER := -DCMAKE_CXX_COMPILER=g++-4.8 -DCMAKE_C_COMPILER=gcc-4.8`，并且将其添加到 cmake 的参数中。
```shell
make clang-full       # first time only
make clang            # from the next
```

#### 构建 path database
合并文件系统代码
```shell
cd analyzer
./ctrl.py merge_all   # for all file systems
./ctrl.py merge ext4  # for ext4
```

对合并的文件系统代码静态分析
```shell
./ctrl.py clang_all   # for all file systems
./ctrl.py clang ext4  # for ext4
```

构建路径数据库
```shell
./ctrl.py pickle_all  # for all file systems
```

#### code checker
```shell
./ckrtn.py            # Return code checker
./ckstore.py          # Side-effect checker
./ckcall.py           # Function call checker
./ckcond.py           # Path condition checker
./call_flags.py       # Argument checker
./ckapi.py            # Error handling checker
./lock.py             # Lock checker
./spec.py             # Spec. generator
```

#### get sorted results of checker output
```shell
./catcklog.sh [checker output dir]
```
## 代码分析
### 目录结构
```
.
|-- LICENSE
|-- Makefile
|-- README.md
|-- analyzer  // 代码分析工具
|-- bin
|-- config    // 保存有Linux的编译配置
|-- llvm      // llvm的源码
`-- scripts
```
analyzer目录如下所示：
```
.
|-- NOTE.fops-3.17      // 整理了对文件系统的操作
|-- NOTE.fops-4.0
|-- NOTE.fs
|-- README
|-- _eval-grep-count.sh
|-- analyzer.py
|-- api_parser.py
|-- argnorm.py
|-- batch_bug_dist.py
|-- bug_dist.py
|-- bugginess.py
|-- call_flags.py
|-- catckfn.sh
|-- catcklog.sh
|-- checker.py
|-- ckapi.py
|-- ckcall.py
|-- ckcond.py
|-- ckrtn.py
|-- ckstore.py
|-- color.py
|-- condition_container.py
|-- ctrl.py
|-- data
|-- dbg.py
|-- dump-ast.py
|-- eval-count.py
|-- eval-grep-count.py
|-- fgrep.py
|-- fsop.py
|-- inc
|-- libs
|-- lock.py
|-- lock_promo.py
|-- lock_promo_check.py
|-- lock_range.py
|-- lock_range_cmp.py
|-- merger.py
|-- mining.py
|-- ops.py
|-- parser.py
|-- path.py
|-- path_container.py
|-- pathbin.py
|-- pickler.py
|-- proof.z3.template
|-- pygments -> libs/pygments/pygments
|-- pymake -> libs/pymake/pymake
|-- return_cond.py
|-- return_cond_container.py
|-- rsf.py
|-- rsv.py
|-- scripts
|-- spec.py
|-- utils.py
|-- z3 -> libs/z3
`-- z3helper.py
```

### ctrl.py 文件
主要功能包括:

- merge:将多个文件系统的源代码合并成一个文件,方便进行静态分析。
- clang:使用Clang对合并后的代码进行静态分析,生成结果文件。
- grep_decl:搜索并输出文件系统的常见操作函数声明。
- analyze_lock:分析文件系统对锁的使用情况。
- analyze_lock_range:分析文件系统锁的作用范围。
- analyze_lock_promo:分析文件系统锁的提升情况。
- status:显示文件合并和静态分析的状态。

主要流程:

- 通过merge命令将多个文件系统的源代码合并成一个文件,存放在out目录下。
- 通过clang命令使用Clang对合并后的代码进行静态分析,生成结果文件存放在clang-log目录下。
- 分析脚本从clang-log目录读取结果文件,进行各种分析,如锁使用分析等,结果输出到results目录下。
- status命令可以查看合并和分析的状态。

### merger.py 文件

#### merge_fs函数
`adjust_file_path(src_d, files)`输入的`src_d`是Linux某一文件系统的绝对路径，`files`是该文件系统具体文件，`adjust_file_path`函数返回相对于当前目录（analyzer）的路径。

`prepare_dir(fs, opts.linux, src_d, dst_d, cflags)`将文件系统下的`Kconfig`、`Makefile`、以及`.h`文件复制到`analyzer/out`目录对应的文件系统下，并创建用于构建模块的Makefile.build和Makefile（舍弃了之前复制过来的）。

随后将代码合并并写入`one.c`文件中，每个文件系统都有一个`one.c`文件。再对预处理后的代码进行重写，将特定符号替换为新的符号，并将重写后的代码写入输出文件。还会返回符号表和重写计划用于调试。

```python
# process a fs
def merge_fs(opts, fs):
    src_d = os.path.join(opts.linux, "fs", fs)
    dst_d = "out/%s" % fs
    kconf = load_kconfig(os.path.join(opts.linux, ".config"))

    (target, files, cflags) = parse_makefile(kconf, src_d)

    print("> fs   : %s" % target)
    print("> files: %s" % files)

    if fs != target:
        print("! WARNING: you might not want: %s vs %s?" % (fs, target))

    # adjust path
    files = adjust_file_path(src_d, files)

    # copy non-c files
    prepare_dir(fs, opts.linux, src_d, dst_d, cflags)

    # preprocess: opt out headers & expand
    codes = preprocess(fs, src_d, files)

    # rewrite
    out = os.path.join(dst_d, "one.c")
    (out_symbols, out_rewriting) = rewrite(fs, codes, out)

    # kept them for debugging
    dump_symbols_to_file(dst_d, "rewriting.info", out_rewriting)
    dump_symbols_to_file(dst_d, "symbols.info", out_symbols)

    # final touch
    post_adjust_per_fs(fs, dst_d, out)
```

#### preprocess函数
preprocess(fs, src_d, files)对某一文件系统下的所有文件进行预处理，包括处理头文件引用和内联C文件，并将处理后的代码存储在一个列表中以供后续使用。

```python
# opt out headers & expand
def preprocess(fs, src_d, files):
    headers = set()
    codes = []
    for f in files:
        with open(f) as fd:
            code = []
            for l in preprocess_headers(fs, src_d, fd.read(), headers):
                code.append(l)
            code = "\n".join(code)
            codes.append((f, code))

    return codes
```
#### preprocess_headers函数
`preprocess_headers(fs, src_d, code, headers)`函数预处理文件系统C代码中的头文件。

`preprocess_headers` 函数接受四个参数：

- `fs`：表示文件系统的名称。
- `src_d`：表示文件系统源代码的目录路径。
- `code`：表示要预处理的C代码。
- `headers`：表示已经处理过的头文件集合（一个集合数据结构）。

```python
# preprocess headers
def preprocess_headers(fs, src_d, code, headers):
    prev_inc = False
    for l in code.splitlines():
        m = re.match('#[ \t]*include[ \t]*[<"]([^>"]+)[">]', l)
        if m:
            inc = m.groups()[0]

            # NOTE. include ".c" (by minix)
            if inc.endswith('.c'):
                print("> inlining {}".format(inc))

                # inlining .c code
                yield "/* inlined: " + inc + "*"*40 + "/"
                inlined_code = load_fs_file(fs, os.path.join(src_d, inc))
                for l in preprocess_headers(fs, src_d, inlined_code, headers):
                    yield l
                yield "/" + "*"*60 + "/"

                continue

            # to make sure that header files not duplicated
            if inc in headers: 
                l = "// %s" % l 
            else:
                headers.add(inc)
                prev_inc = True
        elif prev_inc:
            yield "#include \"../../inc/__fss.h\""
            if os.path.exists(os.path.join(ROOT, "inc", "__%s.h" % fs)):    # seems never used
                yield "#include \"../../inc/__%s.h\"" % fs
            prev_inc = False
        yield l
```

使用正则表达式 `re.match` 匹配行中的 `#include` 语句。如果匹配成功，将头文件路径提取出来，存储在 `inc` 变量中。
如果头文件路径以 ".c" 结尾，表示要内联这个C文件的代码，然后调用递归函数 `preprocess_headers` 处理内联的C代码。内联的代码会被注释以标识它们是内联的。这可以用于将某个C文件的内容嵌入到当前C代码中。

如果 `inc` 在 `headers` 集合中，表示这个头文件已经被处理过，那么将当前行注释掉。

如果 `inc` 不在 `headers` 集合中，表示这个头文件是第一次遇到，将它添加到 `headers` 集合中，并标记 `prev_inc` 为 `True`。
`prev_inc` 变量用于跟踪前一个行是否是 `#include` 行，在 `#include` 代码块结束时添加`#include "../../inc/__fss.h"`，`__fss.h`用于代码的调试和错误处理。
```c
// __fss.h
// SPDX-License-Identifier: MIT
#ifndef ____FSS_H__
#define ____FSS_H__

#undef BUG
#undef BUG_ON
extern void BUG();
#define BUG_ON(condition) do { if (unlikely(condition)) BUG(); } while (0)
// while(0) is to make sure the sentence will be executed once

#endif /* ____FSS_H__ */

```
最后，将处理后的代码行逐行 `yield` 出来，生成器函数返回处理后的C代码。


#### rewrite函数
`rewrite(fs, codes, out)`将代码中的特定符号（symbols）替换为新的符号，然后将重写后的代码写入输出文件。

`rewritten = highlight(code, CLexer(), TokenRewritter(pn, rewriting_plan))`这一行代码用于重写代码。它执行以下操作：

- 使用 `CLexer()` 创建一个C语言的词法分析器，用于将代码拆分为 tokens。
- 创建一个 `TokenRewritter` 对象，传递文件名 `pn`(path name的缩写，实际上为文件路径) 和重写计划 `rewriting_plan`，以便在词法分析期间替换符号。
- 调用 `highlight` 函数，将 `code` 传递给它，同时传递词法分析器和重写器对象，以执行重写操作。重写后的代码存储在 `rewritten` 变量中。
  
```python
# rewrite codes and flush them to out
def rewrite(fs, codes, out):
    with open(out, "w") as fd:
        (static_symbols, rewriting_plan) = prepare_rewritting(fs, codes)
        for (pn, code) in codes:
            # reschedule static symbols
            rewritten = highlight(code, CLexer(), TokenRewritter(pn, rewriting_plan))
            for l in rewritten.splitlines():
                fd.write(l.encode("utf8"))
                fd.write("\n")

            fd.write("/" + "*" * 60 + "/\n")
            fd.write("/* %s */\n" % pn)
            fd.write("/" + "*" * 60 + "/\n")

    return (static_symbols, rewriting_plan)
```
#### prepare_rewritting函数
`FS_FORCE_REWRITE` 是一个字典，它的键是文件系统的名称，值是需要手动进行符号替换的符号列表。

例如`nfs3_ftypes`，既存在于`nfsd/nfs3xdr.c`中，又存在于`nfsd/nfs3proc.c`中，合并文件后会导致函数重复定义的错误，所以需要手动替换。
```python
# manual opt out
#  (see, logfs defines hash_32() which was included by other c files)
global FS_FORCE_REWRITE
FS_FORCE_REWRITE = {
    "logfs": ["hash_32"],
    "minix": ["DIRECT", "DEPTH", "block_t", "Indirect", "pointers_lock", "block_to_cpu"],
    "ncpfs": ["ncp_symlink", "ncp_symlink_readpage"],
    "jffs2": ["deflate_mutex"],
    "nfsd" : ["NFSDDBG_FACILITY", "nfs3_ftypes"],
}
```

`def _to_canonical(pn, sym)`是一个内部辅助函数，用于将符号名 `sym` 转换为规范化的形式，以确保唯一性。它基于文件名 `pn` 构建了一个规范化的符号名，并返回。

```python
# parse and prepare conflicted symbols
def prepare_rewritting(target, codes):
    global FS_FORCE_REWRITE

    def _to_canonical(pn, sym):
        base = os.path.basename(pn)
        return sym + "_" + base.replace(".", "_").replace("-", "_")

    # get static symbols
    static_symbols = {}
    for (pn, code) in codes:
        formatter= StaticDecl()
        highlight(code, CLexer(), formatter)

        print("> %-50s: %d" % (pn, len(formatter.table)))
        static_symbols[pn] = formatter.table

    # check collisions
    rewriting_plan = defaultdict(dict)
    for (pivot_pn, pivot_tbl) in static_symbols.iteritems():
        # rewrite if collapsed
        for sym in pivot_tbl:
            for (target_pn, target_tbl) in static_symbols.iteritems():
                if pivot_pn == target_pn:
                    continue
                if sym in target_tbl:
                    print("> %s collaposed with %s & %s" % (sym, pivot_pn, target_pn))
                    rewriting_plan[pivot_pn][sym] = _to_canonical(pivot_pn, sym)

        # update pivot_tbl to minize rewriting
        for (sym, new_sym) in rewriting_plan[pivot_pn].iteritems():
            pivot_tbl.remove(sym)
            pivot_tbl.add(new_sym)

    # manual rewriting (e.g., collision in headers)
    for sym in FS_FORCE_REWRITE.get(target, []):
        print("> manually include %s" % sym)
        for (pivot_pn, pivot_tbl) in static_symbols.iteritems():
            rewriting_plan[pivot_pn][sym] = _to_canonical(pivot_pn, sym)

    return (static_symbols, rewriting_plan)
```
#### StaticDecl类
16 - 31 行的目的是为了找出所有的静态函数并将函数名添加到`self.table`中（除去函数名开头为`DEFINE_`、`LIST_HEAD`、`LLIST_HEAD`、`DECLARE_DELAYED_WORK`的函数）。

32 - 44 行的目的是为了找出所有的结构体并将结构体名添加到`self.table`中。

最后一行`lookup.append((ttype, value))`跳过了`self.blacklist`（不包括预处理指令中的，预处理指令中的格式是`Token.Comment.Preproc, u'define nfsd3_voidres\t\t\tnfsd3_voidargs'`，没有单独的`nfsd3_voidargs`）和函数名开头为`DEFINE_`、`LIST_HEAD`、`LLIST_HEAD`、`DECLARE_DELAYED_WORK`的函数。
```python
class StaticDecl(Formatter):
    def __init__(self):
        self.table = set()
        # NOTE. nothing yet
        self.blacklist = set(["nfsd3_voidargs"])

    def is_dot_tok(self, token, value):     # it seems never used
        return token is Token.Punctuation and value == "."

    def format(self, tokensource, outfile): # outfile never used
        lookup = []
        for ttype, value in tokensource:
            # strong blacklisting
            if value in self.blacklist:
                continue
            if ttype is Token.Name.Function:
                # e.g., DEFINE_SPINLOCK() or DEFINE_RWLOCK()
                if value.startswith("DEFINE_") \
                   or value.startswith("LIST_HEAD") \
                   or value.startswith("LLIST_HEAD") \
                   or value.startswith("DECLARE_DELAYED_WORK"):
                    continue

                # NOTE. arbitrary tokens can be inserted before static
                # shows up, but usually 5.
                is_static = False   # never used
                for i in range(8):
                    if lookup[-i][0] is Token.Keyword \
                       and lookup[-i][1] == "static":
                        self.table.add(value)
                        break
            if ttype is Token.Punctuation and value == "{":
                struct_name = None
                struct_keyword = None
                for i in range(5):
                    (typ, val) = lookup[-i]
                    if typ is Token.Keyword and val == "struct":
                        struct_keyword = True
                        break
                    if typ is Token.Name and struct_keyword is None:
                        struct_name = val

                if struct_keyword and struct_name:
                    self.table.add(struct_name)

            lookup.append((ttype, value))
```

#### TokenRewritter类
`TokenRewritter`是一个自定义的Pygments格式化器类，它继承自Pygments中的 `Formatter` 类。这个类的主要目的是在代码高亮和着色的过程中，根据预定的重写计划对特定符号进行替换。
```python
class TokenRewritter(Formatter):
    def __init__(self, pn, plan):
        self.pn = pn
        self.plan = plan
        self.syms = plan[pn]
        self.stat = Counter()
        self.lookup = []

    def is_dot_tok(self, token, value):
        return token is Token.Punctuation and value == "."

    def format(self, tokensource, outfile):
        lookup = []
        for ttype, value in tokensource:
            # tokens used in macro
            if ttype is Token.Comment.Preproc:
                for (k, v) in self.syms.iteritems():
                    if k in value:
                        print("> rewrite (mcro) %s -> %s" % (k, v))
                        value = value.replace(k, v)
                        self.stat[value] += 1
            # check regular function call
            if ttype is Token.Name \
                    and not self.is_dot_tok(*lookup[-1]):
                #
                # NOTE (ignore, wrong pygment parser)
                #   struct.func_ptr => [Token.Punctuation][Toen.Name]
                #
                new_sym = self.syms.get(value, None)
                if new_sym:
                    self.stat[value] += 1
                    print("> rewrite (call) %s -> %s" % (value, new_sym))
                    value = new_sym

            # check func decls
            if ttype is Token.Name.Function or ttype is Token.Keyword.Type:
                new_sym = self.syms.get(value, None)
                if new_sym:
                    print("> rewrite (decl) %s -> %s" % (value, new_sym))
                    value = new_sym

            lookup.append((ttype, value))

            outfile.write(value)
            lookup.append((ttype, value))
```
`pn`表示当前处理的文件名（包含路径）。
`plan`表示重写计划，一个字典，包含了要替换的符号对应关系。

`Token.Comment.Preproc`（预处理器注释令牌）是指用于表示预处理器指令或注释的特殊类型的标记（Token）。在 C 语言中常见的可标注为`Token.Comment.Preproc`的例子如下：

1. 条件编译指令：
```c
#if DEBUG
    // 这是一个调试模式下的代码块
#endif
```
2. 宏定义：
```c
#define MAX_SIZE 100
```


## 结果分析
TODO

## 改进
TODO

向量数据库？
相关系数矩阵？协方差矩阵？等传统数学建模技术