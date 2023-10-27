---
title: "Cross-checking Semantic Correctness: The Case of Finding File System Bugs 阅读"
category: CS&Maths
#id: 57
date: 2023-8-31 20:42:32

tags: 
  - Linux
  - File System
  - Staitc Analysis
  - Symbolic Execution
  - SOSP
  - SOSP'15
  - Paper
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: article  # 展示在时间线列表中
---

主要思路是统计多个文件系统的实现，计算具体文件系统与多个文件系统总体之间的差异性（直方图/信息熵），从而窥探出其文件系统的具体实现在语义上的差异。
<!--more-->

## 设计
![Overview](/Cross-checking Semantic Correctness: The Case of Finding File System Bugs 阅读/image1.png)

1. To enable comparison, the source code of each file system is merged into one large file. This allows interprocedural analysis within a file system.
2. JUXTA collects execution information for each function in the file systems by constructing control-flow graphs (CFGs) and symbolically exploring these graphs from the entry point to the end of each function.
3. Symbols are canonicalized so that equivalent symbols have the same name across file systems. A path database that stores the extracted path information is also created, which allows applications like checkers and specification extractors to use the path information without reexecuting symbolic analysis. All of these facilitate comparison.
4. Symbolic execution is used to explore paths through functions and build a path database. The path info contains path conditions, return values, side effects, etc.
5. Two statistical methods are used to compare paths:
   - Histogram-based: Integer ranges are encoded into histograms and distances between histograms are computed.
   - Entropy-based: Used for **events** like flags or return value checks. Low non-zero entropy indicates a likely bug.（别的的实现均不包含这一event，则说明this event could be wrong implementation）
  
  ![Histogram-based comparison](/Cross-checking Semantic Correctness: The Case of Finding File System Bugs 阅读/image2.png)
  
6. Bug reports are ranked to prioritize investigation of likely true positives. Metrics include histogram distance and entropy values.

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
主要功能包括：

- merge：将多个文件系统的源代码合并成一个文件，方便进行静态分析。
- clang：使用Clang对合并后的代码进行静态分析，生成结果文件。
- grep_decl：搜索并输出文件系统的常见操作函数声明。
- analyze_lock：分析文件系统对锁的使用情况。
- analyze_lock_range：分析文件系统锁的作用范围。
- analyze_lock_promo：分析文件系统锁的提升情况。
- status：显示文件合并和静态分析的状态。

主要流程：

- 通过merge命令将多个文件系统的源代码合并成一个文件，存放在out目录下。
- 通过clang命令使用Clang对合并后的代码进行静态分析，生成结果文件存放在clang-log目录下。
- 分析脚本从clang-log目录读取结果文件，进行各种分析，如锁使用分析等，结果输出到results目录下。
- status命令可以查看合并和分析的状态。

#### merge 操作
```python
def cmd_merge(opts, args):
    """merge fs specified (e.g., 'merge ext3 ext4')"""

    import merger
    for fs in args:
        merger.merge_fs(opts, fs)
```
当执行`./ctrl.py merge ext3`的时候，调用这里的`cmd_merge`函数，该函数调用下文写到的`merger`模块中的`merge_fs`函数，实现文件系统的合并。

```python
def cmd_merge_all(opts, _):
    """merge all fs (e.g., 'merge_all')"""

    pool = mp.Pool(mp.cpu_count())
    for fs in fsop.get_fs("M"):
        pool.apply_async(_run_merger, args = (fs, opts.linux))
    pool.close()
    pool.join()

    return 0
```
`./ctrl.py merge_all`的情况则更为复杂一点。通过调用`fsop.get_fs`函数来获得需要被合并的文件列表。

`fsop`的`get_fs`函数如下所示，先读取`analyzer`目录下的`NOTE.fs`文件，从中获得内容的格式为
```
[配置参数] 键名 其他文本
```

`配置参数`有

- M: known to be merged as a single code
- C: known to run it with clang pass
- F: failed with clang pass

