---
ms.date: 06/12/2017
keywords: dsc,powershell,配置,安装程序
title: 适用于 Linux 的 DSC nxScript 资源
ms.openlocfilehash: 339968512ab1c16c4c3785a3a19b00c3fbbf9ea1
ms.sourcegitcommit: e04292a9c10de9a8391d529b7f7aa3753b362dbe
ms.translationtype: MTE95
ms.contentlocale: zh-CN
ms.lasthandoff: 01/04/2019
ms.locfileid: "54047138"
---
# <a name="dsc-for-linux-nxscript-resource"></a><span data-ttu-id="36626-103">适用于 Linux 的 DSC nxScript 资源</span><span class="sxs-lookup"><span data-stu-id="36626-103">DSC for Linux nxScript Resource</span></span>

<span data-ttu-id="36626-104">PowerShell Desired State Configuration (DSC) 中的 **nxScript** 资源提供了在 Linux 节点上运行 Linux 脚本的机制。</span><span class="sxs-lookup"><span data-stu-id="36626-104">The **nxScript** resource in PowerShell Desired State Configuration (DSC) provides a mechanism to run Linux scripts on a Linux node.</span></span>

## <a name="syntax"></a><span data-ttu-id="36626-105">语法</span><span class="sxs-lookup"><span data-stu-id="36626-105">Syntax</span></span>

```
nxScript <string> #ResourceName
{
    GetScript = <string>
    SetScript = <string>
    TestScript = <string>
    [ User = <string> ]
    [ Group = <string> ]
    [ DependsOn = <string[]> ]

}
```

## <a name="properties"></a><span data-ttu-id="36626-106">“属性”</span><span class="sxs-lookup"><span data-stu-id="36626-106">Properties</span></span>

