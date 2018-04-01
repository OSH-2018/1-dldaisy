<h1>实验报告</h1>

<h2>实验目的</h2>

<p>系统的启动过程，归纳关键事件</p>

<h2>实验平台&amp;工具</h2>

<ul>
<li>ubuntu16.04-LTS</li>
<li>gdb7.11.1</li>
<li>qemu 1:2.5+dfsg-5ubuntu10.24</li>
</ul>

<h2>实验过程</h2>

<ul>
<li>从<a href="www.kernel.org">此处</a>下载最新稳定版本linuxkernel(4.15.14)</li>
<li>解压
<code>bash
mkdir linuxkernel <br />
cd linuxkernel 
tar -xf linux-4.15.14.tar.gz 
cd linux-4.15.14
</code></li>
<li>编译配置
<code>bash <br />
make menuconfig //配置编译选项
</code>
注意，在弹出的配置界面中需要注意设置以下选项：
    1. <code>Kernel hacking</code>-><code>Compile the kernel with debug info</code> 选中
<ol>
<li>（可选）<code>Processor type and features</code>-><code>Build a relocatable kernel</code>-><code>Randomize the address of the kernel image</code>
其中，  为方便，第二个最好选中。若不选，在使用QEMU时，需要加上-append nokaslr
原因：开机时，默认给kernel image分配随机空间。好处是可以更加安全，灵活;坏处是，调试的时候设置的断点位置无法定位image，导致最终
断点处不停止</li>
</ol></li>
</ul>

<p><code>bash
make -j4 //4核加速编译
     -bzImage //编译内核
     -vmlinux//符号表，供QEMU使用
</code>
以上选项可以避免编译无用模块，加速编译过程
此过程大约需要15分钟</p>

<p>编译完成后，bzImage文件出现在./arch/x86/boot/目录下，vmlinux出现在当前目录下，随时可供后续调用</p>
