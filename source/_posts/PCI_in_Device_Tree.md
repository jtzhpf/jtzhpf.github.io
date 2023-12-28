---
title: "PCI in Device Tree"
category: CS&Maths
#id: 57
date: 2023-12-25 09:00:00
tags: 
  - Linux
  - Device Tree
  - PCIE/PCI
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: article  # 展示在时间线列表中
---

假设有如下PCI设备：
```dtb
pci@10180000 {
    compatible = "arm,versatile-pci-hostbridge", "pci";
    reg = <0x10180000 0x1000>;
    interrupts = <8 0>;
};
```

## PCI Host Bridge
### PCI Bus numbering
每个PCI总线段都有唯一的编号，总线编号通过使用包含两个单元的`bus-range`属性在pci节点中公开，第一个单元给出分配给此节点的总线号，第二个单元给出任何从属PCI总线的最大总线号。

示例机器只有一个pci总线，因此两个单元都是0。

<figure class="highlight plaintext"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br></pre></td><td class="code"><pre><code class="hljs plaintext">pci@0x10180000 {<br>    compatible = "arm,versatile-pci-hostbridge", "pci";<br>    reg = &lt;0x10180000 0x1000&gt;;<br>    interrupts = &lt;8 0&gt;;<br>    <b>bus-range = &lt;0 0&gt;;</b><br>};<br></code></pre></td></tr></tbody></table></figure>


### PCI Address Translation
PCI地址空间与CPU地址空间完全分开，因此需要地址转换，以从PCI地址转换为CPU地址。这是通过使用`range`、`#address-cells`和`#size-cells`属性完成的。

<figure class="highlight plaintext"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br><span class="line">12</span><br></pre></td><td class="code"><pre><code class="hljs plaintext">pci@0x10180000 {<br>    compatible = "arm,versatile-pci-hostbridge", "pci";<br>    reg = &lt;0x10180000 0x1000&gt;;<br>    interrupts = &lt;8 0&gt;;<br>    bus-range = &lt;0 0&gt;;<br><br>  <b>  #address-cells = &lt;3&gt;<br>    #size-cells = &lt;2&gt;;<br>    ranges = &lt;<font color="red">0x42000000 0 0x80000000</font> <font color="blue">0x80000000</font> <font color="#808026">0 0x20000000</font><br>              <font color="red">0x02000000 0 0xa0000000</font> <font color="blue">0xa0000000</font> <font color="#808026">0 0x10000000</font><br>              <font color="red">0x01000000 0 0x00000000</font> <font color="blue">0xb0000000</font> <font color="#808026">0 0x01000000</font>&gt;;</b><br>};<br></code></pre></td></tr></tbody></table></figure>
如上所示，<font color="red">子地址</font>（PCI地址）使用3个单元，而PCI范围则编码为2个单元。

第一个问题，为什么我们需要三个32位单元来指定一个PCI地址。这三个单元标记为`phys.hi`、`phys.mid`和`phys.low`。

- `phys.hi cell: npt000ss bbbbbbbb dddddfff rrrrrrrr`
- `phys.mid cell: hhhhhhhh hhhhhhhh hhhhhhhh hhhhhhhh`
- `phys.low cell: llllllll llllllll llllllll llllllll`

PCI地址是64位宽，编码到`phys.mid`和`phys.low`中。然而，真正有趣的是在`phys.high`中的位字段：

- `n`：可重定位区域标志（在这里不起作用）
- `p`：可预取（可缓存）区域标志
- `t`：别名地址标志（在这里不起作用）
- `ss`：空间代码
  - 00：配置空间
  - 01：I/O空间
  - 10：32位内存空间
  - 11：64位内存空间
- `bbbbbbbb`：PCI总线号。PCI可能按层次结构构建。因此，我们可能有定义子总线的PCI/PCI桥。
- `ddddd`：设备号，通常与IDSEL信号连接相关。
- `fff`：功能号。用于多功能PCI设备。
- `rrrrrrrr`：寄存器号；用于配置周期。

对于PCI地址转换，`phys.hi`中的`p`和`ss`字段是重要的。`phys.hi`中的`p`和`ss`的值确定正在访问哪个PCI地址空间。因此，查看我们的`ranges`属性，我们有三个区域：

- 从PCI地址0x80000000开始的512 MByte大小的32位可预取内存区域，将映射到主机CPU上的地址0x80000000。
- 从PCI地址0xa0000000开始的256 MByte大小的32位非预取内存区域，将映射到主机CPU上的地址0xa0000000。
- 从PCI地址0x00000000开始的16 MByte大小的I/O区域，将映射到主机CPU上的地址0xb0000000。

### PCI DMA Address Translation
上述范围定义了CPU如何看待PCI内存，并帮助CPU设置正确的内存窗口并将正确的参数写入各种PCI设备寄存器。这有时被称为出站内存。

地址转换的一个特殊情况涉及PCI主机硬件如何看待系统的核心内存。这发生在PCI主机控制器充当主设备并独立访问系统核心内存时。由于这通常是与CPU视图不同的视图（由于内存线的布线方式），因此可能需要在PCI主机控制器初始化时对其进行编程。这被视为一种DMA，因为PCI总线独立执行直接内存访问，因此这些映射被称为`dma-ranges`。这种类型的内存映射有时被称为入站内存，不是PCI设备树规范的一部分。

在某些情况下，ROM（BIOS）或类似物会在启动时设置这些寄存器，但在其他情况下，PCI控制器完全未初始化，这些转换需要从设备树中设置。然后，PCI主机驱动程序通常会解析`dma-ranges`属性并相应地在主机控制器中设置一些寄存器。

对上面的示例进行扩展：

<figure class="highlight plaintext"><table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br><span class="line">12</span><br><span class="line">13</span><br></pre></td><td class="code"><pre><code class="hljs plaintext">pci@0x10180000 {<br>    compatible = "arm,versatile-pci-hostbridge", "pci";<br>    reg = &lt;0x10180000 0x1000&gt;;<br>    interrupts = &lt;8 0&gt;;<br>    bus-range = &lt;0 0&gt;;<br><br>    #address-cells = &lt;3&gt;<br>    #size-cells = &lt;2&gt;;<br>    ranges = &lt;0x42000000 0 0x80000000 0x80000000 0 0x20000000<br>              0x02000000 0 0xa0000000 0xa0000000 0 0x10000000<br>              0x01000000 0 0x00000000 0xb0000000 0 0x01000000<br>    <b>dma-ranges = &lt;<font color="red">0x02000000 0 0x00000000</font><font color="blue"> 0x80000000</font> <font color="#808026">0 0x20000000</font>&gt;;</b><br>};<br></code></pre></td></tr></tbody></table></figure>

这个`dma-ranges`条目表示从PCI主机控制器的角度来看，在PCI地址0x00000000处的512 MB将出现在主核心内存中的地址0x80000000。如上所示，只是将ss地址类型设置为0x02，表示这是一些32位内存。

## Devicetree Specification
<object data="/PCI_in_Device_Tree/devicetree-specification-v0.4.pdf" type="application/pdf" width="100%" height="750">
  <p>This browser does not support PDFs. Please download the PDF to view it: <a href="https://github.com/devicetree-org/devicetree-specification/releases/download/v0.4/devicetree-specification-v0.4.pdf">Download PDF</a></p>
</object>

[^1]:https://elinux.org/Device_Tree_Usage#PCI_Host_Bridge