> However, Linux also includes processes in uninterruptible sleep states (usually waiting for [disk](https://en.wikipedia.org/wiki/Hard_disk_drive "Hard disk drive") activity), which can lead to markedly different results if many processes remain blocked in [I/O](https://en.wikipedia.org/wiki/Input/output "Input/output") due to a busy or stalled I/O system.[[1]](https://en.wikipedia.org/wiki/Load_(computing)#cite_note-1)This, for example, includes processes blocking due to an [NFS](https://en.wikipedia.org/wiki/Network_File_System "Network File System") server failure...

-- https://en.wikipedia.org/wiki/Load_(computing)

Number of tasks in runqueue does not necessarily correlate with system load. Runqueue will not include processes that are hanging on I/O, but system load will.

Can see system load & runqueue with `sar -q` and typically history is stored in `/var/log/sa`.

```bash
[mnguyen1@chcss312 ~]$ sar -q -f /var/log/sa/sa14
Linux 5.10.154-1.base.x86_64 (chcss312.trading.imc.intra) 	08/14/2023 	_x86_64_	(72 CPU)

12:00:01 AM   runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15   blocked
12:10:01 AM        40      2247     23.38     18.37     15.27         0
12:20:01 AM        21      2010      7.49      8.26     11.47         0
12:30:01 AM        31      2003     10.14      7.65      9.46         0
12:40:01 AM        23      4430     11.79     18.35     15.84         0
12:50:01 AM        25      1958      6.06      7.28     10.77         0
01:00:01 AM        21      1966      9.34      7.77      9.09         0
01:10:01 AM        31      2026      6.44      5.89      7.24         0

```