此处`get_fs`将配置参数为`M`的文件系统列表返回给`cmd_merge_all`，`cmd_merge_all`并行进行文件系统合并。
```python
ROOT = os.path.dirname(__file__)

# XXX. hide
def load_conf(pn):
    conf = []
    for l in open(pn):
        m = re.match("\[([\\w ]*)\] (\\w+).*", l)
        if m:
            tok = m.groups()
            conf.append((tok[0].strip(), tok[1]))
    return conf

global CONF
CONF = load_conf(os.path.join(ROOT, "NOTE.fs"))

def get_fs(kind):
    global CONF
    rtn = []
    for (tag, fs) in CONF:
        if all(t in tag for t in kind):
            rtn.append(fs)
    return rtn
```
以ext3文件系统为例，执行完merge操作后的`out/ext3`目录下的文件如下所示：
```
.
|-- Kconfig
|-- Makefile
|-- Makefile.build
|-- acl.h
|-- ext3.h
|-- namei.h
|-- one.c
|-- rewriting.info
|-- symbols.info
`-- xattr.h
```
具体各文件是如何生成的将在[merger.py 文件](#merger.py-文件)中具体介绍。

#### clang 操作（静态分析）
类似于`cmd_merge_all`函数，`cmd_clang`和`cmd_clang_all`函数通过调用`_run_clang`函数进行静态分析。
```python
def _run_clang(fs, clang):
    one_d = join(ROOT, "out", fs)
    if not os.path.exists(one_d):
        print("ERROR: %s doesn't exist (need to merge first)" % one_d)
        return None

    out_d = join(ROOT, "out", fs, "clang-log")

    # could you tweak this to run clang pass
    env = copy.copy(os.environ)
    env["CCC_CC"] = clang
    env["CLANG"] = clang

    mkdirp(out_d)

    clang_result = join(out_d, "fss_output")
    clang_stdout = open(join(out_d, "log.stdout"), "w")
    clang_stderr = open(join(out_d, "log.stderr"), "w")

    with chdir(one_d):
        env["PWD"] = os.getcwd()
        p = subprocess.Popen([FSCK,
                              "--use-analyzer=%s" % clang,
                              "--fss-output-dir=%s" % clang_result,
                              "make",
                              "CC=%s" % clang,
                              "-f", "Makefile.build"],
                             stdout=clang_stdout,
                             stderr=clang_stderr,
                             cwd=os.getcwd(),
                             env=env)
        print("[%s] parsing %s" % (p.pid, fs))
        p.wait()

    del clang_stdout
    del clang_stderr
