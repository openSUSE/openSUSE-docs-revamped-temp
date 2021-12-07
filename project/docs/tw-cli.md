## tumbleweed-cli

!!! warning
    `tumbleweed-cli` is an experimental tool that comes with a couple of safety issues:
    
    - the evaluation algorithm used to [collect statistics](https://review.tumbleweed.boombatower.com/) about the 'health condition' of available Tumbleweed snapshots has been alleged to produce false positives.
    
    - the advantages of rolling back to a previous Tumbleweed snapshot comes with the drawback that the user is no longer in sync with possible upcoming security fixes, which are shipped to target only the latest available snapshot.

This tool is an _unsupported_ community-made tool exclusively designed for _Tumbleweed_. If you have not already, please read the short [introduction](updating_upgrading_reverting.md#introduction).

`tumbleweed-cli` enables the user to control finely the update process by making explicit which Tumbleweed snapshot one is upgrading to. It also allows the user to select and rollback (or fast-forward) to any Tumbleweed snapshot comprised in a collection of recently released snapshots.

!!! info
    In this paragraph _snapshot_ is used to denote Tumbleweed snapshots unless qualified otherwise. Tumbleweed snapshots are "slices" coming out Factory -- the repository and production line for the Tumbleweed distribution. They consist of a collection of updates targeting a subset of all packages available. Do not conflate them with _Btrfs snapshots_, which are on-disk temporal stages of the file system. Both are discussed below.

`tumbleweed-cli` may come in handy for users running into outstanding issues experienced with the latest or several snapshots across several snapshots -- including the latest one. It may be useful when three conditions occur:

1. There is evidence that the cause(s) of the issue is one or several system packages; and
2. The likely cause(s) "spill over" several single _Btrfs snapshots_ including the latest one; and
3. No previous _Btrfs snapshot_ captures the state of the system which the user desires to rollback to.

In addition to the _responsive_ workflow described above, the user might use the tool _preventively_, by updating their system to a Tumbleweed snapshot located before the latest snapshot provided by Factory. Notice however that both the responsive and the preventive workflows could make your system unstable. This happens typically with packages provided by foreign repositories, which escape the scope of the rollbacks/fast-forwards mechanism. When foreign packages rely on dependences not or longer satisfied by the state of your system after rolling-back/fast-forwarding, they become unusable. Proceed only if you can accomodate these limitations.

### Installation and use
To install `tumbleweed-cli`:

`zypper in tumbleweed-cli`

To initialize it:

`tumbleweed init`

The developer suggests to run this command to verify that the URLs of the repos contain a snapshot version:

`zypper lr -EUP`

Once this is done you can finally take advantage of the snapshotting system employed for producing Tumbleweed packages, keeping your system in sync with recent snapshot of your choice. Updating to the latest snapshot released is done with:

`tumbleweed update`

As explained above, packages installed from foreign repositories are not included in Tumbleweed snapshots, and will not update when using `tumbleweed-cli`. For these packages you will still have to run `(sudo zypper ref &&) sudo zypper dup`.

At any time you can then check the status of your system with respect to the latest snapshots released:

`tumbleweed status`

which returns for example:

```
latest   : 20210414
target   : 20210414
installed: 20210414
```

where `target` refers to the next snapshot the system is going to update to with `tumbleweed-update`.

Setting the target to the previous snapshot and reverting to it is performed with:

`tumbleweed revert --install`

or if the previous snapshot is for instance 20210411, the equivalent:

```
tumbleweed switch 20210411
sudo zypper dup
```

`tumbleweed-switch` is thus the more generic option allowing you to set the target to whichever snapshot you fancy, provided it is available. The full list of snapshots available for switching is obtained with:

`tumbleweed history`

### Uninstall
Should you want to uninstall `tumbleweed-cli`, make sure you deactivate its particular update mechanism before. This is done with:

`tumbleweed uninit`

From now on `zypper-dup` will again fetch updates across all snapshots, which is the default. You can remove `tumbleweed-cli` with:

`sudo zypper rm tumbleweed-cli`

## Further reading
* [tumbleweed-cli](https://github.com/boombatower/tumbleweed-cli)
* [snapshow reviewer](https://review.tumbleweed.boombatower.com/)
