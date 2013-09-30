---
author: tom165
comments: true
date: 2013-08-13 00:25:02+00:00
layout: post
slug: ios_hacker_handbook_chap10
title: iOS黑客手册翻译-第10章越狱
wordpress_id: 145
categories:
- 编程技术
---

[![p8165990](http://tom165.files.wordpress.com/2013/08/p8165990.jpg?w=238)](http://tom165.files.wordpress.com/2013/08/p8165990.jpg)

第10章 越狱 （Jailbreaking）

如果你看完书中的所有例子，你很可能已经做完你的实验和在已经越狱的iPhone上的研究。因为和许多人一样，几乎所有的iPhone安全研究都在已经越狱的设备上实施。然而，对包括安全社区和iPhone安全研究者在内的大部分人来说，越狱的内部工作原理是完全不知道的。许多人把越狱当黑匣子看待；当他们在越狱工具选项里点击jailbreak按钮，认为工作起来像魔术一样。这是由于对于他们所进行的越狱操作并不需要知道越狱的内部工作原理，如用户层漏洞。

但如果想知道越狱过程的内部工作原理，这章将是你许多问题的答案。

对不同越狱类型的简短介绍后，我们用红雪(redsn0w)越狱做例子，向你一步步介绍设备所发生的越狱过程。这章也向你介绍内核应用补丁的内部工作原理，你将知道对应不同选项哪些补丁是实际需要的。<!-- more -->

为何要越狱？

人们对他们的ios设备越狱有许多原因，有的是为了开发软件需要一个开放的平台，有的喜欢对他们的设备的全部控制的感觉，有的需要越狱去安装解锁运营商的软件（如ultrasn0w），有的越狱为了安装私人的iPhone应用。

另一方面，安全研究者进行自己的iOS设备的越狱通常有其他原因。事实上由于正常的iPhone被紧紧的锁定，不允许运行未签名的代码，对于评估系统安全，挖掘安全漏洞，这是一个大障碍。

甚至即使拥有一个苹果的iOS开发者账号，由于沙盒(sandbox)和其他限制，代码在iPhone上运行也是有限制的。例如，进程不允许执行其他进程和复制。另外。沙盒制止了研究者修改其他应用程序的文件，也不可能对MobileSafari浏览器挂接调试器进行调试。

虽然可以在一个正常的iPhone应用里检测到运行进程的名称，但用户无法制止可疑进程的运行或分析他们在做什么。回想下苹果公司把在每台iPhone上存储GPS轨迹文件归结于软件bug的事情，如果没有越狱，这个“定位门”问题永远不会被发现。

最重要的，如果没有公开越狱的工具，这本书的大部分研究将没有可能。你将惊奇的发现大部分iPhone的安全研究人员把整个越狱工作留给 iPhone Dev Team或 Chronic Dev Team去做，而他们只是越狱工具的使用者。但每个新一代的iOS的硬件和软件版本出来，越狱变得越来越困难。所以有更多的安全社区的人来帮助越狱团队是很重要的。我们希望这章的其它部分能吸引你来参加未来越狱的发展。

越狱类型

虽然多年来人们能对他们的各种iOS版本的iPhone进行越狱，这些越狱都有不同的特性。主要是因为越狱的质量在很大部分依赖于发现的安全漏洞，这些漏洞被用来解除设备的强制约束。一旦一个越狱的漏洞被知晓，苹果公司自然会尽快在下一版本的iOS中进行修补。所以几乎每个新版本的iOS都需要一套新的漏洞来进行越狱。 有时漏洞是驻留在硬件层面的，苹果公司不能进行简单的软件升级来修补它，这需要一套新的硬件，需要发布下一版本的iPhone或iPad，将花费苹果公司更长的时间来修补。

越狱的保持（Jailbreak Persistence）

依赖越狱使用的漏洞，越狱的效果可能长期的，也可能在设备关机再开启后消失。为了描述两种不同的越狱，越狱社区把这两种方式叫做 完美越狱（tethered jailbreak）和非完美越狱（untethered jailbreak）。

非完美越狱（Tethered Jailbreaks）

非完美越狱是设备重启后就消失的越狱。越狱后的设备每次重启后需要某种方式的重新越狱。通常意味着每次关机再重新开机时需要重新连接到电脑上。由于这个过程中需要USB线缆连接，这就是tethered的意思。这个词tethered也用于不需要USB连接，但需要访问特定网站或执行特定应用程序的重新越狱，

如果漏洞是某些提权代码，一个非完美越狱能只由单个漏洞组成。一个例子是limera1n的bootrom漏洞，被目前的大部分iOS4和iOS5越狱使用。另一个例子是iOS的USB内核驱动程序的漏洞。然而，目前没有类似的公开漏洞。

如果没有类似的漏洞可用，进入设备的初始入口可以通过一个应用程序的漏洞来获得很少的权限，如MobileSafari浏览器。然而，单独的这样一个漏洞不能认为是越狱，因为没有附加的内核漏洞，不能禁止所有的安全特性。 所以一个非完美越狱由一个提权代码漏洞组成，或一个非提权代码漏洞结合其他权限提升漏洞。

完美越狱（Untethered Jailbreaks）

完美越狱是利用一个设备重启后不会消失的持久漏洞来完成的，完美（untethered）因为设备每次重新启动后不需要重新越狱，所以它是更好的越狱方式。

由于完美越狱需要启动环节的非常特殊地方的漏洞，自然更难完成。过去由于在设备硬件里发现了非常强有力的漏洞，允许在设备的启动环节的早期进行破解，使完美越狱可能实现。 但这些漏洞现在已经消失了，而同样品质的漏洞没有出现。

由于以上原因，完美越狱通常由某种非完美越狱结合附加的允许在设备上保持的漏洞组成。利用开始的非完美越狱来在设备的根目录文件系统上安装附加漏洞。由于首先专有未签名代码必须执行，其次权限要提升以便能对内核打补丁，所以至少需要2个附加的漏洞。

接下来的部分将介绍越狱的全貌，当你读完，将会很清晰的了解设备越狱的完整详细的过程。

漏洞类型(Exploit Type)

漏洞的存在位置影响你对设备的存取级别。一些允许低级别的硬件存取。另一些受限于沙盒内的许可权限。
Bootrom级别
从越狱者的角度看，Bootrom级别的漏洞是最有力的。bootrom在iPhone的硬件内部，它的漏洞不能通过软件更新推送来修复。相反，只能在下一代的硬件版本里修复。在存在limera1n漏洞的情况下，苹果没有发布iPad1或iPhone4的新产品，直到A5处理器的设备，iPad2和iPhone4S发布前，这个漏洞长期存在并为人所知。

Bootrom级别的漏洞不能修复，并且允许对整个启动环节的每个部分进行替换或打补丁（包括内核的启动参数），是最有力的漏洞。由于漏洞在启动环节发生的很早，而且漏洞Payload拥有对硬件的全部读取权限。 如它可以利用AES硬件引擎的GID密码来解密IMG3文件，而IMG3文件允许解密新的iOS更新。

iBoot级别

当iBoot里的漏洞达到能提供的特性时，几乎和bootrom里的漏洞一样有力。这些漏洞效果下降是由于iBoot没有固化入硬件，能通过简单的软件升级来修复。

除了这点，iBoot漏洞在启动环节任然很早，能提供给内核启动参数，对内核打补丁，或对硬件直接进行GID密码的AES操作。

Userland级别

用户层面级别的越狱是完全基于用户层面进程的漏洞的，像JBME3([http://jailbreakme.com](http://jailbreakme.com/))。这些进程如果是系统进程，就拥有超级用户root权限，如果是用户应用程序，就拥有稍低级别的如mobile用户的权限。不管哪种情况，越狱设备至少需要2个漏洞。第一个漏洞用来完成专有代码执行，第二个漏洞用来使内核的安全措施失效，进行权限提升。

在以前的iOS版本里，只要破解的进程以root超级用户权限运行，代码签名功能就会失效。现在，内核内存崩溃或执行内核代码需要禁止强制代码签名。

和bootrom、iBoot级别漏洞相比，用户层面的漏洞更弱一些。因为即使内核代码执行已经可能了，如GID密码的AES引擎的硬件特性依然不能读取。所以苹果公司对用户层面的漏洞更容易修补；并由于远程用户层面的漏洞能用于iPhone恶意软件的注入，所以苹果公司对这些漏洞经常很快进行修补。

理解越狱过程(Understanding the Jailbreaking Process)

这个章节观察了红雪(redsn0w)越狱工具的内部工作原理。红雪是由 iPhone Dev Team团队开发的，可以在他们的网站([http://blog.iphone-dev.org/](http://blog.iphone-dev.org/))上下载。由于它支持大部分的iOS版本，容易使用，被视为最稳定的越狱，而且同时支持Windows和OS X操作系统，是目前对A5以前的设备进行越狱的最流行的工具。

使用红雪，越狱只是点击几个按钮，把你的iPhone设置为设备固件升级(DFU)状态。新手使用它来越狱iPhone都是很容易的。图10.1显示了红雪的启动屏幕截屏画面：











[![p8165970](http://tom165.files.wordpress.com/2013/08/p8165970.jpg?w=232)](http://tom165.files.wordpress.com/2013/08/p8165970.jpg)













当你点击Jailbreak越狱按钮后，红雪指导设置你的iPhone为DFU模式，根据你连接上的设备，提供你可选择的不同的越狱特性。你简单选择你的选项（如多任务手势），点击Next继续按钮，等待红雪完成它的工作。

从用户角度看这是非常简单的过程，但是这这些操作后面发生了很多事情，并且除了越狱社区的少数人，没人真的知道发生了什么。当你读完下面章节后，你将成为了解红雪内部工作的少数人之一。

经作者的允许，下面章节的所有信息都来自一个反编译版本的红雪越狱。由于A5处理器的iPad2或iPhone 4S设备没有一个公开知晓的bootrom漏洞，这些设备的所有越狱必须是用户层面级别的。 这表示bootrom破解和启动ramdisk的头两个步骤必须由MobileSafari漏洞和一个内核漏洞替代，其余的越狱过程工作是一样的。

破解Bootrom(Exploiting the Bootrom)

红雪的越狱开始时利用the limera1n 的DFU bootrom漏洞来使在高权限级别执行代码成为可能。这个漏洞是一个前A5处理器设备的bootrom中USB DFU堆栈中的基于堆的缓冲区溢出漏洞。我们这里不讨论漏洞的细节。如果感兴趣，你可以在诸如THEiPHONEWiKi（[http://theiphonewiki.com/wiki/index.php?title=Limera1n_Exploit](http://theiphonewiki.com/wiki/index.php?title=Limera1n_Exploit)）的一些地方找到这个漏洞的说明和源代码。

对越狱来说，你唯一需要知道的是这个漏洞用来给bootrom里的签名验证代码打补丁修改，允许你启动专门的ramdisk，给Low-Level-Bootloader (LLB)、 iBoot和内核的版本打补丁。 Chronic Dev Team团队已在GitHub([https://github.com/Chronic-Dev/syringe](https://github.com/Chronic-Dev/syringe))上释放了同样执行效果的源代码。由于红雪没有公开源代码，如果你想重头编写自己的越狱工具，这些代码是一个很好的起点。

启动Ramdisk(Booting the Ramdisk)

红雪利用 limera1n 漏洞启动一个打补丁修改过的内核和一个预先定制的ramdisk。内核被打了一些越狱补丁以便允许执行未签名的代码。然而，它并没包括在已完美越狱的系统中发现所有内核补丁。依据用户在越狱时设置的开关选项，ramdisk在每次运行时都在根目录创建不同的文件，形成定制的ramdisk。这些文件的存在在晚些时候会被ramdisk上的越狱执行过程检测到，以此决定红雪的哪种特性被激活。例如noUntetherHacks文件的存在会使红雪跳过完美越狱漏洞的安装。

ramdisk启动时，内核会运行 ramdisk的/sbin/launchd的二进制文件，它包括一个初始化越狱的小型stub程序。这个程序首先在系统里装载root根文件系统和数据分区，为修改需要，以上装载好的文件系统和数据分区是可读写。最终，一个越狱的执行程序会接管并执行所有接下来的步骤。

越狱文件系统(Jailbreaking the Filesystem)

iPhone的文件系统缺省分成2个分区，第一个分区是root根文件系统，存放iOS的操作系统文件和如MobileMail，MobileSafari的标准应用程序集。早期的iOS版本里，root根文件系统的大小接近分区的容量大小，分区没有多少剩余空间。虽然root文件系统是假设为不可修改的，分区也缺省装载为只读的，但如今root根文件系统接近1GB大小，分区有200MB左右空间剩余。设备的其余存储空间分配给第二个分区，即数据分区，它装载在/private/var目录下，可以读写。在根目录的 /etc/fstab文件里有以下配置：

/dev/disk0s1 / hfs ro 0 1
/dev/disk0s2 /private/var hfs rw,nosuid,nodev 0 2

你可以看到，数据分区的装载配置文件有 nodev 和nosuid标志。为了防止文件系统层面的攻击，nodev标志是确保设备节点在可写的数据分区里的存在将会被忽略。nosuid标志是告知内核在数据分区里执行时忽略suid位。suid位表示执行时需要以超级用户root身份执行，或与当前用户不同的一个用户来执行。这些标志都是iOS内部用来防止权限提升漏洞的防御措施。

不管是bootrom级别，还是用户层面级别的漏洞，这个缺省的配置是所有越狱面临的难题。因为他们通常需要对root根文件系统进行修改，使程序在设备重启后还能存在，或能增加服务和守护进程。所以每次越狱获得超级用户root的权限后的第一个动作就是重新装载root根文件系统，使之可读可写。为了使设备重启后修改也生效，下一个步骤就是用下面类似的配置替换系统的/etc/fstab文件：

/dev/disk0s1 / hfs rw 0 1
/dev/disk0s2 /private/var hfs rw 0 2

这个新的文件系统的配置文件将装载root根目录文件为可读可写，并从第二个分区的装载配置中移去nosuid和nodev标志。

安装完美越狱破解(Installing the Untethering Exploit)

每次当一个新版本的iOS出现，原先的已知漏洞就被修复了。 所以在一个有限的时间窗口里，红雪在老设备上能越狱新的固件，但不能安装完美越狱漏洞。

一旦得到了一个新的完美越狱漏洞，红雪的作者就会修改程序来安装新的漏洞。 由于每个漏洞都是不同的，所以都需要不同的安装步骤。

虽然实际的完美越狱漏洞安装是不同的，但都是在root根文件系统里重命名和移动一些文件，并拷贝一些文件到它上面。当你反编译当前的红雪软件版本，你会发现它从4.2.1到5.0.1的大部分iOS版本上都支持安装完美越狱漏洞，发现每个完美越狱漏洞需要那些文件。

安装 AFC2服务(Installing the AFC2 Service)

苹果文件连接 (AFC)是一个运行在每台iPhone上的文件传送服务，它允许你通过USB连线存取iPhone的/var/mobile/Media的目录里的文件。AFC服务由lockdownd守护进程提供，被命名为com.apple.afc 。但lockdownd守护进程只提供存取服务，实际由afcd守护进程实现。在MAC上通过MobileDevice.framework，在Windows PC上通过 iTunesMobileDevice.dll来使用。

afcd还提供lockdownd的第二个服务，它注册为com.apple.crashreportcopymobile。它用来从设备向电脑复制CrashReporter报告，并只限制于/var/mobile/Library/Logs/CrashReporter目录和子目录的读写存取 。

由于这些服务运行于mobile用户的许可权限下，被锁定于特定目录下，他们对于越狱的作用是受限的。所以红雪和其他越狱工具向lockdownd注册了一个额外的服务，叫做com.apple.afc2。这个服务利用afcd守护进程提供对整个文件系统的root超级用户权限的读写存取，这是一个大部分用户都不知道的相当危险的越狱特性。这意味着一台没有密码保护或在未锁定状态的越狱过的iPhone，只要连接上一个USB充电站或其他人的电脑，就给了另一方在没有用户交互下的整个文件系统的读取存取权限。他们可以窃取你的所有数据或安装系统后门。

com.apple.afc2服务通过更改在/System/Library/Lockdown/Services.plist文件里的lockdownd配置来安装。这是一个正常的.plist 文件，可以通过 .plist文件的标准工具或API来修改。红雪通过在文件里增加以下内容来安装新的服务：

<key>com.apple.afc2</key>

<dict>

<key>AllowUnactivatedService</key>
<true />
<key>Label</key>
<string>com.apple.afc2</string>
<key>ProgramArguments</key>
<array>
<string>/usr/libexec/afcd</string>
<string>–lockdown</string>
<string>-d</string>
<string>/</string>
</array>
</dict>

由于文件系统越狱和新的AFC2服务可以通过简单配置文件的更改来完成，并不需要执行未签名的二进制代码。所以他们在重启后就可以工作，即使一个设备还没有发现完美越狱漏洞。

安装基本程序库(Installing Base Utilities)

苹果没有在iPhone上提供UNIX外壳程序，所以它的根目录下的/bin和/usr/bin目录基本是空的，没有你所期待的可执行的二进制文件。实际上，最新的iOS5.0.1只在这两个目录下预先安装提供了以下5个可执行的二进制文件：

/bin/launchctl
/usr/bin/awd_ice3
/usr/bin/DumpBasebandCrash
/usr/bin/powerlog
/usr/bin/simulatecrash

因为这样，类似红雪的越狱工具通常在这两个目录下安装一套基本程序库来实现基本功能，使越狱的文件安装更简单。下面的工具清单是从红雪的ramdisk里的越狱二进制文件里展开的，显示了红雪安装的基本程序库清单。这些工具也用于越狱二进制文件自身，例如压缩tar文档或更改.plist文件内容。

/bin/mv
/bin/cp
/bin/tar
/bin/gzip
/bin/gunzip
/usr/sbin/nvram
/usr/bin/codesign_allocate
/usr/bin/ldid
/usr/bin/plutil

除了这些文件，一些附加库和文件也被安装用于越狱过程而不是用于UNIX外壳用户。所以我们没有把他们列出来。一个有趣的事是当前的iOS固件已经有一个/usr/sbin/nvram二进制文件，但被红雪重新覆盖。

应用程序隐藏(Application Stashing)

当从苹果的应用程序商店安装应用时，它们被安装在/var/mobile/Applications目录里，该目录驻留在iPhone的大容量的数据分区里。可以安装的应用程序的数量依赖于数据分区的空闲空间。空闲空间通常有上GB 容量，所以对安装应用程序数量来讲不是真正的限制.

Cydia是越狱者制做的相当于苹果AppStore的应用程序商店，通过Cydia下载的越狱应用的安装地点是不同的。像Cydia自身和其他内置的二进制文件安装在root根目录的/Application目录下。前面提到，root根文件系统的容量取决于固件的版本、容量和设备类型。通常在1GB和1.5GB之间，有大约200MB的空闲，这点容量对安装应用程序来说是不多的。

另外，墙纸和铃声也存放在根目录的 /Library/Wallpaper 和 /Library/Ringtones目录下，所以从Cydia安装的每个墙纸和铃声也也会占用掉有限的应用程序空间。

为解决这个问题，各种越狱都实现了应用程序隐藏技术。它是在iPhone的数据分区创建了/var/stash的新目录，并把一些正常情况下位于root根文件系统下的应用程序移到/var/stash目录下，原来的目录替换为链接到新目录的符号链接。

以下清单是通过程序隐藏转移到/var/stash下的目录：

/Applications
/Library/Ringtones
/Library/Wallpaper
/usr/include
/usr/lib/pam
/usr/libexec
/usr/share

然而，不是所有的越狱工具和所有版本的越狱工具都进行应用程序隐藏。因为这样，Cydia创建时在第一次调用时会进行检测，这是Cydia里的重新组织文件系统（“Reorganizing Filesystem”）的耗时较长的步骤。

安装程序包 (Bundle Installation)

越狱安装过程的下一步是安装程序包。根据使用的工具，它可以是一个高级用户自己定制的程序包，或越狱后缺省附带的Cydia程序包。例如，红雪接收的程序包是简单的tar文档，可以选择是否由gzip压缩。 它们可以通过前面安装的基本程序库解压，所以越狱不需要文档解压代码。

程序库的安装是通过对存放在ramdisk上的程序包逐个解压缩来进行的，解压缩过程中，解压缩程序(tar)保留了UNIX的许可权限，以便允许你把suid的超级用户位的设置一起打包设置。Cydia需要这个设置，因为没有root超级用户权限，它不能安装新的应用程序。对于苹果的欺骗是很有趣的，值得说明下，因为图形界面程序在他们的主目录上不能有suid位设置，所以Cydia通过运行一个叫做MobileCydia的shell脚本，来给主目录设置suid位。

然而，应用程序包解压缩到/Application目录后，安装过程并未完成，所有的已安装的应用程序需要注册到一个特殊的系统层面的安装缓存里，它存储在/var/mobile/Library/Caches/com.apple.mobile.installation.plist文件里。这个文件时一个正常的.plist文件，格式如下：

<plist version=”1.0″>

<dict>

<key>LastDevDirStat</key>

<integer>…</integer>

<key>Metadata</key>

<dict>…</dict>

<key>System</key>

<dict>

<key>com.apple.xxx</key>

<dict>…</dict>

</dict>

<key>User</key>

<dict>

<key>someuserapp</key>

<dict>…</dict>

</dict>

</dict>

</plist>

缓存包括一个时间戳，一些元数据，冠以所有系统和用户程序的信息。系统程序都在/Application目录里，从苹果的AppStore下载的用户程序放在/var/mobile/Applications里。 所有的程序包都要注册系统缓存条目。红雪会读取应用程序的Info.plist 文件，利用里面的信息来创建新的缓冲条目。首先，读取CFBundleIdentifier值并用来作为缓冲的一个新值，然后Info.plist文件里的系统值词典里增加了一个应用程序类型（ ApplicationType）的新的值。

安装后过程(Post-Installation Process)

全部安装完成后，红雪调用sync()系统调用，确保全部东西写入磁盘。然后，root根文件系统被重新装载为只读状态，确保所有可写缓冲区同步到磁盘上。装载到 /var目录的数据分区被卸载。在装载操作失败的情况下，重复这个过程直到它成功或直到超出重试次数。

越狱最后调用reboot()系统调用来重启系统，结束越狱过程。如果是非完美越狱，设备重启为未越狱状态，除非安装的程序包修改了重启需要的文件。红雪需要非完美重启来使设备进入越狱状态。

如果设备已经完美越狱，设备重启会进入越狱状态。重启过程中已安装的完美漏洞会修改一些应用，利用其它内核漏洞来在内核里执行代码。在接下来的章节你将学习这些内核payload。

执行内核Payloads并修改内核(Executing Kernel Payloads and Patches)

这章节将对前面关于内核漏洞章节里没有讨论的内核级别的Payload进行讨论。这是因为执行内核payload是越狱过程中用来真正打破监狱封锁的，是越狱最重要的部分。因此我们认为最好放在这一章节专门描述。

虽然每个内核漏洞和每个payload是不同的，但能把越狱所用的内核级的payload分为为4个部分：

（1）修复内核状态Kernel State Reparation

（2）权限提升Privilege Escalation

（3）修改内核Kernel patching
（4）清理并返回Clean return
接下来的章节对他们进行详细描述

修复内核状态(Kernel State Reparation)

虽然存在不同类型的内核漏洞，但在内核里执行特定代码通常是一些内核级的函数指针被覆盖的结果。按照漏洞的类型，被覆盖的函数指针可以是内核内存里唯一崩溃的地方，但通常不是这样。像堆载和堆缓冲区溢出这样的类型的漏洞通常引起大面积的崩溃区域。尤其是由于攻击堆的元数据结构引起的堆缓冲区溢出，在执行漏洞代码后内核堆可能变成不稳定的状态，结果就是或迟或早的内核错误。

所以在每个内核漏洞执行后，修正它引起的内存问题或崩溃的状态是很重要的。刚开始把覆盖的函数指针恢复成崩溃前的值，然而，通常这还不够。对于堆漏洞，由于被攻击堆的元数据需要修复，内核修复是可能是一个很复杂的任务。依据内核堆的信息使用的方法，它可能需要扫描内核内存，查找需要释放的被泄露的堆内存块，来确保内核运行不越界。

对于堆栈数据崩溃，内核堆栈是否需要修补取决于漏洞的情况。 一个系统调用可能有处理例外的内核线程，它的堆栈缓冲区溢出不需要修补，因为不会引起内核错误。

权限提升(Privilege Escalation)

由于iPhone上的所有应用程序的都以权限较少的用户（如mobile,_wireless,_mdsnresponder,_securityd）角色运行，一个内核漏洞的payload执行后，通常要把应用程序的运行进程的权限提升到root超级用户权限。 如果少了这一步骤，有些操作是不可能完成的的，像重新装载root根文件系统到可写状态，或修改属于root超级用户的文件。所有这些在越狱的初始安装时都时需要的。只用于完美越狱的重启后执行的内核漏洞不需要这一步骤，因为通常已经已root超级用户角色运行。

在内核内部，提升当前运行进程的权限是很容易的。只需要修改进程的proc_t结构的信用值。这个结构在XNU源代码的/bsd/sys/proc_internal.h文件的proc结构里定义。依据不同的内核漏洞payload开始运行方法的不同，取得当前进程的proc_t结构的指针的方法是不同的。在以前公开的许多iOS内核漏洞里，用于覆盖系统调用表里的系统调用程序的地址的办法是不同的。内核漏洞的payload由调用被覆盖的系统调用来触发。在这种情况下，它获得的proc_t结构是微不足道的，因为它是系统调用程序的第一参数。

得到proc_t结构的地址的更通用的方法是调用current_proc()内核函数，它能取回这个结构的的地址。这个函数是一个内核输出的符号，很容易找到。由于刚开始的内核漏洞能检测使用的内核版本，并且内核里没有地址随机化，所以它可以把函数的地址硬编码到内核漏洞里。

取到proc_t结构的地址的第三个方法是使用通过sysctl接口泄露的内核地址信息。这项技术是由noir在破解OpenBSD内核的过程中首先发表的(详见[www.phrack.org/issues.html?issue=60&id=06](http://www.phrack.org/issues.html?issue=60&id=06))，并被nemo用于XNU内核(详见[www.phrack.org/issues.html?issue=64&id=11](http://www.phrack.org/issues.html?issue=64&id=11))。这个泄露的信息允许用户态进程通过一个简单的sysctl系统调用取到proc结构的内核地址。

在取到进程proc_t结构的地址后，结构里的p_ucred成员用来修改连接的ucred结构。 这个元素可以通过proc_ucred()函数读取，或直接读取。下面的反汇编代码表明在当前iOS版本里结构里p_ucred域的偏移量是0×84.

_proc_ucred:

LDR.W R0, [R0,#0x84]
BX LR

在/bsd/sys/ucred.h文件里有ucred结构的定义。结构里还包含拥有这个结构的进程的用户ID和组ID。

struct ucred {
TAILQ_ENTRY(ucred) cr_link; /* never modify this without
KAUTH_CRED_HASH_LOCK */
u_long cr_ref; /* reference count */
struct posix_cred {
/*
* The credential hash depends on everything from this point on
* (see kauth_cred_get_hashkey)
*/
uid_t cr_uid; /* effective user id */
uid_t cr_ruid; /* real user id */
uid_t cr_svuid; /* saved user id */
short cr_ngroups; /* number of groups in advisory list */
gid_t cr_groups[NGROUPS]; /* advisory group list */
gid_t cr_rgid; /* real group id */
gid_t cr_svgid; /* saved group id */
uid_t cr_gmuid; /* UID for group membership purposes */
int cr_flags; /* flags on credential */
} cr_posix;
struct label *cr_label; /* MAC label */
/*
* NOTE: If anything else (besides the flags)
* added after the label, you must change
* kauth_cred_find().
*/
struct au_session cr_audit; /* user auditing data */

};

为了提升拥有这个结构的进程的权限，结构偏移量0x0c处的cr_uid域被设置为0。如你预料的，偏移量是0x0c，而不是0×08，是因为一个TAILQ_ENTRY条目是8byte的。当然，其他元素也能被修改。然而一旦uid值等于0，用户态的进程就能利用系统调用更改它的许可权限。

修改内核(Kernel Patching)

内核级payload最重要的部分是对内核代码和数据进行内核级别的修改，使安全功能失效，让未签名的代码可以执行，设备也就被越狱了。这几年，不同的越狱团队开发了他们自己的修改补丁，所以大部分越狱都有不同的内核修改补丁，有不同的功能特性。最流行的内核修改补丁是由comex开发的，可以在他的github里的datautils0程序库得到([https://github.com/comex/datautils0](https://github.com/comex/datautils0))。它不仅被comex用于comex自己的越狱网站（[http://jailbreakme.com](http://jailbreakme.com/)），也被许多其他iOS内核内部研究的人做参考。然而，这些GitHub程序库的特定的补丁，不可能适用将来的内核版本；因为comex已经加入苹果公司，很可能在从事和以前相反的工作，防止将来的iPhone被越狱。

然而，接下来的章节向你介绍这些补丁和它们背后的原理，你可以用它们来开发你自己的内核修改补丁，运用于未来的iOS版本。

security.mac.proc_enforce变量

security.mac.proc_enforce是个系统调用变量，它控制着是否在进程操作中使能MAC策略。变量禁止时，各种进程策略检查和限制就被关掉了。例如，对fork(), setpriority(), kill() 和 wait() 系统调用的限制。 除此以外，这个变量还控制着代码签名blob的数字签名是否合法。变量禁止时，代码签名blob的数字签名是非法的二进制代码也可以被执行。

在iOS的4.3以前的版本里，这个变量对以root超级用户方式运行的完美越狱漏洞来说是个捷径。漏洞可以通过sysctl()系统调用禁止这个变量，来允许他们在内核漏洞里执行二进制代码。像现在需求的用对象返回编程来编写整个内核漏洞没有必要了。为了防止这个攻击，苹果公司在iOS4.3里已经把security.mac.proc_enforce系统调用变量改为只读了。

对于内核payload，因为能分配0值给这个变量，禁用该变量不是一个大问题。唯一需要做的工作是决定变量在内存中的地址。一个可能的解决办法是扫描内核里的_sysctl_set段，这里定义了sysctl变量和变量的地址。由于变量在内核数据段内，它是一个永久静态地址。

内核的cs_enforcement_disable变量

/osfmk/vm/vm_fault.c文件的页错误处理程序的源代码里有一个cs_enforcement_disable变量，它控制着页错误处理程序的代码签名是否起作用。该变量在iOS内核里缺省的初始化为0，使之起作用。如果反之，赋予非零值i，将不起作用。

查看代码，你会发现这个变量只在vm_fault_enter()函数里用了2次。下面的代码是使用这个变量的第一次，对于发生了什么，代码注解得很详细：

/* If the map is switched, and is switch-protected, we must protect

* some pages from being write-faulted: immutable pages because by
* definition they may not be written, and executable pages because
* that would provide a way to inject unsigned code.
* If the page is immutable, we can simply return. However, we can’t
* immediately determine whether a page is executable anywhere. But,
* we can disconnect it everywhere and remove the executable
* protection from the current map.
* We do that below right before we do the
* PMAP_ENTER.
*/
if(!cs_enforcement_disable && map_is_switched &&
map_is_switch_protected && page_immutable(m, prot) &&
(prot & VM_PROT_WRITE))
{
return KERN_CODESIGN_ERROR;
}

你在代码里可以看到，如果设置了cs_enforcement_disable标志，其他条件检查就会被忽略。代码接下来的检查一页内存是否未签名但想执行的条件也会成立：

if (m->cs_tainted ||
(( !cs_enforcement_disable && !cs_bypass ) &&
(/* The page is unsigned and wants to be executable */
(!m->cs_validated && (prot & VM_PROT_EXECUTE)) ||
/* … */
(page_immutable(m, prot) && ((prot & VM_PROT_WRITE) || m->wpmapped))
))
)
{

在这两种情况下，设置cs_enforcement_disable变量就能使保护失效。考虑到变量初始化为0并没有被改写，我们很庆幸编译器没有对它进行优化。这个变量在二进制内核里被定位后，能被越狱修改。在iOS5里，comex没有修改变量，而是选择修改检查的代码。即使在将来的iOS版本里该变量不再使用，直接修改代码这个方法也仍然可行。

datautils0里的内核修改补丁通过搜索如下的字符串来定位检查代码：

df f8 88 33 1d ee 90 0fa2 6a1b 68 00 2b

它的反汇编代码是：

80045730 LDR.W R3, =dword_802DE330

80045734 MRC p15, 0, R0,c13,c0, 4
80045738 LDR R2, [R4,#0x28]
8004573A LDR R3, [R3]
8004573C CMP R3, #0

可以看到cs_enforcement_disable变量位于0x802DE330地址，它的值被装入R3寄存器，再和0做比较。最简单的办法是修改为在R3寄存器里装入1。变量在vm_fault_enter（）的使用也很容易修改，因为编译器生成的代码不重新装载变量，而是使用寄存器的缓存副本。

AMFI模块的cs_enforcement_disable变量

第四章讨论过的苹果手机文件完整性 Apple Mobile File Integrity (AMFI) 内核模块，用来检查一些参数的存在性。其中一个是cs_enforcement_disable。如果设置了这个参数，它会影响AMFI_vnode_check_exec()函数的处理策略。在策略检查的反编译代码里可以看到，通过设置进程签名代码标志里的CS_HARD和CS_KILL标志，它停止了AMFI内核模块。

int AMFI_vnode_check_exec(kauth_cred_t cred, struct vnode *vp, struct label

*label, struct label *execlabel, struct componentname *cnp, u_int *csflags)
{
if ( !cs_enforcement_disable )
{
if ( !csflags )
Assert(
“/SourceCache/AppleMobileFileIntegrity/AppleMobileFileIntegrity-
79/AppleMobileFileIntegrity.cpp”,
872,
“csflags”);
*csflags |= CS_HARD|CS_KILL;
}
return 0;
}

如果CS_HARD和CS_KILL标志没有设置，代码签名也能被有效的禁止。然而，不清楚当前的越狱有无修改这个变量，因为execve() 和 posix_spawn()系统调用里使用的mac_vnode_check_exec()的策略检查已经被proc_enforce补丁禁用了，可以在如下代码里看到：

int mac_vnode_check_exec(vfs_context_t ctx, struct vnode *vp,

struct image_params *imgp)
{
kauth_cred_t cred;
int error;

if (!mac_vnode_enforce || !mac_proc_enforce)

return (0);
cred = vfs_context_ucred(ctx);
MAC_CHECK(vnode_check_exec, cred, vp, vp->v_label,
(imgp != NULL) ? imgp->ip_execlabelp : NULL,
(imgp != NULL) ? &imgp->ip_ndp->ni_cnd : NULL,
(imgp != NULL) ? &imgp->ip_csflags : NULL);
return (error);
}

如果像大部分公开的越狱所做的，把proc_enforce设为0，AMFI策略检查根本不会执行，而是成功返回。只有在一些我们知道的未公开的越狱里proc_enforce标志没有更改，在这种情况下，补丁是有效的。

PE_i_can_has_debugger函数

iOS内核有一个PE_i_can_has_debugger()函数，内核和一些内核扩展里都用它来决定是否允许调试。例如，如果这个函数返回值是真，那么KDP内核调试器不能使用。XNU的源代码里没有这个函数，因此我们只能通过反编译代码查看：

int PE_i_can_has_debugger(int *pFlag)

{
int v1; // r1@3
if ( pFlag )
{
if ( debug_enable )
v1 = debug_boot_arg;
else
v1 = 0;
*pFlag = v1;
}
return debug_enable;
}

在iOS4.3以前的越狱版本里，这个函数被修改为永远返回为真。我们试着使用KDP内核调试器，但不能工作。由于只是返回真值并没有完全仿真这个原始函数的行为，在一些iOS内核扩展里设置调试启动参数会引起内核错误。所以当前的大部分越狱不再修改这个函数代码，而是修改内核里的debug_enable 变量。为了确定这个变量的地址，必须分析PE_i_can_has_debugger()函数的代码。因为这个变量位于一个未初始化的数据段里，修改只能在运行时进行。为了查找启动时初始化这个变量的代码，需要搜索debug-enabled字符串，可以直接找到把值复制到变量的代码。

vm_map_enter函数

当内存映射到进程的地址空间时，内核函数vm_map_enter()被用于分配一段虚拟地址映射。例如，可以用mmap()系统调用来触发这个函数。这个函数对越狱有吸引力，因为它能使映射的内存不能同时可写和可执行。下面的代码表明了这点。在/osfmk/vm/vm_map.c文件里可以查看这个函数的完整代码，可以看到，如果设置了VM_PROT_WRITE 标志，VM_PROT_EXECUTE标志就被清除。

kern_return_t vm_map_enter(

vm_map_t map,
vm_map_offset_t *address, /* IN/OUT */
vm_map_size_t size,
vm_map_offset_t mask,
int flags,
vm_object_t object,
vm_object_offset_t offset,
boolean_t needs_copy,
vm_prot_t cur_protection,
vm_prot_t max_protection,
vm_inherit_t inheritance)
{
…
if (cur_protection & VM_PROT_WRITE){
if ((cur_protection & VM_PROT_EXECUTE) && !(flags &
VM_FLAGS_MAP_JIT)){
printf(“EMBEDDED: %s curprot cannot be write+execute.
turning off execute\n”, _PRETTY_FUNCTION_);
cur_protection &= ∼VM_PROT_EXECUTE;
}
}

像你在第四章里看到的一样，这个规则有个叫做JIT(just-in-time)映射的例外。这是一个特殊类型的内存区域，允许同时可写和可执行，MobileSafari手机浏览器里的JIT javaScript编译器需要这样做。一个应用程序只有当它有动态代码签名，才能使用一次这种例外。

迄今为止，只有MobileSafari手机浏览器才能这样。其他所有应用程序都没有可以修改自身的代码、动态代码生成器或JIT编译器，不具有第四章讨论过的 Charlie Miller发现的动态代码签名漏洞。为了完全越狱，需要取消这个限制；因为它不允许应用程序的运行时修改，而这是流行的MobileSubstrate公用程序库所需要的。另外，越狱过的iPhone里的一些模拟器需要自我修改的代码。

为了找到修改这个检查的最好方法，需要查看iOS的二进制内核。虽然没有vm_map_enter() 函数的符号标志，但通过查找包含vm_map_enter的字符串，很容易找到这个函数。通过查看检查功能的ARM的汇编代码，可以发现有多个不同的取消检查的方法，并且只需要修改一个字节。例如， AND.W R0, R1, #6 可以改为 AND.W R0, R1, #8，或BIC.W R0, R0, #4 可以改为 BIC.W R0, R0, #0：

800497C6 LDR R1, [R7,#cur_protection]

800497C8 AND.W R0, R4, #0×80000
800497CC STR R0, [SP,#0xB8+var_54]
800497CE STR R1, [SP,#0xB8+var_78]
800497D0 AND.W R0, R1, #6
800497D4 CMP R0, #6
800497D6 ITT EQ
800497D8 LDREQ R0, [SP,#0xB8+var_54]
800497DA CMPEQ R0, #0
800497DC BNE loc_800497F0
800497DE LDR.W R1, =aKern_return_
800497E2 MOVS R0, #0
800497E4 BL sub_8001D608
800497E8 LDR R0, [R7,#cur_protection]
800497EA BIC.W R0, R0, #4
800497EE STR R0, [SP,#0xB8+var_78]

对于用iPhone越狱来进行安全研究和shell访问的的人来说，这个修改是不需要的。对这个限制进行修改实际上适得其反，因为手机的表现更像iPhone的默认行为。

vm_map_protect函数

vm_map_protect() 是内核函数，它在映射内存的保护改变时被调用。例如，你可以用mprotect()系统调用来触发它。和vm_map_enter()函数类似，它不能把内存保护改为同时可写可执行。下面的代码表明了这点。如果要看更多的细节，在/osfmk/vm/vm_map.c文件里可以查看这个函数的完整代码，可以看到，如果设置了VM_PROT_WRITE 标志，VM_PROT_EXECUTE标志又会被清除。

kern_return_t vm_map_protect(

register vm_map_t map,
register vm_map_offset_t start,
register vm_map_offset_t end,
register vm_prot_t new_prot,
register boolean_t set_max)
{
. . .

#if CONFIG_EMBEDDED

if (new_prot & VM_PROT_WRITE) {
if ((new_prot & VM_PROT_EXECUTE) && !(current->used_for_jit)) {
printf(„EMBEDDED: %s can’t have both write and exec at the
same time\n”, _FUNCTION_);
new_prot &= ∼VM_PROT_EXECUTE;
}
}
#endif

你又能发现一个用于JIT范围的内存例外，这只能由动态代码签名权限的应用程序创建。没有其他应用程序能利用mprotect() 函数来使内存同时可写可执行。所以标准的越狱修改了这个检查，使应用程序允许前面分配的内存可写可执行。

为了修改这个函数首先需要找到它，虽然没有指向这个函数的内核符号指针，但在函数里有vm_map_protec字符串的引用，使它容易发现。再次查看你看到的ARM的汇编代码，两个可选择的单字节修改可以用来清除这个安全检查。 AND.W R1,R6, #6 可以改为 AND.W R1, R6, #8; 或 BIC.W R6, R6, #4 可以改为BIC.W R6, R6, #0:

8004A950 AND.W R1, R6, #6

8004A954 CMP R1, #6
8004A956 IT EQ
8004A958 TSTEQ.W R0, #0×40000000
8004A95C BNE loc_8004A96A
8004A95E BIC.W R6, R6, #4

因为这个修改，越狱减弱了iOS设备的内存保护。我们建议当用户想运行需要修改自身代码的应用程序时才进行这个补丁修改。这些修改的问题是禁止了非执行内存的限制，以致对iPhone应用程序的远程攻击不需要实现百分百的ROP操作。相反，这些攻击或恶意程序只需要一个利用 mprotect()函数来注入执行代码的简短的ROP stub。

AMFI二进制信任缓冲区(AMFI Binary Trust Cache)

AMFI内核模块负责检查签名代码blob的数字签名的合法性。它注册了一些MAC的策略处理句柄，如vnode_check_signature钩子，它在每次内核增加一个新的签名代码blob时都会被调用；AMFI处理程序对来自苹果的证书验证签名。然而，如果基于bootrom或 iBoot的越狱设置了启动参数amfi_get_out_of_my_way 或 amfi_allow_any_signature，验证将会被跳过。如果签名代码的blob的SHA1散列可以在一个内置的大于2200个已知散列里找到的话，验证也会被跳过。这些散列就是AMFI二进制信任缓冲区。这个信任缓冲区的查找是在一个单独函数里实现的，comex对它进行了修改，使它总是返回为成功。这使得AMFI相信每个签名都在缓冲区里，是可信任的；这有效的禁止了签名代码blob的数字签名。

可以通过在AMFI的MAC策略表里搜索AMFI的叫做vnode_check_signature 的MAC策略处理程序，通过搜索第一个内部函数调用来找到这个函数的地址。另一个办法是在内核里搜索如下字节匹配模式：f0 b5 03 af 2d e9 00 05 04 46 .. ..14 f8 01 0b4ff0 130c

这些代码接下来被一个返回为真的函数覆盖后，会帮助跳过数字签名验证。但对内核修改的进一步研究会发现这根本不需要。因为查看在/security/mac_vfs.c里定义的 mac_vnode_check_signature代码，可以发现AMFi处理程序已经被前面的proc_enforce修改完全禁止了。

int mac_vnode_check_signature(struct vnode *vp, unsigned char *sha1, void *

signature, size_t size)

{

int error;

if (!mac_vnode_enforce || !mac_proc_enforce)

return (0);

MAC_CHECK(vnode_check_signature, vp, vp->v_label, sha1, signature, size);

return (error);

}

如果the mac_proc_enforce标志被禁止，AMFI的vnode_check_signature检查将不会被调用。所有的MAC策略处理程序都会使用AMFI二进制信任缓冲区。

0号进程任务陷阱(Task_for_pid 0)

虽然这个补丁修改对大部分越狱者都是没有必要的，我们把它记录下来是因为它涉及一个mach陷阱，下面我们会向你介绍一种在iOS二进制内核里寻找mach陷阱表（mach_trap_table）的策略。

task_for_pid()是一个mach陷阱，会返回另一个进程的任务端口，以它的进程ID命名。这局限于用户ID相同的进程，除非请求任务端口的进程是特权的。在早期的Mac OS X版本中，通过请求0号进程的任务端口可以得到内核进程的任务端口。在Mac OS X的rootkit里使用了这项技术，因为它允许用户空间的进程去读写专有的内核空间。

这可能是task_for_pid()不再允许访问0号进程的任务端口的原因，可以在XNU源代码的/bsd/vm/vm_unix.c文件里的的看到如下的代码：

kern_return_t task_for_pid(struct task_for_pid_args *args)

{

mach_port_name_t target_tport = args->target_tport;

int pid = args->pid;

user_addr_t task_addr = args->t;

proc_t p = PROC_NULL;

task_t t1 = TASK_NULL;

mach_port_name_t tret = MACH_PORT_NULL;

ipc_port_t tfpport;

void * sright;

int error = 0;

AUDIT_MACH_SYSCALL_ENTER(AUE_TASKFORPID);

AUDIT_ARG(pid, pid);

AUDIT_ARG(mach_port1, target_tport);

/* Always check if pid == 0 */

if (pid == 0) {

(void ) copyout((char *)&t1, task_addr, sizeof(mach_port_name_t));

AUDIT_MACH_SYSCALL_EXIT(KERN_FAILURE);

return(KERN_FAILURE);

}

可以看到，对于0号进程有一个详细的检查，如果是0号进程会直接返回一个错误代码。Comex修改了这个检查，把if语句里的条件跳转改为无条件跳转。修改的地址是通过对西面字节的模式匹配发现的：91 e8 01 04 d1 f8 08 80 00 21 02 91 ba f1 000f01 91

发现修改地方的另一个办法是在mach的陷阱表里搜索task_for_pid()函数的地址。然而定义在/osfmk/kern/syscall_sw.c里的mach陷阱表的符号表并未输出，需要额外的办法去发现它。当你查看表的定义时会看到如下类似的东西：

mach_trap_t mach_trap_table[MACH_TRAP_TABLE_COUNT] = {

/* 0 */ MACH_TRAP(kern_invalid, 0, NULL, NULL),

/* 1 */ MACH_TRAP(kern_invalid, 0, NULL, NULL),

/* 2 */ MACH_TRAP(kern_invalid, 0, NULL, NULL),

. . .

/* 26 */ MACH_TRAP(mach_reply_port, 0, NULL, NULL),

/* 27 */ MACH_TRAP(thread_self_trap, 0, NULL, NULL),

/* 28 */ MACH_TRAP(task_self_trap, 0, NULL, NULL),

. . .

/* 45 */ MACH_TRAP(task_for_pid, 3, munge_www, munge_ddd),

可以看到，这张表的前面是一些非法内核陷阱，这可以用来检测内存里的mach陷阱表的地址。在公开的XNU的源代码里定义的表，可以看到前面26个mach陷阱是非法的。然而查看iOS内核，却发现只有前面10个mach陷阱是非法的。

不幸的是，kern_invalid()函数没有输出，无法用来查找第一个mach陷阱。但这个问题可以解决，因为在下面的代码可以看到它引用了一个很有揭示作用的字符串。

kern_return_t kern_invalid(_unused struct kern_invalid_args *args)

{

if (kern_invalid_debug) Debugger(“kern_invalid mach trap”);

return(KERN_INVALID_ARGUMENT);

}

由于这个引用的字符串在整个代码里只使用了一次，对这个字符串的唯一的交叉引用是在kern_invalid()函数里。通过这个地址的帮助，在搜索四字节“0”和紧随的这个函数地址的重复匹配模式后，可以找到mach陷阱表。但在当前的iOS内核里，查找mach陷阱表并不需要kern_invalid()函数的地址，四字节“0”的重复匹配模式的搜索后的指针已经足够发现mach陷阱表了。

修改沙盒 (Sandbox Patches)

Comex的内核修改补丁的最后一步就是改变沙盒的行为。没有这个修改补丁，越狱过的iPhone不能运行类似MobileSafari 和 MobileMail的应用程序。因为越狱后/Applications 目录已经移到/var/stash/Applications目录了，这违反了沙盒机制。 奇怪的是目前我们只知道这两个应用程序受影响。即使没有修改沙盒，其他所有的内置应用程序看起来都能完美运行。

这个修改补丁包括两块：第一块是用钩子（hook）覆盖sb_evaluate()函数的开始部分；第二块是在内核里的未用区域写入新代码。这个函数的更多信息，可以回顾第5章。这个修改补丁改变了沙盒仿真的修为，去存取处理不同的特定的目录。

在描述新的仿真功能前，由于没有符号表可用，我们要找到在内核代码里定位sb_evaluate()函数的方法。一个可能性是在沙盒内核扩展里搜索mac策略处理程序表。一些mac策略处理程序会用到sb_evaluate()函数。当前的iOS内核里很容易搜索到错误操作码的字符。因为只能在你的函数里使用它，一旦找到它的数据引用，你要找到它所用的函数的开始的地方。

定位到sb_evaluate()函数的地址后，可以给它安装一个钩子函数，让它跳转到一个内核未使用的区域，在那里你可以放置其余代码。我们在第9章里已经讨论过如何找到未用的区域。在comex的GitHub程序库里可以找到datautils0程序的源代码，它模拟了钩子的功能，我们现在要仔细研究它。代码的总体想法是对/private/var/mobile和private/var/mobile/Library/Preferences里的文件避免进行沙盒检查。代码开始时检查提供的vnode是否为0，如果为0，不调用钩子函数，只是跳过检查，执行原来的处理程序。

start:

push {r0-r4, lr}

sub sp, #0×44

ldr r4, [r3, #0x14]

cmp r4, #0

beq actually_eval

接下来的代码调用vn_getpath()函数，得到提供的vnode的路径。如果返回错误，ENOSPC错误被忽略，其他错误则返回执行原来的处理程序。

ldr r3, vn_getpath

mov r1, sp

movs r0, #0×40

add r2, sp, #0×40

str r0, [r2]

mov r0, r4

blx r3

cmp r0, #28

beq enospc

cmp r0, #0

bne actually_eval

如果没有错误返回或者没有获得路径全名的足够空间，返回的路径名称会和/private/var/mobile字符串作比较，如果不匹配，允许访问：

enospc:

# that error’s okay…

mov r0, sp

adr r1, var_mobile ; # “/private/var/mobile”

movs r2, #19 ;# len(var_mobile)

ldr r3, memcmp

blx r3

cmp r0, #0

bne allow

如果路径名称匹配，会再和/private/var/mobile/Library/Preferences/com.apple相匹配，如果匹配成功，会调用原来的sb_evaluate()函数。

mov r0, sp

adr r1, pref_com_apple

; # “/private/var/mobile/Library/Preferences/com.apple”

movs r2, #49 ;# len(preferences_com_apple)

ldr r3, memcmp

blx r3

cmp r0, #0

beq actually_eval

下一步检查路径名称是否在/private/var/mobile/Library/Preferences 里，如果是，允许访问；否则调用原来的处理程序：

mov r0, sp

adr r1, preferences ;# “/private/var/mobile/Library/Preferences”

movs r2, #39 ;# len(preferences)

ldr r3, memcmp

blx r3

cmp r0, #0

bne actually_eval

代码记下允许访问的信息，返回提供的数据结构，第5章里有更详细的描写：

allow:

# it’s not in /var/mobile but we have a path, let it through

add sp, #0×44

pop {r0}

movs r1, #0

str r1, [r0]

movs r1, #0×18

strb r1, [r0, #4]

pop {r1-r4, pc}

代码的其他部分只是跳过检查，返回原来的函数。因为这只是标准的API拦截技术，这里不进行讨论。

清空缓存(Clearing the Caches)

由于整个内核镜像是在一个可读、可写、可执行的内存里，前面的内核补丁修改都是直接了当进行的。然而内核级别的payload可以对原始代码打补丁，不需要修改内存的许可权限。

唯一混乱的地方是修改内核时CPU指令和数据缓存需要清空，否则越狱修改的结果不会立即激活。

为了达到这个目的，漏洞payload应该在每次修改内核代码或数据时立即调用iOS内核的两个输出函数。为了清空指令缓存，需要调用invalidate_icache()函数，这个函数需要三个参数，第一个参数是要清空的无效的内存区域地址，第二个参数是内存区域的长度，第三个参数应该是0。

清空数据缓存的函数是flush_dcache()函数，也需要同样的三个调用参数。

清理并返回(Clean Return)

提升权限，修改安全特性，脱离内核的掌控后，唯一要做的是让内核空间保持在清洁的状态，防止内核不稳定或立即崩溃。通常只需要把普遍意义上的CPU寄存器的值恢复到调用内核payload前的值，并返回保存的程序指针。万一内核堆栈被溢出了，由于真实的堆栈的值被溢出的缓冲区覆盖，导致不可能恢复。这种情况下，可以返回一个未被破坏的以前的堆栈帧。

另一个退出内核的办法是调用内核的thread_exception_return()函数。由于这个函数在内核里没有符号表，需要通过模式匹配扫描和交叉引用扫描找到这个函数。当内核堆栈帧不可能回退时需要执行完当前内核线程的例外情况，需要调用这个函数来恢复内核。因此，可以用它来离开漏洞payload后的内核。但是，内核应该尽可能通过返回正确的堆栈帧的来离开，否者，离开后内核不能保证还在稳定状态。

小结Summary

越狱被大多数人认为是个黑盒子，这一章我们对它进行了深入研究。介绍了为安全研究使用越狱后的iPhone而不是原始iPhone的原因，讨论了不同类型的越狱的优缺点。

我们分析了红雪越狱工具的内部工作原理，介绍了越狱过程的每一个步骤。从可用性和安全性的角度说明了越狱后的iPhone和未越狱的iPhone的不同点。

我们还说明了越狱所使用的内核修改补丁，讨论了每一个补丁的的原由、如何发现修改地址、修改的方法。有了这些知识，你也可以不依靠越狱社区，自己把这些修改补丁移植到将来的iOS版本。