|  <span data-ttu-id="36626-107">属性</span><span class="sxs-lookup"><span data-stu-id="36626-107">Property</span></span> |  <span data-ttu-id="36626-108">说明</span><span class="sxs-lookup"><span data-stu-id="36626-108">Description</span></span> |
|---|---|
| <span data-ttu-id="36626-109">GetScript</span><span class="sxs-lookup"><span data-stu-id="36626-109">GetScript</span></span>| <span data-ttu-id="36626-110">提供调用 [Get-DscConfiguration](https://technet.microsoft.com/en-us/library/dn521625.aspx) cmdlet 时运行的脚本。</span><span class="sxs-lookup"><span data-stu-id="36626-110">Provides a script that runs when you invoke the [Get-DscConfiguration](https://technet.microsoft.com/en-us/library/dn521625.aspx) cmdlet.</span></span> <span data-ttu-id="36626-111">该脚本必须以 shebang 开头，如 #!/bin/bash。</span><span class="sxs-lookup"><span data-stu-id="36626-111">The script must begin with a shebang, such as #!/bin/bash.</span></span>|
| <span data-ttu-id="36626-112">SetScript</span><span class="sxs-lookup"><span data-stu-id="36626-112">SetScript</span></span>| <span data-ttu-id="36626-113">提供脚本。</span><span class="sxs-lookup"><span data-stu-id="36626-113">Provides a script.</span></span> <span data-ttu-id="36626-114">调用 [Start-DscConfiguration](https://technet.microsoft.com/en-us/library/dn521623.aspx) cmdlet 时，将首先运行 **TestScript**。</span><span class="sxs-lookup"><span data-stu-id="36626-114">When you invoke the [Start-DscConfiguration](https://technet.microsoft.com/en-us/library/dn521623.aspx) cmdlet, the **TestScript** runs first.</span></span> <span data-ttu-id="36626-115">如果 **TestScript** 块返回非 0 退出代码，则将运行 **SetScript** 块。</span><span class="sxs-lookup"><span data-stu-id="36626-115">If the **TestScript** block returns an exit code other than 0, the **SetScript** block will run.</span></span> <span data-ttu-id="36626-116">如果 **TestScript** 返回退出代码 0，则将不运行 **SetScript**。</span><span class="sxs-lookup"><span data-stu-id="36626-116">If the **TestScript** returns an exit code of 0, the **SetScript** will not run.</span></span> <span data-ttu-id="36626-117">该脚本必须以 shebang 开头，如 `#!/bin/bash`。</span><span class="sxs-lookup"><span data-stu-id="36626-117">The script must begin with a shebang, such as `#!/bin/bash`.</span></span>|
| <span data-ttu-id="36626-118">TestScript</span><span class="sxs-lookup"><span data-stu-id="36626-118">TestScript</span></span>| <span data-ttu-id="36626-119">提供脚本。</span><span class="sxs-lookup"><span data-stu-id="36626-119">Provides a script.</span></span> <span data-ttu-id="36626-120">调用 [Start-DscConfiguration](https://technet.microsoft.com/en-us/library/dn521623.aspx) cmdlet 时，将运行此脚本。</span><span class="sxs-lookup"><span data-stu-id="36626-120">When you invoke the [Start-DscConfiguration](https://technet.microsoft.com/en-us/library/dn521623.aspx) cmdlet, this script runs.</span></span> <span data-ttu-id="36626-121">如果它返回非 0 退出代码，则将运行 SetScript。</span><span class="sxs-lookup"><span data-stu-id="36626-121">If it returns an exit code other than 0, the SetScript will run.</span></span> <span data-ttu-id="36626-122">如果它返回退出代码 0，则将不运行 **SetScript**。</span><span class="sxs-lookup"><span data-stu-id="36626-122">If it returns an exit code of 0, the **SetScript** will not run.</span></span> <span data-ttu-id="36626-123">调用 [Test-DscConfiguration](https://technet.microsoft.com/en-us/library/dn407382.aspx) cmdlet 时，也会运行 **TestScript**。</span><span class="sxs-lookup"><span data-stu-id="36626-123">The **TestScript** also runs when you invoke the [Test-DscConfiguration](https://technet.microsoft.com/en-us/library/dn407382.aspx) cmdlet.</span></span> <span data-ttu-id="36626-124">但是，在这种情况下，无论从 **TestScript** 返回何种退出代码，都不会运行 **SetScript**。</span><span class="sxs-lookup"><span data-stu-id="36626-124">However, in this case, the **SetScript** will not run, no matter what exit code is returned from the **TestScript**.</span></span> <span data-ttu-id="36626-125">如果实际配置与当前所需状态配置相匹配，则 **TestScript** 必定返回退出代码 0；如两者不匹配，则返回非 0 退出代码。</span><span class="sxs-lookup"><span data-stu-id="36626-125">The **TestScript** must return an exit code of 0 if the actual configuration matches the current desired state configuration, and an exit code other than 0 if it does not match.</span></span> <span data-ttu-id="36626-126">（当前所需状态配置是在使用 DSC 的节点上执行的最后一个配置）。</span><span class="sxs-lookup"><span data-stu-id="36626-126">(The current desired state configuration is the last configuration enacted on the node that is using DSC).</span></span> <span data-ttu-id="36626-127">该脚本必须以 shebang 开头，如 1#!/bin/bash.1</span><span class="sxs-lookup"><span data-stu-id="36626-127">The script must begin with a shebang, such as 1#!/bin/bash.1</span></span>|
| <span data-ttu-id="36626-128">用户</span><span class="sxs-lookup"><span data-stu-id="36626-128">User</span></span>| <span data-ttu-id="36626-129">将该脚本作为此用户运行。</span><span class="sxs-lookup"><span data-stu-id="36626-129">The user to run the script as.</span></span>|
| <span data-ttu-id="36626-130">组</span><span class="sxs-lookup"><span data-stu-id="36626-130">Group</span></span>| <span data-ttu-id="36626-131">将该脚本作为此组运行。</span><span class="sxs-lookup"><span data-stu-id="36626-131">The group to run the script as.</span></span>|
| <span data-ttu-id="36626-132">DependsOn</span><span class="sxs-lookup"><span data-stu-id="36626-132">DependsOn</span></span> | <span data-ttu-id="36626-133">指示必须先运行其他资源的配置，再配置此资源。</span><span class="sxs-lookup"><span data-stu-id="36626-133">Indicates that the configuration of another resource must run before this resource is configured.</span></span> <span data-ttu-id="36626-134">例如，如果你想要首先运行 **ID** 为 **ResourceName**、类型为 **ResourceType** 的资源配置脚本块，则使用此属性的语法为 `DependsOn = "[ResourceType]ResourceName"`。</span><span class="sxs-lookup"><span data-stu-id="36626-134">For example, if the **ID** of the resource configuration script block that you want to run first is **ResourceName** and its type is **ResourceType**, the syntax for using this property is `DependsOn = "[ResourceType]ResourceName"`.</span></span>|

## <a name="example"></a><span data-ttu-id="36626-135">示例</span><span class="sxs-lookup"><span data-stu-id="36626-135">Example</span></span>

<span data-ttu-id="36626-136">以下示例演示了如何使用 **nxScript** 资源进行其他配置管理。</span><span class="sxs-lookup"><span data-stu-id="36626-136">The following example demonstrates the use of the **nxScript** resource to perform additional configuration management.</span></span>

```
Import-DSCResource -Module nx

Node $node {
nxScript KeepDirEmpty{

    GetScript = @"
#!/bin/bash
ls /tmp/mydir/ | wc -l
"@

    SetScript = @"
#!/bin/bash
rm -rf /tmp/mydir/*
"@

    TestScript = @'
#!/bin/bash
filecount=`ls /tmp/mydir | wc -l`
if [ $filecount -gt 0 ]
then
    exit 1
else
    exit 0
fi
'@
}
}
```