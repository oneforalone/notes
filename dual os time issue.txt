RTC：Real-Time Clock，即实时时钟，在计算机领域作为硬件时钟的简称。

世界上不同地区所在的时区是不同的，这些时区决定了当地的本地时间。比如北京处于东八区，即北京时间为 UTC + 8，如果 UTC 时间现在是上午 6 点整，那么北京时间为 14 点整。

Windows 与 Linux 看待硬件时间的方式不同。Windows 把电脑的硬件时钟（RTC）看成是本地时间，即 RTC = Local Time，Windows 会直接显示硬件时间；而 Linux 则是把电脑的硬件时钟看成 UTC 时间，即 RTC = UTC，那么 Linux 显示的时间就是硬件时间加上时区

解决办法
既然知道了问题原因，我们就知道如何去解决，大概思路分为两种，一是让 Windows 认为硬件时钟是 UTC 时间，二是让 Linux 认为硬件时钟是本地时间。

修改 Windows 硬件时钟为 UTC 时间
以管理员身份打开 「PowerShell」，输入以下命令：

> Reg add HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation /v RealTimeIsUniversal /t REG_DWORD /d 1
或者打开「注册表编辑器」，定位到 计算机\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation 目录下，新建一个 DWORD 类型，名称为 RealTimeIsUniversal 的键，并修改键值为 1 即可。

修改 Linux 硬件时钟为本地时间
这里大家可以根据自己 Linux 发行版的方法来修改。我这里给出现在大多数发行版由 timedatectl 来管理时间的修改方法。

在终端中，输入以下命令：

$ timedatectl set-local-rtc 1 --adjust-system-clock
现在，系统时间显示正常了。

这两种方法中，本人推荐第一种方法，修改 Windows 的时间管理。因为在 Linux 系统中修改后，输入 timedatectl 命令后，会出现警告，提示你使用 RTC 时钟会导致一些程序错误；并且 Windows 也在更改时间管理方式为 UTC 时间。