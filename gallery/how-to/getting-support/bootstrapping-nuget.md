---
ms.date: 06/12/2017
contributor: manikb
keywords: 库,powershell,cmdlet,psget
title: 正在启动 NuGet
ms.openlocfilehash: f707e23737361ee7f82a16150402c9e719ee0ae1
ms.sourcegitcommit: 54534635eedacf531d8d6344019dc16a50b8b441
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/17/2018
ms.locfileid: "34221793"
---
# <a name="bootstrap-the-nuget-provider-and-nugetexe"></a><span data-ttu-id="53e83-103">启动 NuGet 提供程序和 NuGet.exe</span><span class="sxs-lookup"><span data-stu-id="53e83-103">Bootstrap the NuGet provider and NuGet.exe</span></span>

<span data-ttu-id="53e83-104">最新的 NuGet 提供程序不包含 NuGet.exe。</span><span class="sxs-lookup"><span data-stu-id="53e83-104">NuGet.exe is not included in the latest NuGet provider.</span></span>
<span data-ttu-id="53e83-105">若要对模块或脚本执行发布操作，PowerShellGet 需要使用二进制可执行 NuGet.exe。</span><span class="sxs-lookup"><span data-stu-id="53e83-105">For publish operations of either a module or script, PowerShellGet requires the binary executable NuGet.exe.</span></span>
<span data-ttu-id="53e83-106">对于其他所有操作（包括查找、安装、保存和卸载），只需要使用 NuGet 提供程序。</span><span class="sxs-lookup"><span data-stu-id="53e83-106">Only the NuGet provider is required for all other operations, including *find*, *install*, *save*, and *uninstall*.</span></span>
<span data-ttu-id="53e83-107">PowerShellGet 中的逻辑可以同时启动 NuGet 提供程序和 NuGet.exe，也可以只启动 NuGet 提供程序。</span><span class="sxs-lookup"><span data-stu-id="53e83-107">PowerShellGet includes logic to handle either a combined bootstrap of the NuGet provider and NuGet.exe, or bootstrap of only the NuGet provider.</span></span>
<span data-ttu-id="53e83-108">无论属于上述哪种情况，都应该仅显示一条提示消息。</span><span class="sxs-lookup"><span data-stu-id="53e83-108">In either case, only a single prompt message should occur.</span></span>
<span data-ttu-id="53e83-109">如果计算机未连接 Internet，用户或管理员必须将 NuGet 提供程序和/或 NuGet.exe 文件的受信任实例复制到已断开连接的计算机。</span><span class="sxs-lookup"><span data-stu-id="53e83-109">If the machine is not connected to the Internet, the user or an administrator must copy a trusted instance of the NuGet provider and/or the NuGet.exe file to the disconnected machine.</span></span>

