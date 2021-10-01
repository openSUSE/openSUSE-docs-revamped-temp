## Safety and usability on Tumbleweed

There are two "don't-s" to keep in mind when using Tumbleweed:

- don't use the "1-click installers" available from https://software.opensuse.org
- don't install from [Open Build Service](https://build.opensuse.org/) `home:` repositories

Apart from that finding the sweet spot between safety and usability mostly depends on your preferences and your use-case.

### Finding where you fit and moving from there 

Linux distributions try to strike a balance between the time and effort expected from two groups: 

- maintainers and developers, whose main goal is to produce the operating system(s) of the distribution with reproducible and streamlined processes;
- end-users, whose main goal is to keep their running installation productive and healthy without too much hassle.

 Moreover individuals have different priorities, preferences and degrees of curiosity. For Tumbleweed this means different possible answers to the question: "How should I maintain my running system productive and healthy?".

On one end there might be users who want to make the least possible effort to keep their system productive and healthy, or are not deeply interested in learning how it works. On the other end there might be users willing to sacrifice a lot of time and effort for tinkering and learning -- and hopefully in the long run, [contributing to openSUSE](https://en.opensuse.org/Portal:How_to_participate).

### The lazy way

This is a workflow where the user minimizes the need for manual interventions on their running installation. The way is fine for most users who:

- don't want to spend time configuring their system; or 
- don't want to take any risk associated with maintenance routines.

Here are some rules of thumb for following this approach:

1. When installing user applications, favour containerized versions (ideally _flatpaks_, see [here](/alternative_procurement#flatpaks)). Use software from the official repositories only when no containerized version is available.
2. Do not manually install important system libraries or codecs directly from _zypper_ or _Yast2_. Use [opi](/best-of-post#codecs) if you must, but prefer software that ship with their own static libraries and codecs when possible (or flatpaks).
3. Update every other week or twice a month, not more often. Use `sudo zypper dup`, plus if that applies to you, `flatpak update`.
4. If you need fresh updates for essential parts of your toolchain (typically when working with a programming language stack), always favour the language's official, remote installer.
5. If you don't need to always be running the latest kernel (and that's likely your case), use the multiversion zypp setting described [here](/updating_upgrading_reverting#reverting-to-a-previous-kernel-image).

### Start lazy, get busy!

We encourage users to deviate -- little by little, as they build skills and self-confidence -- from the "lazy way" described above.

Doing so means that users will be more often using packages from the official repositories and get familiar with the typical openSUSE tools (such as _Zypper_, _Snapper_ and _Yast_) and configuration scenarios. In the long run this will strengthen their understanding of Linux and of the openSUSE distributions, while improving their ability to troubleshoot with autonomy. 

At scale the openSUSE Project benefits more from users in position to report bugs, provide feedback and make direct contributions.

### openSUSE MicroOS as desktop workstation

Although it is not recommended for new users, it is possible to use MicroOS on a desktop computer, taking advantage of transactional updates. Consider our [guide to get started](/microos_getting_started).

This might appeal to advanced users who agree to spend more time configuring their system in exchange for a system that will require little to no maintenance (especially suited for friends and relatives with little to no experience of Linux or openSUSE). 

The reader might also be interested in [the MicroOS documentation](https://microos.opensuse.org/) for the official documentation about this use case.
