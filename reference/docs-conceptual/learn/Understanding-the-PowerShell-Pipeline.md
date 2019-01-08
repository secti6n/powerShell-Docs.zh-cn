---
ms.date: 08/23/2018
keywords: powershell,cmdlet
title: 了解 PowerShell 管道
ms.assetid: 6be50926-7943-4ef7-9499-4490d72a63fb
ms.openlocfilehash: fc7c7f57bdce458185a0f5bdb8bc1fbbd81d0d61
ms.sourcegitcommit: 00ff76d7d9414fe585c04740b739b9cf14d711e1
ms.translationtype: MTE95
ms.contentlocale: zh-CN
ms.lasthandoff: 12/14/2018
ms.locfileid: "53400388"
---
# <a name="understanding-pipelines"></a><span data-ttu-id="b35b2-103">了解管道</span><span class="sxs-lookup"><span data-stu-id="b35b2-103">Understanding pipelines</span></span>

<span data-ttu-id="b35b2-104">管道的行为就像一系列连接的管道段一样。</span><span class="sxs-lookup"><span data-stu-id="b35b2-104">Pipelines act like a series of connected segments of pipe.</span></span> <span data-ttu-id="b35b2-105">沿着管道移动的项会通过每个管道段。</span><span class="sxs-lookup"><span data-stu-id="b35b2-105">Items moving along the pipeline pass through each segment.</span></span> <span data-ttu-id="b35b2-106">若要在 PowerShell 中创建管道，请使用管道运算符“|”将命令连接在一起。</span><span class="sxs-lookup"><span data-stu-id="b35b2-106">To create a pipeline in PowerShell, you connect commands together with the pipe operator "|".</span></span> <span data-ttu-id="b35b2-107">每个命令的输出都将被用作下一命令的输入。</span><span class="sxs-lookup"><span data-stu-id="b35b2-107">The output of each command is used as input to the next command.</span></span>

<span data-ttu-id="b35b2-108">用于管道的符号类似于其他 shell 中使用的符号。</span><span class="sxs-lookup"><span data-stu-id="b35b2-108">The notation used for pipelines is similar to the notation used in other shells.</span></span> <span data-ttu-id="b35b2-109">初看起来 PowerShell 中管道的不同之处可能并不明显。</span><span class="sxs-lookup"><span data-stu-id="b35b2-109">At first glance, it may not be apparent how pipelines are different in PowerShell.</span></span> <span data-ttu-id="b35b2-110">尽管你会在屏幕上看到文本，但 PowerShell 通过管道在命令之间传递对象，而不是文本。</span><span class="sxs-lookup"><span data-stu-id="b35b2-110">Although you see text on the screen, PowerShell pipes objects, not text, between commands.</span></span>

## <a name="the-powershell-pipeline"></a><span data-ttu-id="b35b2-111">PowerShell 管道</span><span class="sxs-lookup"><span data-stu-id="b35b2-111">The PowerShell pipeline</span></span>

<span data-ttu-id="b35b2-112">管道可能是命令行界面中使用的最有价值的概念。</span><span class="sxs-lookup"><span data-stu-id="b35b2-112">Pipelines are arguably the most valuable concept used in command-line interfaces.</span></span> <span data-ttu-id="b35b2-113">如果使用得当，管道可以减少使用复杂命令的工作量，并且可以更轻松地查看命令的工作流程。</span><span class="sxs-lookup"><span data-stu-id="b35b2-113">When used properly, pipelines reduce the effort of using complex commands and make it easier to see the flow of work for the commands.</span></span> <span data-ttu-id="b35b2-114">管道中的每个命令（称为管道元素）将其输出逐项传递到管道中的下一个命令。</span><span class="sxs-lookup"><span data-stu-id="b35b2-114">Each command in a pipeline (called a pipeline element) passes its output to the next command in the pipeline, item-by-item.</span></span> <span data-ttu-id="b35b2-115">命令不必一次处理多个项目。</span><span class="sxs-lookup"><span data-stu-id="b35b2-115">Commands don't have to handle more than one item at a time.</span></span> <span data-ttu-id="b35b2-116">结果是减少了资源消耗，并且能够立即开始获取输出。</span><span class="sxs-lookup"><span data-stu-id="b35b2-116">The result is reduced resource consumption and the ability to begin getting the output immediately.</span></span>

<span data-ttu-id="b35b2-117">例如，如果使用 `Out-Host` cmdlet 来强制逐页显示来自于另一个命令的输出，那么这一输出看起来就像分页显示在屏幕上的普通文本：</span><span class="sxs-lookup"><span data-stu-id="b35b2-117">For example, if you use the `Out-Host` cmdlet to force a page-by-page display of output from another command, the output looks just like the normal text displayed on the screen, broken up into pages:</span></span>

```powershell
Get-ChildItem -Path C:\WINDOWS\System32 | Out-Host -Paging
```