```
当执行`./ctrl.py merge ext3`时，传递给`subprocess.Popen`的参数为`['/home/juxta/analyzer/../llvm/tools/clang/tools/scan-build/fss-build', '--use-analyzer=/home/juxta/analyzer/../bin/llvm/bin/clang', '--fss-output-dir=/home/juxta/analyzer/out/ext3/clang-log/fss_output', 'make', 'CC=/home/juxta/analyzer/../bin/llvm/bin/clang', '-f', 'Makefile.build']`。

相较于merge之后，`out/ext3`目录下多出来了如下文件。其中`log.stdout`和`log.stderr`是Clang的标准输出和标准错误日志。`make`命令将其以模块的形式编译。
```
.
|-- Module.symvers
|-- clang-log
|   |-- fss_output
|   |   |-- analyzer
|   |   |   `-- out
|   |   |       `-- ext3
|   |   |           |-- ext3.mod.c.fss
|   |   |           `-- one.c.fss
|   |   |-- clang_output_25P0jk
|   |   `-- dev
|   |       `-- null.fss
|   |-- log.stderr
|   `-- log.stdout
|-- ext3.ko
|-- ext3.mod.c
|-- ext3.mod.o
|-- ext3.o
|-- modules.order
`-- one.o
```
接下来我们将进入`analyzer/../llvm/tools/clang/tools/scan-build`目录下，来[深入探究一下`fss-build`文件做了哪些工作](#fss-build-脚本)。

### merger.py 文件

#### merge_fs 函数
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
`adjust_file_path(src_d, files)`输入的`src_d`是Linux某一文件系统的绝对路径，`files`是该文件系统具体文件，`adjust_file_path`函数返回相对于当前目录（analyzer）的路径。

`prepare_dir(fs, opts.linux, src_d, dst_d, cflags)`将文件系统下的`Kconfig`、`Makefile`、以及`.h`文件复制到`analyzer/out`目录对应的文件系统下，并创建用于构建模块的Makefile.build和Makefile（舍弃了之前复制过来的）。

随后将代码合并并写入`one.c`文件中，每个文件系统都有一个`one.c`文件。再对预处理后的代码进行重写，将特定符号替换为新的符号，并将重写后的代码写入输出文件。还会返回符号表和重写计划用于调试。

最后注释掉或修复特定文件中的特定代码段。自此就完成了文件系统的合并。

#### preprocess 函数
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
#### preprocess_headers 函数
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


#### rewrite 函数
`rewrite(fs, codes, out)`将代码中的特定符号（symbols）替换为新的符号，然后将重写后的代码写入输出文件。

首先利用`prepare_rewritting`函数生成重写计划。

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
#### prepare_rewritting 函数
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
`FS_FORCE_REWRITE` 是一个字典，它的键是文件系统的名称，值是需要手动进行符号替换的符号列表。

例如`nfs3_ftypes`，既存在于`nfsd/nfs3xdr.c`中，又存在于`nfsd/nfs3proc.c`中，合并文件后会导致函数重复定义的错误，所以需要手动替换。


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
首先先通过名为`formatter`的`StaticDecl`对象，使用`highlight`函数，`CLexer()`词法分析器和`formatter`来解析代码，获取`static symbols`。

`static symbols`的格式如下所示：
```
{'../../linux/fs/minix/namei.c': set([u'minix_create', u'minix_rename', u'minix_unlink', u'add_nondir', u'minix_mknod', u'minix_lookup', u'minix_link', u'minix_mkdir', u'minix_rmdir', u'minix_tmpfile', u'minix_symlink']), 
'../../linux/fs/minix/inode.c': set([u'V2_minix_iget', u'minix_destroy_inode', u'minix_fill_super', u'minix_writepage', u'exit_minix_fs', u'minix_write_inode', u'minix_remount', u'minix_evict_inode', u'minix_get_block', u'destroy_inodecache', u'minix_i_callback', u'minix_mount', u'minix_write_failed', u'minix_bmap', u'minix_alloc_inode', u'minix_statfs', u'V1_minix_iget', u'init_once', u'init_inodecache', u'minix_readpage', u'minix_put_super', u'minix_write_begin', u'init_minix_fs']), 
...}
```

随后创建`rewriting_plan`字典，用于存储重写计划。它将跟踪哪些符号需要进行重写，以解决符号冲突问题。

20-33行检查符号冲突和生成重写计划。外部循环遍历`static_symbols`字典的键值对，其中`pivot_pn`是文件名，`pivot_tbl`是该文件的静态符号表。
内部循环遍历同样的`static_symbols`字典，但排除了与外部文件名相同的文件。如果在不同文件中的找到相同的符号，会调用`_to_canonical`函数，更新外部循环对应文件的符号重写计划。`_to_canonical(pn, sym)`是一个内部辅助函数，为符号 `sym` 生成一个新的规范名称，以确保符号的唯一性。一个文件的符号重写计划处理完之后（第二个`for`循环执行结束后）会更新该文件的符号表为重写后的符号，以最小化重写。

36-39行制作`FS_FORCE_REWRITE`中符号的重写计划。

最后，函数返回一个包含两个元素的元组，第一个元素是新的`static_symbols`字典，第二个元素是静态符号的`rewriting_plan`字典。




#### StaticDecl 类
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
16 - 31 行的目的是为了找出所有的静态函数并将函数名添加到`self.table`中（除去函数名开头为`DEFINE_`、`LIST_HEAD`、`LLIST_HEAD`、`DECLARE_DELAYED_WORK`的函数）。

32 - 44 行的目的是为了找出所有的结构体并将结构体名添加到`self.table`中。

`self.table`中存储的即为static symbols。

最后一行`lookup.append((ttype, value))`跳过了`self.blacklist`（不包括预处理指令中的，预处理指令中的格式是`Token.Comment.Preproc, u'define nfsd3_voidres\t\t\tnfsd3_voidargs'`，没有单独的`nfsd3_voidargs`）和函数名开头为`DEFINE_`、`LIST_HEAD`、`LLIST_HEAD`、`DECLARE_DELAYED_WORK`的函数。

#### TokenRewritter 类
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

`plan`表示重写计划，一个字典，包含了要替换的符号对应关系，将旧的标识符映射到新的标识符或宏定义。

`self.stat` 是一个 `Counter` 对象，用于跟踪替换操作的统计信息。

`self.lookup` 是一个空列表，用于存储遍历Token时的历史信息。


### Clang 相关代码解析
Juxta 所用 Clang 为修改后的版本，其之间的差别见 [https://github.com/jtzhpf/llvm-project-juxta-sosp/blob/juxta-sosp/README.md](https://github.com/jtzhpf/llvm-project-juxta-sosp/blob/juxta-sosp/README.md)。

#### Options.td 文件
`./llvm/tools/clang/include/clang/Driver/Options.td` 文件定义了clang编译器支持的所有命令行选项。

```TableGen
def ffs_semantic_out_dir_EQ : Joined<["-"], "ffs-semantic-out-dir=">,
    Group<f_Group>, Flags<[DriverOption, CC1Option]>,
    HelpText<"Output directory of fs semantics analysis">;	
def ffs_semantic_EQ : Joined<["-"], "ffs-semantic=">,
    Group<f_Group>, Flags<[DriverOption, CC1Option]>,
    HelpText<"Enable fs semantics analysis">;	
```
`Options.td` 文件新增了上面的代码，这两个选项 `ffs_semantic_out_dir_EQ` 和 `ffs_semantic_EQ` 是 Clang 编译器的命令行选项，用于控制文件系统语义分析（Filesystem Semantics Analysis）的行为。下面是对这两个选项的详细解释：

1. `ffs_semantic_out_dir_EQ` 选项：
   - `ffs_semantic_out_dir_EQ` 是一个带有参数的选项，它的形式是 `-ffs-semantic-out-dir=<directory>`。
   - `<directory>` 是用户提供的路径，用于指定文件系统语义分析的输出目录。
   - 这个选项属于 `f_Group` 组，表示它与 `-f` 开头的选项相关联。
   - 该选项被标记为 `DriverOption` 和 `CC1Option`，表示它既可以在编译器驱动程序中使用，也可以在 Clang 的前端（`-cc1`）中使用。
   - `HelpText` 字段提供了对该选项用途的描述，它说明了该选项用于指定文件系统语义分析的输出目录。

2. `ffs_semantic_EQ` 选项：
   - `ffs_semantic_EQ` 也是一个带有参数的选项，它的形式是 `-ffs-semantic=<value>`。
   - `<value>` 是用户提供的值，用于启用或禁用文件系统语义分析。
   - 这个选项同样属于 `f_Group` 组，表示它与 `-f` 开头的选项相关联。
   - 类似地，该选项也被标记为 `DriverOption` 和 `CC1Option`，表示它可以在编译器驱动程序中和 Clang 前端中使用。
   - `HelpText` 字段提供了对该选项用途的描述，它说明了该选项用于启用或禁用文件系统语义分析。

这两个选项通常用于控制编译器在进行文件系统操作时的语义分析行为。用户可以使用 `ffs_semantic_out_dir_EQ` 选项来指定分析结果的输出目录，同时使用 `ffs_semantic_EQ` 选项来启用或禁用文件系统语义分析。

<!--
#### ProgramState.h 文件
`./llvm/tools/clang/include/clang/StaticAnalyzer/Core/PathSensitive/ProgramState.h` 头文件用于clang静态分析器中的路径敏感状态跟踪。

主要功能包括：
- 定义了ProgramState类，用于表示分析中的程序状态。包括环境映射(Env)、存储映射(Store)、泛型数据映射(GDM)等，可以绑定值到语句和位置等。
- 定义了ProgramStateManager类，用于管理ProgramState对象的工厂。提供创建、重用ProgramState的方法。
- 定义了各种遍历ProgramState历史的迭代器。
- 提供了访问和操作ProgramState中环境、存储、泛型数据映射等的方法。
- 提供了各种"assume"方法，在ProgramState上添加约束。
- 一些辅助类，如ScanReachableSymbols用于访问可达符号。
- ProgramStateTrait类模板，用于泛型数据映射中类型安全的访问。
- HistoricalEvent类，记录ProgramState变化事件。
-->

#### fss-build 脚本
`fss-build` 是一个 Perl 脚本，位于`./llvm/tools/clang/tools/scan-build/fss-build`，用于在构建过程中运行 Clang 静态分析工具。`fss-build` 是在 `scan-build` 的基础上修改而来，在这里我们只用到了其中的部分功能。

主要流程:

1. 解析命令行选项，包括启用/禁用检查器、输出目录等。
2. 设置环境变量，将 `CC` 和 `CXX` 重定向到 [`fssccc-analyzer` 和 `fssc++-analyzer` wrapper 脚本](#fssccc-analyzer-与-fssc-analyzer-脚本)。
3. 调用构建命令，如 `make`、`xcodebuild` 等，在构建过程中运行静态分析。
4. 收集并处理分析结果。

```perl
sub RunBuildCommand {
  my $Args = shift;
  my $IgnoreErrors = shift;
  my $Cmd = $Args->[0];
  my $CCAnalyzer = shift;
  my $CXXAnalyzer = shift;
  my $Options = shift;

  if ($Cmd =~ /\bxcodebuild$/) {
    return RunXcodebuild($Args, $IgnoreErrors, $CCAnalyzer, $CXXAnalyzer, $Options);
  }

  # Setup the environment.
  SetEnv($Options);

  if ($Cmd =~ /(.*\/?gcc[^\/]*$)/ or
      $Cmd =~ /(.*\/?cc[^\/]*$)/ or
      $Cmd =~ /(.*\/?llvm-gcc[^\/]*$)/ or
      $Cmd =~ /(.*\/?clang$)/ or
      $Cmd =~ /(.*\/?fssccc-analyzer[^\/]*$)/) {

    if (!($Cmd =~ /fssccc-analyzer/) and !defined $ENV{"CCC_CC"}) {
      $ENV{"CCC_CC"} = $1;
    }

    shift @$Args;
    unshift @$Args, $CCAnalyzer;
  }
  elsif ($Cmd =~ /(.*\/?g\+\+[^\/]*$)/ or
        $Cmd =~ /(.*\/?c\+\+[^\/]*$)/ or
        $Cmd =~ /(.*\/?llvm-g\+\+[^\/]*$)/ or
        $Cmd =~ /(.*\/?clang\+\+$)/ or
        $Cmd =~ /(.*\/?fssc\+\+-analyzer[^\/]*$)/) {
    if (!($Cmd =~ /fssc\+\+-analyzer/) and !defined $ENV{"CCC_CXX"}) {
      $ENV{"CCC_CXX"} = $1;
    }
    shift @$Args;
    unshift @$Args, $CXXAnalyzer;
  }
  elsif ($Cmd eq "make" or $Cmd eq "gmake" or $Cmd eq "mingw32-make") {
    AddIfNotPresent($Args, "CC=$CCAnalyzer");
    AddIfNotPresent($Args, "CXX=$CXXAnalyzer");
    if ($IgnoreErrors) {
      AddIfNotPresent($Args,"-k");
      AddIfNotPresent($Args,"-i");
    }
  }

  return (system(@$Args) >> 8);
}
```




重点在`RunBuildCommand`函数，这个函数将 `CC` 和 `CXX` 重定向到 [`fssccc-analyzer` 和 `fssc++-analyzer`](#fssccc-analyzer-与-fssc-analyzer-脚本)，并通过执行`system(@$Args)`来运行静态分析，其中`$Args`的内容为
```shell
make CC=/home/juxta/analyzer/../bin/llvm/bin/clang -f Makefile.build CC=/home/juxta/llvm/tools/clang/tools/scan-build/fssccc-analyzer CXX=/home/juxta/llvm/tools/clang/tools/scan-build/fssc++-analyzer
```
此时的工作目录为`./analyzer/out/具体文件系统目录`。



#### fssccc-analyzer 与 fssc++-analyzer 脚本
`fssccc-analyzer` 脚本位于 `./llvm/tools/clang/tools/scan-build/fssccc-analyzer`，`fssccc-analyzer`的主要执行流程如下：

> 1 获取编译命令行参数，分析出编译选项、链接选项和输入文件\
> 2 调用系统的编译器/链接器执行编译/链接\
> 3 对每个输入文件:\
> 
>> 3.1 判断文件语言类型\
>> 3.2 构建调用Clang的命令行，包含静态分析选项\
>> 3.3 调用Clang进行编译+分析\
>> 3.4 如果Clang发生崩溃或错误，保存相关文件用于调试\
>> 3.5 提取Clang分析结果到单独的.fss文件\
>
> 4 如果设置了输出目录，将Clang分析结果保存到该目录\
> 5 根据环境变量设置verbose模式，输出执行过程信息\
> 6 返回系统编译器/链接器的返回值

```perl
sub GetCCArgs {
  my $mode = shift;
  my $Args = shift;

  pipe (FROM_CHILD, TO_PARENT);
  my $pid = fork();
  if ($pid == 0) {
    close FROM_CHILD;
    open(STDOUT,">&", \*TO_PARENT);
    open(STDERR,">&", \*TO_PARENT);
    exec $Clang, "-###", $mode, @$Args;
  }
  close(TO_PARENT);
  my $line;
  while (<FROM_CHILD>) {
    next if (!/\s"?-cc1"?\s/);
    $line = $_;
  }

  waitpid($pid,0);
  close(FROM_CHILD);

  die "could not find clang line\n" if (!defined $line);
  # Strip leading and trailing whitespace characters.
  $line =~ s/^\s+|\s+$//g;
  my @items = quotewords('\s+', 0, $line);
  my $cmd = shift @items;
  die "cannot find 'clang' in 'clang' command\n" if (!($cmd =~ /clang/));
  return \@items;
}
```
`GetCCArgs` 函数用于获取编译器**实际**执行的命令行参数。该函数创建了一个子进程，在子进程中执行`$Clang -### $mode @$Args`命令。

>`-###` 是作为命令行参数传递给 `$Clang` 的一个选项。这个选项在 Clang 中具有特殊的含义，它并不是常见的编译或链接选项，而是用于显示编译过程中的详细信息的一种特殊模式。
>
>具体来说，`-###` 在 Clang 中的作用是：
>
>1. **显示编译过程**：当你将 `-###` 选项传递给 Clang 时，它不会实际执行编译操作，而是会将编译过程的详细信息输出到标准错误（stderr）。这包括了 Clang 执行的实际命令以及每个命令的参数。
>2. **调试和诊断**：`-###` 通常用于调试和诊断构建过程。通过查看输出，开发人员可以了解 Clang 在执行编译操作时究竟执行了哪些命令，以及这些命令使用了哪些参数。这有助于识别构建问题和调试构建系统的配置。
>
>**示例：**
>假设你有一个名为 `example.c` 的源代码文件，并且你希望查看 Clang 编译该文件时实际执行的命令，你可以运行如下命令：
>
>```bash
>clang -### example.c
>```
>
>这将导致 Clang 输出类似以下内容的信息：
>
>```
>clang version X.X.X (...)
>Target: ...
>Thread model: ...
>InstalledDir: ...
> "/path/to/clang" -cc1 -triple ... -emit-obj -mrelax-all ...
>```
>
>这些输出中包括了 Clang 执行的实际命令和参数，以及编译器版本信息等。

子进程的标准输出流和错误流都重定向到管道上，被父进程读取，父进程通过查找包含`-cc1`的行，即为所需要的**实际**执行的命令行参数，最后使用`quotewords`函数将命令行分解为单词并返回。

随后进过一系列的命令行参数处理之后，调用`Analyze`函数。

```perl
Analyze($Clang, \@CmdArgs, \@AnalyzeArgs, $FileLang, $Output, $Verbose, $HtmlDir, $file);

sub Analyze {
  my ($Clang, $OriginalArgs, $AnalyzeArgs, $Lang, $Output, $Verbose, $HtmlDir,
      $file) = @_;

  my @Args = @$OriginalArgs;
  my $Cmd;
  my @CmdArgs;
  my @CmdArgsSansAnalyses;

  if ($Lang =~ /header/) {  # not used
    exit 0 if (!defined ($Output));
    $Cmd = 'cp';
    push @CmdArgs, $file;
    # Remove the PCH extension.
    $Output =~ s/[.]gch$//;
    push @CmdArgs, $Output;
    @CmdArgsSansAnalyses = @CmdArgs;
  }
  else {
    $Cmd = $Clang;

    # Create arguments for doing regular parsing.
    my $SyntaxArgs = GetCCArgs("-fsyntax-only", \@Args);
    @CmdArgsSansAnalyses = @$SyntaxArgs;

    # Create arguments for doing static analysis.
    if (defined $ResultFile) {
      push @Args, '-o', $ResultFile;
    }
    elsif (defined $HtmlDir) {
      push @Args, '-o', $HtmlDir;
    }
    if ($Verbose) {
      push @Args, "-Xclang", "-analyzer-display-progress";
    }

    foreach my $arg (@$AnalyzeArgs) {
      push @Args, "-Xclang", $arg;
    }

    # Display Ubiviz graph?
    if (defined $ENV{'CCC_UBI'}) {
      push @Args, "-Xclang", "-analyzer-viz-egraph-ubigraph";
    }

    my $AnalysisArgs = GetCCArgs("--analyze", \@Args);
    @CmdArgs = @$AnalysisArgs;
  }

  my @PrintArgs;
  my $dir;

  if ($Verbose) {
    $dir = getcwd();
    print STDERR "\n[LOCATION]: $dir\n";
    push @PrintArgs,"'$Cmd'";
    foreach my $arg (@CmdArgs) {
        push @PrintArgs,"\'$arg\'";
    }
  }
  if ($Verbose == 1) {
    # We MUST print to stderr.  Some clients use the stdout output of
    # gcc for various purposes.
    print STDERR join(' ', @PrintArgs);
    print STDERR "\n";
  }
  elsif ($Verbose == 2) {
    print STDERR "#SHELL (cd '$dir' && @PrintArgs)\n";
  }

  # Capture the STDERR of clang and send it to a temporary file.
  # Capture the STDOUT of clang and reroute it to ccc-analyzer's STDERR.
  # We save the output file in the 'crashes' directory if clang encounters
  # any problems with the file.
  pipe (FROM_CHILD, TO_PARENT);
  my $pid = fork();
  if ($pid == 0) {
    close FROM_CHILD;
    open(STDOUT,">&", \*TO_PARENT);
    open(STDERR,">&", \*TO_PARENT);
    exec $Cmd, @CmdArgs;
  }

  close TO_PARENT;
  my ($ofh, $ofile) = tempfile("clang_output_XXXXXX", DIR => $HtmlDir);
  my $fss_file = abs_path($file);
  my @dentry = split(/\//, $fss_file);
  my $start = ($#dentry >= 3) ? $#dentry - 3 : 0;
  $fss_file  = "${HtmlDir}/";
  for my $i ($start .. $#dentry)
  {
      if ($i == $#dentry) {
	  system "mkdir -p ${fss_file}";
	  print "[XXXX] ${fss_file}\n";
      }
      $fss_file = "${fss_file}/${dentry[$i]}";
  }
  open(my $fss_fh, '>', "${fss_file}.fss")  or die "Cannot open $fss_file\n";

  my $is_path = 0;
  while (<FROM_CHILD>) {
    print $ofh $_;
    print STDERR $_;

    if (/^@@<</) {
      $is_path = 1;
    }
    
    if ($is_path) {
      print $fss_fh $_
    }

    if (/^@@>>/) {
      $is_path = 0;
    }	
  }
  close $ofh;
  close $fss_fh;

  waitpid($pid,0);
  close(FROM_CHILD);
  my $Result = $?;

  # Did the command die because of a signal?
  if ($ReportFailures) {
    if ($Result & 127 and $Cmd eq $Clang and defined $HtmlDir) {
      ProcessClangFailure($Clang, $Lang, $file, \@CmdArgsSansAnalyses,
                          $HtmlDir, "Crash", $ofile);
    }
    elsif ($Result) {
      if ($IncludeParserRejects && !($file =~/conftest/)) {
        ProcessClangFailure($Clang, $Lang, $file, \@CmdArgsSansAnalyses,
                            $HtmlDir, $ParserRejects, $ofile);
      } else {
        ProcessClangFailure($Clang, $Lang, $file, \@CmdArgsSansAnalyses,
                            $HtmlDir, $OtherError, $ofile);
      }
    }
    else {
      # Check if there were any unhandled attributes.
      if (open(CHILD, $ofile)) {
        my %attributes_not_handled;

        # Don't flag warnings about the following attributes that we
        # know are currently not supported by Clang.
        $attributes_not_handled{"cdecl"} = 1;

        my $ppfile;
        while (<CHILD>) {
          next if (! /warning: '([^\']+)' attribute ignored/);

          # Have we already spotted this unhandled attribute?
          next if (defined $attributes_not_handled{$1});
          $attributes_not_handled{$1} = 1;

          # Get the name of the attribute file.
          my $dir = "$HtmlDir/failures";
          my $afile = "$dir/attribute_ignored_$1.txt";

          # Only create another preprocessed file if the attribute file
          # doesn't exist yet.
          next if (-e $afile);

          # Add this file to the list of files that contained this attribute.
          # Generate a preprocessed file if we haven't already.
          if (!(defined $ppfile)) {
            $ppfile = ProcessClangFailure($Clang, $Lang, $file,
                                          \@CmdArgsSansAnalyses,
                                          $HtmlDir, $AttributeIgnored, $ofile);
          }

          mkpath $dir;
          open(AFILE, ">$afile");
          print AFILE "$ppfile\n";
          close(AFILE);
        }
        close CHILD;
      }
    }
  }

  unlink($ofile);
}
```

`Analyze` 函数接受多个参数，包括 `$Clang`（Clang 编译器的路径）、`$OriginalArgs`（原始的编译选项和参数数组）、`$AnalyzeArgs`（分析相关的选项数组）、`$Lang`（文件的语言类型）、`$Output`（输出文件名）、`$Verbose`（详细程度）、`$HtmlDir`（HTML 输出目录，实际上也用作`clang-log`输出目录，只不过和HTML无关）和 `$file`（要处理的文件名）。

首先，函数检查 `$Lang` 是否包含 "header"，如果是，说明要处理的文件是头文件，而不是源代码文件。如果 `$Output` 未定义，就退出，否则执行文件拷贝操作。
如果不是头文件，函数将构建用于执行 Clang 静态代码分析的命令参数。它使用 `GetCCArgs` 函数获取了语法分析的参数（`$SyntaxArgs`）和静态分析的参数（`@CmdArgs`）。

函数通过 `fork` 创建了一个子进程，然后在子进程中执行 Clang 命令，进行语法分析和静态分析。它使用 `pipe` 创建了两个管道，分别用于捕获 Clang 的标准输出和标准错误。然后，子进程将标准输出和标准错误重定向到这两个管道，并执行 Clang 命令。

父进程中，函数会从管道中读取 Clang 的标准输出和标准错误，并将它们同时输出到标准错误流和一个临时文件（`$ofile`）中。在处理输出的过程中，它还会检测是否遇到特定的标记（`@@<<` 和 `@@>>`），它将标记之间的内容写入一个名为 `one.cs.fss` 的文件中，用于后续的分析。


与 `fssccc-analyzer` 脚本同一目录下的 `fssc++-analyzer` 脚本通过直接调用 `fssccc-analyzer` 脚本来进行语法分析和静态分析。




## 结果分析
TODO

## 改进
TODO

向量数据库？
相关系数矩阵？协方差矩阵？等传统数学建模技术