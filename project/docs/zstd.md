## Enable zstd compression
First enable `zstd` compression on `/` (root) and `/home` by adding the corresponding parameters to `/etc/fstab`:

```
<UUID> / btrfs compress=zstd:1 0 0
<UUID> /home btrfs subvol=/@/home,compress=zstd:1 0 0
```

This will apply `zstd` compression to all the files directly under the relevant directories. The files already present under these directories will not be touched, however. To retroactively compress them, use:

`sudo btrfs filesystem defrag -czstd -rv / /home/`

## Costs & benefits
Compression is about making your files take less space on your hard drive, but it requires extra computations when reading or writing. In other words it trades (very marginally increased) processor and RAM usage for less hard drive usage. The tradeoff is likely to benefit the most modern machines with solid state drives (_SSD_), which are likely to have less storage capacity than their desktop counterparts, while having comparable computational power. 

It should also be noticed that `zstd`compression can increase the lifespan of SSDs, as it may decrease reads over time.

## References & credits
- [SDB:Btrfs](https://en.opensuse.org/SDB:BTRFS)
    - [authors](https://en.opensuse.org/index.php?title=SDB:BTRFS&action=history)