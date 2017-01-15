# Chocolatey packages

This repository contains the sources of the [packages I maintain or have maintained in the past](https://chocolatey.org/profiles/wget/) for [Chocolatey, the package manager for Windows](https://chocolatey.org/).

## Cloning

Each package consists in a git submodule. This way, packages are independant of each other and Chocolatay admins or even vendors can pick the package they want without cloning everything or rewriting the history to only keep the history related to their packages. Also, proceeding this way allows to merge a package keeping its history with ease thanks to a simple `git merge --allow-unrelated-histories`.

You have two ways when working with this repository. Either work with all of the packages or work with the one you are interested in.

### Work with all packages

* Since adding submodules freezes the state of the submodules from the moment they were added to this repository, simply cloning will not be enough. You will need to get upstream changes for these repositories as well.
* Also, pulling changes from upstream clones by default changes from the branch called `master` into a local branch of the same name, but the local branch will not track changes and will be in a detached HEAD state. In this case, you will need to checkout that branch in order for the local HEAD pointer to follow the remote HEAD pointer branch. Now, you will be able to work on these submodules and commit upstream directly from the submodule directory.
 
The following commands take care of these caveats.

    git clone --recursive git@github.com:wget/chocolatey_packages.git
    git submodule foreach 'git checkout master'

Update parent and submodules from upstream, trying to replay the committed changes still not pushed upstream at the top of your locally committed changes:

    git submodule update --remote --rebase

Push changes made in submodules, then try to push changes made to this parent directory:

    git push --recurse-submodules=on-demand
    
For more information, please read the [chapter related to submodules from the progit2 book](https://github.com/progit/progit2/blob/master/book/07-git-tools/sections/submodules.asc).

### Work with only one package

Simply click on the submodule folder in the Web UI, this should lead you to the URL of the submodule on Github. The workfload is as usual from there.

Please note that some packages make use of a [library](https://github.com/wget/chocolatey_custom_functions) used as a submodule. In that case, use the following command:

    git clone --recursive git@github.com:wget/chocolatey_package_<package name>.git

If you want to be able to publish changes to that library as well, either clone it as a separate repository or use the git submodules commands described above in the [Work with all packages section](#Work_with_all_packages).

## Prerequisites

The packages need to be compliant with Powershell 2.0 running the .NET framework 4.0 for the following reasons:

* This is a [requirement to install chocolatey](https://chocolatey.org/install#requirements).
* Powershell 2.0 is the latest version Windows Vista can support. The latter ends its extended support on 2017-04-11. 
* Sysadmins often prepare Windows 7/Windows Server 2008 installation images when recent Powershell and .NET framework versions are not installed yet. These Windows versions are only bundled with Powershell 2.0 and .NET 4.0 by default. [(source)](https://chocolatey.org/packages/openvpn#comment-2991181108) Sysadmins usually do not update Powershell and .NET on these machines to make sure pieces of software developed for these particular versions will still work (often government sofware used in administrations, particularly slow to move foward).
* Windows Server (2008 and 2008 R2) will receive an additional 6 years support for companies paying an additional subscription. [(source)](https://blogs.technet.microsoft.com/hybridcloud/2016/12/08/introducing-windows-server-premium-assurance-and-sql-server-premium-assurance/) The support for Windows Server 2008 and 2008 R2 will thus not end on 2020-01-14, but on 2026-01-14. Powershell 3.0 is the latest release Windows Server 2008 can support. [(source)](https://en.wikipedia.org/wiki/PowerShell#PowerShell_3.0)

The [Chocolatey Automatic Package Updater Module](https://github.com/majkinetor/au) does require at least Powershell 5.0.

To solve this [dependency hell](https://en.wikipedia.org/wiki/Dependency_hell), you will need two machines:

* One running Windows 7 SP1 or Windows Server 2008 R2 SP1. You can install the recommended security upgrades as they are not changing the Powershell nor the .NET framework versions. To install updates faster, install the [Convenience rollup update](https://support.microsoft.com/en-us/help/3125574) first which will update your install from 2011-02-22 to 2015-04, then run Windows Update for the remaining updates from 2015-04 to now.
* Another machine [running at least Windows Powershell 5.0](https://msdn.microsoft.com/en-us/powershell/scripting/setup/windows-powershell-system-requirements#operating-system-requirements).

To learn more about the Windows Powershell availability, please read [this article](https://4sysops.com/archives/powershell-versions-and-their-windows-version/).

## Testing

Launch a PowerShell prompt as Administrator.

Build the `.nupkg` package from the `.nuspec` file with

    cpack

Test and install the `.nupkg` package with the following line.

* `-source` is used to specify where to find the sources. As our package uses gpg as a dependency to check the signatures, we need to specify from where to get it (here, from the Chocolatey website)
* The install must be forced (`-f`) if the same version is already installed (will remove and reinstall the package from the updated `.nupkg`)
* Test is performed in debug mode (`-d`)
* Being verbose (`-v`)
* Avoid asking for confirmation when installing the package (`--yes`)

More information is available in the [Chocolatey documentation](https://chocolatey.org/docs/create-packages#testing-your-package).

    choco install <package name> -fdv -source "'.;https://chocolatey.org/api/v2/'" --yes
    
Do not forget to test the uninstallation as well:

    choco uninstall <package name> -dv --yes

## Deploy to Chocolatey

Get your API key on your [Chocolatey account page](https://chocolatey.org/account). The command you will need to type is like this one:

    choco apiKey -k 12345678-90ab-cdef-1234-567890abcdef -source https://chocolatey.org/

Push your package to moderation review:

    choco push <package name><version>.nupkg -s https://chocolatey.org/

About 30 minutes later, you should receive an email revealing if the automatic tests have passed. [If all tests have succeeded](https://github.com/chocolatey/package-validator/wiki#requirements), the package will be ready for review and will be approved manually within 48 hours by a Chocolatey admin.

For urgent releases like CVE fixes, connect to the [Chocolatey channel on Gitter](https://gitter.im/chocolatey/choco) and ping one of the following person:
* [Rob Reynolds (@ferventcoder)](https://github.com/ferventcoder), Chocolatey founder
* [Gary Ewan Park (@gep13)](https://github.com/gep13)

## Contribute

If you have comments to make or push requests to submit, feel free to contribute to this repository or one of its submodules.

## License

[As Apache 2 software can be included in GPLv3 projects, but GPLv3 software cannot be included in Apache projects](https://www.apache.org/licenses/GPL-compatibility.html) and in order to comply with [NuGet](https://www.nuget.org/policies/About) and Chocolatey licenses, this software is licensed under the terms of the Apache License 2.0. 