```Output
    Directory: C:\WINDOWS\system32

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        4/12/2018   2:15 AM                0409
d-----        5/13/2018  11:31 PM                1033
d-----        4/11/2018   4:38 PM                AdvancedInstallers
d-----        5/13/2018  11:13 PM                af-ZA
d-----        5/13/2018  11:13 PM                am-et
d-----        4/11/2018   4:38 PM                AppLocker
d-----        5/13/2018  11:31 PM                appmgmt
d-----        7/11/2018   2:05 AM                appraiser
d---s-        4/12/2018   2:20 AM                AppV
d-----        5/13/2018  11:10 PM                ar-SA
d-----        5/13/2018  11:13 PM                as-IN
d-----        8/14/2018   9:03 PM                az-Latn-AZ
d-----        5/13/2018  11:13 PM                be-BY
d-----        5/13/2018  11:10 PM                BestPractices
d-----        5/13/2018  11:10 PM                bg-BG
d-----        5/13/2018  11:13 PM                bn-BD
d-----        5/13/2018  11:13 PM                bn-IN
d-----        8/14/2018   9:03 PM                Boot
d-----        8/14/2018   9:03 PM                bs-Latn-BA
d-----        4/11/2018   4:38 PM                Bthprops
d-----        4/12/2018   2:19 AM                ca-ES
d-----        8/14/2018   9:03 PM                ca-ES-valencia
d-----        5/13/2018  10:46 PM                CatRoot
d-----        8/23/2018   5:07 PM                catroot2
<SPACE> next page; <CR> next line; Q quit
...
```

<span data-ttu-id="b35b2-118">分页还会降低 CPU 利用率，因为准备好显示完整页面时，会转为处理 `Out-Host` cmdlet。</span><span class="sxs-lookup"><span data-stu-id="b35b2-118">Paging also reduces CPU utilization because processing transfers to the `Out-Host` cmdlet when it has a complete page ready to display.</span></span> <span data-ttu-id="b35b2-119">管道中位于前面的 cmdlet 暂停执行，直到输出的下一页可用。</span><span class="sxs-lookup"><span data-stu-id="b35b2-119">The cmdlets that precede it in the pipeline pause execution until the next page of output is available.</span></span>

<span data-ttu-id="b35b2-120">可以看到 Windows 任务管理器监视的 PowerShell 使用的 CPU 和内存存在差异。</span><span class="sxs-lookup"><span data-stu-id="b35b2-120">You can see the difference Windows Task Manager to monitor CPU and memory used by PowerShell.</span></span> <span data-ttu-id="b35b2-121">运行以下命令：`Get-ChildItem C:\Windows -Recurse`。</span><span class="sxs-lookup"><span data-stu-id="b35b2-121">Run the following command: `Get-ChildItem C:\Windows -Recurse`.</span></span> <span data-ttu-id="b35b2-122">将 CPU 和内存使用情况与此命令进行比较：`Get-ChildItem C:\Windows -Recurse | Out-Host -Paging`。</span><span class="sxs-lookup"><span data-stu-id="b35b2-122">Compare the CPU and memory usage to this command: `Get-ChildItem C:\Windows -Recurse | Out-Host -Paging`.</span></span>

## <a name="objects-in-the-pipeline"></a><span data-ttu-id="b35b2-123">管道中的对象</span><span class="sxs-lookup"><span data-stu-id="b35b2-123">Objects in the pipeline</span></span>

<span data-ttu-id="b35b2-124">在 PowerShell 中运行 cmdlet 时，可以看到文本输出，因为必须在控制台窗口中以文本形式表示对象。</span><span class="sxs-lookup"><span data-stu-id="b35b2-124">When you run a cmdlet in PowerShell, you see text output because it is necessary to represent objects as text in a console window.</span></span> <span data-ttu-id="b35b2-125">文本输出可能不会显示输出的对象的所有属性。</span><span class="sxs-lookup"><span data-stu-id="b35b2-125">The text output may not display all of the properties of the object being output.</span></span>

<span data-ttu-id="b35b2-126">例如，请考虑 `Get-Location` cmdlet。</span><span class="sxs-lookup"><span data-stu-id="b35b2-126">For example, consider the `Get-Location` cmdlet.</span></span> <span data-ttu-id="b35b2-127">如果运行 `Get-Location`，而当前位置是 C 驱动器的根路径，将看到以下输出：</span><span class="sxs-lookup"><span data-stu-id="b35b2-127">If you run `Get-Location` while your current location is the root of the C drive, you see the following output:</span></span>

```
PS> Get-Location

Path
----
C:\
```

<span data-ttu-id="b35b2-128">文本输出是信息摘要，而非 `Get-Location` 返回的对象的完整表示形式。</span><span class="sxs-lookup"><span data-stu-id="b35b2-128">The text output is a summary of information, not a complete representation of the object returned by `Get-Location`.</span></span> <span data-ttu-id="b35b2-129">输出中的标题通过格式化屏幕显示数据的过程添加。</span><span class="sxs-lookup"><span data-stu-id="b35b2-129">The heading in the output is added by the process that formats the data for onscreen display.</span></span>

<span data-ttu-id="b35b2-130">通过管道将输出传递到 `Get-Member` cmdlet 后，可以获取有关 `Get-Location` 返回的对象信息。</span><span class="sxs-lookup"><span data-stu-id="b35b2-130">When you pipe the output to the `Get-Member` cmdlet you get information about the object returned by `Get-Location`.</span></span>

```powershell
PS> Get-Location | Get-Member
```

```Output
   TypeName: System.Management.Automation.PathInfo

Name         MemberType Definition
----         ---------- ----------
Equals       Method     bool Equals(System.Object obj)
GetHashCode  Method     int GetHashCode()
GetType      Method     type GetType()
ToString     Method     string ToString()
Drive        Property   System.Management.Automation.PSDriveInfo Drive {get;}
Path         Property   string Path {get;}
Provider     Property   System.Management.Automation.ProviderInfo Provider {get;}
ProviderPath Property   string ProviderPath {get;}
```

<span data-ttu-id="b35b2-131">`Get-Location` 返回 PathInfo 对象，其中包含当前路径和其他信息。</span><span class="sxs-lookup"><span data-stu-id="b35b2-131">`Get-Location` returns a **PathInfo** object that contains the current path and other information.</span></span>