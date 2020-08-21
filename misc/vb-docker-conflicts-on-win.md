bcdedit /set hypervisorlaunchtype off ;; enable virtualbox the error is VT-x is not available

reference: https://forums.virtualbox.org/viewtopic.php?f=38&t=89791

To make docker run properly:
bcdedit /set hypervisorlaunchtype auto
