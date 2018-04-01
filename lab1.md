<h1>实验报告</h1>

<h2>实验目的</h2>

<p>追踪操作系统的启动过程，归纳关键事件</p>

<h2>实验平台&amp;工具</h2>

<ul>
<li>ubuntu16.04-LTS    </li>
<li>gdb7.11.1</li>
<li>qemu 1:2.5+dfsg-5ubuntu10.24</li>
<li><p>dracut</p>

<h2>实验过程</h2></li>
<li><p>从<a href="www.kernel.org">此处</a>下载最新稳定版本linuxkernel(4.15.14)</p></li>
<li>解压
<code>bash
mkdir linuxkernel <br />
cd linuxkernel 
tar -xf linux-4.15.14.tar.gz 
cd linux-4.15.14
</code></li>
<li>编译配置
<code>Bash <br />
make menuconfig //配置编译选项
</code>
注意，在弹出的配置界面中需要注意设置以下选项：
    1. <code>Kernel hacking</code>-><code>Compile the kernel with debug info</code> 选中
<ol>
<li>（可选）<code>Processor type and features</code>-><code>Build a relocatable kernel</code>-><code>Randomize the address of the kernel image</code>
其中，  为方便，第二个最好选中。若不选，在使用QEMU时，需要加上-append nokaslr
原因：开机时，默认给kernel image分配随机空间。好处是可以更加安全，灵活;坏处是，调试的时候设置的断点位置无法定位image，导致最终在断点处不停止</li>
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

<ul>
<li>制作initramfs镜像</li>
</ul>

<p><code>Bash
dracut kver linux-4.15.14//匹配内核版本
su root
mv /boot/linuxramfs-linux-4.15.14.img ./arch/x86/boot //移动至bzImage同一目录
//备注：当前目录为linux-4.15.14
</code>
注意,boot目录需要root权限，因此此后均切换为root用户操作</p>

<ul>
<li>安装gdb, qemu,均使用最新版本
<code>Bash
sudo apt-get install gdb qemu
</code></li>
<li>开始调试
<code>Bash
qemu-system-x86_64 -kernel arch/x86/boot/bzImage  //内核映像
               -initrd arch/x86/boot/initramfs-linux-4.15.14.img
               -s -S //-s表示-gdb tcp::1234, -S表示在开始时暂停qemu
               -m 2048 //分配内存，防止开机过程错误
</code></li>
</ul>

<p>另打开一个终端：
<code>Bash
gdb
(gdb) file vmlinux //加载符号表
(gdb) target remote : 1234 //连接1234端口，远程控制qemu
(gdb) b start_kernel //start_kernel处设断点
(gdb) c //run
</code>
遇到如下错误：
<code>Remote 'g' packet reply is too long: 00000000000000005046010000000000000000000000000000000000000000000d352828000000005046010000000000303f2082f
fffffff283f2082ffffffff00405182ffffffff0800000000000000a03e2082ffffffffe300000100000000000000000000000000
0000000000000000000000000000000000000000000000a5bb4982ffffffff4600000010000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000000000007f03000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
00000000000000801f0000</code></p>

<p>参考<a href="https://stackoverflow.com/questions/48620622/how-to-solve-qemu-gdb-debug-error-remote-g-packet-reply-is-too-long">此处</a>,以及<a href="http://www.mamicode.com/info-detail-2139962.html">博客</a>得知问题原因为：</p>

<p>Note that other tutorials also add a "-S" parameter so QEMU starts the kernel stopped, however this is ommitted deliberately. The "-S" parameter would allow gdb to set an initial breakpoint anywhere in the kernel before kernel execution begins. Unfortunately, a change made to the gdbserver in QEMU, to support debugging 32- and 16-bit guest code in an x86_64 session breaks the -S functionality. The symptoms are that gdb prints out "Remote ‘g‘ packet reply is too long:", and fails to interact successfully with QEMU. The suggested fix is to run the QEMU until it is in 64-bit code (i.e. after the boot loader has finished and the kernel started) before connecting from gdb (omitting the -S parameter)</p>

<p>参考<a href="http://bbs.85kf.com/a21.html">博客</a>，得到解决方案：
<code>Bash
(gdb) disconnect
(gdb) set arch i386:x86-64:intel
(gdb) target remote : 1234
</code>
然后即可正常运行</p>