><span data-ttu-id="53e83-110">注意：自版本 6 起，NuGet 提供程序会随 PowerShell 一起安装。</span><span class="sxs-lookup"><span data-stu-id="53e83-110">**Note**: Starting with version 6, the NuGet provider is included in the installation of PowerShell.</span></span> [http://github.com/powershell/powershell](http://github.com/powershell/powershell)

## <a name="resolving-error-when-the-nuget-provider-has-not-been-installed-on-a-machine-that-is-internet-connected"></a><span data-ttu-id="53e83-111">修复在已连接 Internet 的计算机上尚未安装 NuGet 提供程序时出现的错误</span><span class="sxs-lookup"><span data-stu-id="53e83-111">Resolving error when the NuGet provider has not been installed on a machine that is Internet connected</span></span>

```powershell
PS> Find-Module -Repository PSGallery -Verbose -Name Contoso

NuGet provider is required to continue
PowerShellGet requires NuGet provider version '2.8.5.201' or newer to interact with NuGet-based repositories. The NuGet provider must be available in 'C:\Program Files\PackageManagement\ProviderAssemblies' or
'C:\Users\manikb\AppData\Local\PackageManagement\ProviderAssemblies'. You can also install the NuGet provider by running 'Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force'. Do you want PowerShellGet to install and import the NuGet provider
now?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): n
Find-Module : NuGet provider is required to interact with NuGet-based repositories. Please ensure that '2.8.5.201' or newer version of NuGet provider is installed.
At line:1 char:1
+ Find-Module -Repository PSGallery -Verbose -Name Contoso
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (:) [Find-Module], InvalidOperationException
   + FullyQualifiedErrorId : CouldNotInstallNuGetProvider,Find-Module

PS> Find-Module -Repository PSGallery -Verbose -Name Contoso

NuGet provider is required to continue
PowerShellGet requires NuGet provider version '2.8.5.201' or newer to interact with NuGet-based repositories. The NuGet provider must be available in 'C:\Program Files\PackageManagement\ProviderAssemblies' or
'C:\Users\manikb\AppData\Local\PackageManagement\ProviderAssemblies'. You can also install the NuGet provider by running 'Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force'. Do you want PowerShellGet to install and import the NuGet provider
now?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y
VERBOSE: Installing NuGet provider.

Version    Name                                Type       Repository           Description
-------    ----                                ----       ----------           -----------
2.5        Contoso                             Module     PSGallery        Contoso module
```

## <a name="resolving-error-when-the-nuget-provider-is-available-and-nugetexe-is-not-available-during-the-publish-operation-on-a-machine-that-is-internet-connected"></a><span data-ttu-id="53e83-112">修复在已连接 Internet 的计算机上执行发布操作期间 NuGet 提供程序可用但 NuGet.exe 不可用时出现的错误</span><span class="sxs-lookup"><span data-stu-id="53e83-112">Resolving error when the NuGet provider is available and NuGet.exe is not available during the publish operation on a machine that is Internet connected</span></span>

```powershell
PS> Publish-Module -Name Contoso -Repository PSGallery -Verbose

NuGet.exe is required to continue
PowerShellGet requires NuGet.exe to publish an item to the NuGet-based repositories. NuGet.exe must be available under one of the paths specified in PATH environment variable value. Do you want PowerShellGet to install NuGet.exe now?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): N
Publish-Module : NuGet.exe is required to interact with NuGet-based repositories. Please ensure that NuGet.exe is available under one of the paths specified in PATH environment variable value.
At line:1 char:1
+ Publish-Module -Name Contoso -Repository PSGallery -Verbose
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (:) [Publish-Module], InvalidOperationException
    + FullyQualifiedErrorId : CouldNotInstallNuGetExe,Publish-Module

PS> Publish-Module -Name Contoso -Repository PSGallery -Verbose

NuGet.exe is required to continue
PowerShellGet requires NuGet.exe to publish an item to the NuGet-based repositories. NuGet.exe must be available under one of the paths specified in PATH environment variable value. Do you want PowerShellGet to install NuGet.exe now?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y
VERBOSE: Installing NuGet.exe.
VERBOSE: Successfully published module 'Contoso' to the module publish location 'https://www.powershellgallery.com/api/v2/'. Please allow few minutes for 'Contoso' to show up in the search results.
```

## <a name="resolving-error-when-both-nuget-provider-and-nugetexe-are-not-available-during-the-publish-operation-on-a-machine-that-is-internet-connected"></a><span data-ttu-id="53e83-113">修复在已连接 Internet 的计算机上执行发布操作期间 NuGet 提供程序和 NuGet.exe 均不可用时出现的错误</span><span class="sxs-lookup"><span data-stu-id="53e83-113">Resolving error when both NuGet provider and NuGet.exe are not available during the publish operation on a machine that is Internet connected</span></span>

```powershell
PS> Publish-Module -Name Contoso -Repository PSGallery -Verbose

NuGet.exe and NuGet provider are required to continue
PowerShellGet requires NuGet.exe and NuGet provider version '2.8.5.201' or newer to interact with the NuGet-based repositories. Do you want PowerShellGet to install both NuGet.exe and NuGet provider now?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): N
Publish-Module : PowerShellGet requires NuGet.exe and NuGet provider version '2.8.5.201' or newer to interact with the NuGet-based repositories. Please ensure that '2.8.5.201' or newer version of NuGet provider is installed and NuGet.exe is available under
one of the paths specified in PATH environment variable value.
At line:1 char:1
+ Publish-Module -Name Contoso -Repository PSGallery -Verbose
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (:) [Publish-Module], InvalidOperationException
    + FullyQualifiedErrorId : CouldNotInstallNuGetBinaries,Publish-Module

PS> Publish-Module -Name Contoso -Repository PSGallery -Verbose

NuGet.exe and NuGet provider are required to continue
PowerShellGet requires NuGet.exe and NuGet provider version '2.8.5.201' or newer to interact with the NuGet-based repositories. Do you want PowerShellGet to install both NuGet.exe and NuGet provider now?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y
VERBOSE: Installing NuGet provider.
VERBOSE: Installing NuGet.exe.
VERBOSE: Successfully published module 'Contoso' to the module publish location 'https://www.powershellgallery.com/api/v2/'. Please allow few minutes for 'Contoso' to show up in the search results.
```

## <a name="manually-bootstrapping-the-nuget-provider-on-a-machine-that-is-not-connected-to-the-internet"></a><span data-ttu-id="53e83-114">在未连接 Internet 的计算机上手动启动 NuGet 提供程序</span><span class="sxs-lookup"><span data-stu-id="53e83-114">Manually bootstrapping the NuGet provider on a machine that is not connected to the Internet</span></span>

<span data-ttu-id="53e83-115">上面展示的进程假设计算机已连接 Internet，且能从公共位置下载文件。</span><span class="sxs-lookup"><span data-stu-id="53e83-115">The processes demonstrated above assume the machine is connected to the Internet and can download files from a public location.</span></span>
<span data-ttu-id="53e83-116">如果前提条件不成立，只能使用上面展示的进程启动计算机，再通过脱机受信任进程将提供程序手动复制到独立节点。</span><span class="sxs-lookup"><span data-stu-id="53e83-116">If that is not possible, the only option is to bootstrap a machine using the processes given above, and manually copy the provider to the isolated node through an offline trusted process.</span></span>
<span data-ttu-id="53e83-117">此方案的最常见用例是当专用库可用于支持独立环境时。</span><span class="sxs-lookup"><span data-stu-id="53e83-117">The most common use case for this scenario is when a private gallery is available to support an isolated environment.</span></span>

<span data-ttu-id="53e83-118">在遵循上述进程启动已连接 Internet 的计算机后，提供程序文件位于以下位置：</span><span class="sxs-lookup"><span data-stu-id="53e83-118">After following the process above to bootstrap an Internet connected machine, you will find provider files in the location:</span></span>

```
C:\Program Files\PackageManagement\ProviderAssemblies\
```

<span data-ttu-id="53e83-119">NuGet 提供程序的文件夹/文件结构为（版本号可能不同）：</span><span class="sxs-lookup"><span data-stu-id="53e83-119">The folder/file structure of the NuGet provider will be (possibly with a different version number):</span></span>

<span data-ttu-id="53e83-120">NuGet</span><span class="sxs-lookup"><span data-stu-id="53e83-120">NuGet</span></span><br>
<span data-ttu-id="53e83-121">--2.8.5.208</span><span class="sxs-lookup"><span data-stu-id="53e83-121">--2.8.5.208</span></span><br>
<span data-ttu-id="53e83-122">----Microsoft.PackageManagement.NuGetProvider.dll</span><span class="sxs-lookup"><span data-stu-id="53e83-122">----Microsoft.PackageManagement.NuGetProvider.dll</span></span>

<span data-ttu-id="53e83-123">使用受信任进程将这些文件夹和文件复制到脱机计算机。</span><span class="sxs-lookup"><span data-stu-id="53e83-123">Copy these folders and file using a trusted process to the offline machines.</span></span>

## <a name="manually-bootstrapping-nugetexe-to-support-publish-operations-on-a-machine-that-is-not-connected-to-the-internet"></a><span data-ttu-id="53e83-124">在未连接 Internet 的计算机上手动启动 NuGet.exe 以支持发布操作</span><span class="sxs-lookup"><span data-stu-id="53e83-124">Manually bootstrapping NuGet.exe to support publish operations on a machine that is not connected to the Internet</span></span>

<span data-ttu-id="53e83-125">除了手动启动 NuGet 提供程序的进程外，如果计算机将用于使用 Publish-Module 或 Publish-Script cmdlet 向专用库发布模块或脚本，必须使用相关进程启动 NuGet.exe 二进制可执行文件。</span><span class="sxs-lookup"><span data-stu-id="53e83-125">In addition to the process to manually bootstrap the NuGet provider, if the machine will be used to publish modules or scripts to a private gallery using the *Publish-Module* or *Publish-Script* cmdlets, the NuGet.exe binary executable file will be required.</span></span>
<span data-ttu-id="53e83-126">此方案的最常见用例是当专用库可用于支持独立环境时。</span><span class="sxs-lookup"><span data-stu-id="53e83-126">The most common use case for this scenario is when a private gallery is available to support an isolated environment.</span></span>
<span data-ttu-id="53e83-127">可通过两种方法获取 NuGet.exe 文件。</span><span class="sxs-lookup"><span data-stu-id="53e83-127">There are two options to obtain the NuGet.exe file.</span></span>

<span data-ttu-id="53e83-128">一种方法是启动已连接 Internet 的计算机，并使用受信任进程将文件复制到脱机计算机。</span><span class="sxs-lookup"><span data-stu-id="53e83-128">One option is to bootstrap a machine that is Internet connected and copy the files to the offline machines using a trusted process.</span></span>
<span data-ttu-id="53e83-129">启动已连接 Internet 的计算机后，NuGet.exe 二进制文件位于以下两个文件夹之一：</span><span class="sxs-lookup"><span data-stu-id="53e83-129">After bootstrapping the Internet connected machine, the NuGet.exe binary will be located in one of two folders:</span></span>

<span data-ttu-id="53e83-130">如果使用提升的权限（以管理员身份）执行 Publish-Module 或 Publish-Script cmdlet：</span><span class="sxs-lookup"><span data-stu-id="53e83-130">If the *Publish-Module* or *Publish-Script* cmdlets were executed with elevated permissions (As an Administrator):</span></span>

```
$env:ProgramData\Microsoft\Windows\PowerShell\PowerShellGet
```

<span data-ttu-id="53e83-131">如果不使用提升的权限（以用户身份）执行这两个 cmdlet：</span><span class="sxs-lookup"><span data-stu-id="53e83-131">If the cmdlets were executed as a user without elevated permissions:</span></span>

```
$env:userprofile\AppData\Local\Microsoft\Windows\PowerShell\PowerShellGet\
```

<span data-ttu-id="53e83-132">第二种方法是从 NuGet.Org 网站下载 NuGet.exe：[https://dist.nuget.org/index.html](https://dist.nuget.org/index.html)</span><span class="sxs-lookup"><span data-stu-id="53e83-132">A second option is to download NuGet.exe from the NuGet.Org website: [https://dist.nuget.org/index.html](https://dist.nuget.org/index.html)</span></span><br>
<span data-ttu-id="53e83-133">选择生产计算机的 NugGet 版本时，请确保版本高于 2.8.5.208，并确认版本是否已标记为“推荐”。</span><span class="sxs-lookup"><span data-stu-id="53e83-133">When selecting a NugGet version for production machines, make sure it is later than 2.8.5.208, and identify the version that has been labeled "recommended".</span></span>
<span data-ttu-id="53e83-134">如果使用浏览器下载，请务必取消阻止文件。</span><span class="sxs-lookup"><span data-stu-id="53e83-134">Remember to unblock the file if it was downloaded using a browser.</span></span>
<span data-ttu-id="53e83-135">为此，可以使用 Unblock-File cmdlet。</span><span class="sxs-lookup"><span data-stu-id="53e83-135">This can be performed by using the *Unblock-File* cmdlet.</span></span>

<span data-ttu-id="53e83-136">无论属于上述哪种情况，都可以将 NuGet.exe 文件复制到 $env:path 中的任意位置，但标准位置为：</span><span class="sxs-lookup"><span data-stu-id="53e83-136">In either case, the NuGet.exe file can be copied to any location in *$env:path*, but the standard locations are:</span></span>

<span data-ttu-id="53e83-137">若要让可执行文件可用，以便所有用户都可以使用 Publish-Module 和 Publish-Script cmdlet：</span><span class="sxs-lookup"><span data-stu-id="53e83-137">To make the executable available so that all users can use *Publish-Module* and *Publish-Script* cmdlets:</span></span>

```
$env:ProgramData\Microsoft\Windows\PowerShell\PowerShellGet
```

<span data-ttu-id="53e83-138">若要让可执行文件仅对特定用户可用，请仅将文件复制到相应用户配置文件内的位置：</span><span class="sxs-lookup"><span data-stu-id="53e83-138">To make the executable available to only a specific user, copy to the location within only that user's profile:</span></span>

```
$env:userprofile\AppData\Local\Microsoft\Windows\PowerShell\PowerShellGet\
```