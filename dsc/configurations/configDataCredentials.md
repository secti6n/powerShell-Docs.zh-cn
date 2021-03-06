---
ms.date: 06/12/2017
keywords: dsc,powershell,配置,安装程序
title: 配置数据中的凭据选项
ms.openlocfilehash: 10cf3456fcc7104b7dd779db30aebace54ba087a
ms.sourcegitcommit: e04292a9c10de9a8391d529b7f7aa3753b362dbe
ms.translationtype: MTE95
ms.contentlocale: zh-CN
ms.lasthandoff: 01/04/2019
ms.locfileid: "54046635"
---
# <a name="credentials-options-in-configuration-data"></a>配置数据中的凭据选项
>适用于：Windows PowerShell 5.0

## <a name="plain-text-passwords-and-domain-users"></a>纯文本密码和域用户

包含凭据但未加密的 DSC 配置会生成一条关于纯文本密码的错误信息。
此外，使用域凭据时 DSC 会生成一个警告。
要禁止显示这些错误和警告信息，可以使用 DSC 配置数据关键字：
* **PsDscAllowPlainTextPassword**
* **PsDscAllowDomainUser**

> [!NOTE]
> 存储/传输未加密的纯文本密码通常是不安全的。 建议使用此主题后面部分讨论的方法保护凭据。
> Azure Automation DSC 服务可用于集中管理要在配置中编译并安全存储的凭据。
> 有关详细信息，请参阅：[编译 DSC 配置 / 凭据资产](/azure/automation/automation-dsc-compile#credential-assets)

下面是关于传递纯文本凭据的示例：

```powershell
#Prompt user for their credentials
#credentials will be unencrypted in the MOF
$promptedCreds = get-credential -Message "Please enter your credentials to generate a DSC MOF:"

# Store passwords in plaintext, in the document itself
# will also be stored in plaintext in the mof
$password = "ThisIsAPlaintextPassword" | ConvertTo-SecureString -asPlainText -Force
$username = "User1"
[PSCredential] $credential = New-Object System.Management.Automation.PSCredential($username,$password)

# DSC requires explicit confirmation before storing passwords insecurely
$ConfigurationData = @{
    AllNodes = @(
            @{
                # The "*" means "all nodes named in ConfigData" so we don't have to repeat ourselves
                NodeName="*"
                PSDscAllowPlainTextPassword = $true
            },
            #however, each node still needs to be explicitly defined for "*" to have meaning
            @{
                NodeName = "TestMachine1"
            },
            #we can also use a property to define node-specific passwords, although this is no more secure
            @{
                NodeName = "TestMachine2";
                UserName = "User2"
                LocalPassword = "ThisIsYetAnotherPlaintextPassword"
            }
        )
}

configuration unencryptedPasswordDemo
{
    Node "TestMachine1"
    {
        # We use the plaintext password to generate a new account
        User User1
        {
            UserName = $username
            Password = $credential
            Description = "local account"
            Ensure = "Present"
            Disabled = $false
            PasswordNeverExpires = $true
            PasswordChangeRequired = $false
        }
        # We use the prompted password to add this account to the local admins group
        Group addToAdmin
        {
            # Ensure the user exists before we add the user to a group
            DependsOn = "[User]User1"
            Credential = $promptedCreds
            GroupName = "Administrators"
            Ensure = "Present"
            MembersToInclude = "User1"
        }
    }

    Node "TestMachine2"
    {
        # Now we'll use a node-specific password to this machine
        $password = $Node.LocalPassword | ConvertTo-SecureString -asPlainText -Force
        $username = $node.UserName
        [PSCredential] $nodeCred = New-Object System.Management.Automation.PSCredential($username,$password)

        User User2
        {
            UserName = $username
            Password = $nodeCred
            Description = "local account"
            Ensure = "Present"
            Disabled = $false
            PasswordNeverExpires = $true
            PasswordChangeRequired = $false
        }

        Group addToAdmin
        {
            Credential = $promptedCreds
            GroupName = "Administrators"
            DependsOn = "[User]User2"
            Ensure = "Present"
            MembersToInclude = "User2"
        }
    }
}

# We declared the ConfigurationData in a local variable, but we need to pass it
# in to our configuration function
# We need to invoke the configuration function we created to generate a MOF
unencryptedPasswordDemo -ConfigurationData $ConfigurationData

# We need to pass the MOF to the machines we named.
#-wait: doesn't use jobs so we get blocked at the prompt until the configuration is done
#-verbose: so we can see what's going on and catch any errors
#-force: for testing purposes, I run start-dscconfiguration frequently + want to make sure i'm
#        not blocked by previous configurations that are still running
Start-DscConfiguration ./unencryptedPasswordDemo -verbose -wait -force
```

这是一段摘录"TestMachine1"的配置生成的".mof"文件。 `System.Security.SecureString`中使用配置被转换为纯文本，且存储在".mof"文件作为`MSF_Credential`。 一个`SecureString`使用当前用户配置文件进行加密。 这适用于 PowerShell 远程管理的所有窗体。 ".Mof"文件被旨在作为一个独立单独配置机制。 从 PowerShell 5.0 开始，在节点上的".mof"文件进行加密静态，但不是在传输到的节点。 这意味着".mof"文件中的密码会以明文形式公开，当应用于节点。 若要加密的凭据，您需要使用**拉服务器**。 有关详细信息，请参阅[使用证书保护 MOF 文件](../pull-server/secureMOF.md)。

```syntax
instance of MSFT_Credential as $MSFT_Credential1ref
{
Password = "ThisIsYetAnotherPlaintextPassword";
 UserName = "User2";

};

instance of MSFT_UserResource as $MSFT_UserResource1ref
{
ResourceID = "[User]User2";
 Description = "local account";
 UserName = "User2";
 Ensure = "Present";
 Password = $MSFT_Credential1ref;
 Disabled = False;
 SourceInfo = "::66::9::User";
 PasswordNeverExpires = True;
 ModuleName = "PsDesiredStateConfiguration";
 PasswordChangeRequired = False;

ModuleVersion = "1.0";

 ConfigurationName = "unencryptedPasswordDemo";

};
```

## <a name="handling-credentials-in-dsc"></a>在 DSC 中处理凭据

默认情况下，DSC 配置资源以 `Local System` 运行。
但某些资源需要凭据，例如 `Package` 资源需要使用特定用户帐户安装软件。

早期的资源使用硬编码 `Credential` 属性名称来处理此问题。
WMF 5.0 为所有资源都添加了自动 `PsDscRunAsCredential` 属性。
若要了解如何使用 `PsDscRunAsCredential`，请参阅[使用用户凭据运行 DSC](runAsUser.md)。
较新的资源和自定义资源可以使用此自动属性，而不必为凭据创建其自身属性。

> [!NOTE]
> 由于某种原因，某些资源的设计将使用多个凭据，这些凭据有其自己的凭据属性。

使用 ISE 中的 Intellisense (`CTRL+SPACE`) 或 `Get-DscResource -Name ResourceName -Syntax` 来查找资源的可用凭据属性。

```powershell
PS C:\> Get-DscResource -Name Group -Syntax
Group [String] #ResourceName
{
    GroupName = [string]
    [Credential = [PSCredential]]
    [DependsOn = [string[]]]
    [Description = [string]]
    [Ensure = [string]{ Absent | Present }]
    [Members = [string[]]]
    [MembersToExclude = [string[]]]
    [MembersToInclude = [string[]]]
    [PsDscRunAsCredential = [PSCredential]]
}
```

此示例使用来自 `PSDesiredStateConfiguration` 内置 DSC 资源模块中的 [Group](../resources/resources.md) 资源。
它可以创建本地组并添加或删除成员。
它同时接受 `Credential` 属性和自动 `PsDscRunAsCredential` 属性。
但是该资源只使用 `Credential` 属性。

有关 `PsDscRunAsCredential` 属性的详细信息，请参阅[使用用户凭据运行 DSC](runAsUser.md)。

## <a name="example-the-group-resource-credential-property"></a>例如：Group 资源 Credential 属性

DSC 在 `Local System` 下运行，因此它已经有权更改本地用户和组。
如果添加的成员是本地帐户，则无需凭据。
如果 `Group` 资源要添加一个域帐户到本地组，则需要凭据。

不允许匿名查询 Active Directory。
`Group` 资源的 `Credential` 属性是用于查询 Active Directory 的域帐户。
大多数情况下这是个一般用户帐户，因为默认情况下用户可以*读取* Active Directory 中的大多数对象。

## <a name="example-configuration"></a>示例配置

下面的示例代码使用 DSC 来填充带有域用户的本地组：

```powershell
Configuration DomainCredentialExample
{
    param
    (
        [PSCredential] $DomainCredential
    )
    Import-DscResource -ModuleName PSDesiredStateConfiguration

    node localhost
    {
        Group DomainUserToLocalGroup
        {
            GroupName        = 'ApplicationAdmins'
            MembersToInclude = 'contoso\alice'
            Credential       = $DomainCredential
        }
    }
}

$cred = Get-Credential -UserName contoso\genericuser -Message "Password please"
DomainCredentialExample -DomainCredential $cred
```

此代码生成了错误和警告消息：

```
ConvertTo-MOFInstance : System.InvalidOperationException error processing
property 'Credential' OF TYPE 'Group': Converting and storing encrypted
passwords as plain text is not recommended. For more information on securing
credentials in MOF file, please refer to MSDN blog:
http://go.microsoft.com/fwlink/?LinkId=393729

At line:11 char:9
+   Group
At line:297 char:16
+     $aliasId = ConvertTo-MOFInstance $keywordName $canonicalizedValue
+                ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (:) [Write-Error], InvalidOperationException
    + FullyQualifiedErrorId : FailToProcessProperty,ConvertTo-MOFInstance

WARNING: It is not recommended to use domain credential for node 'localhost'.
In order to suppress the warning, you can add a property named
'PSDscAllowDomainUser' with a value of $true to your DSC configuration data
for node 'localhost'.
```

此示例有两个问题：
1. 错误消息说明不推荐使用纯文本密码
2. 警告消息建议不要使用域凭据

## <a name="psdscallowplaintextpassword"></a>PsDscAllowPlainTextPassword

第一条错误消息具有一个带有文档的 URL。
此链接说明如何使用 [ConfigurationData](./configData.md) 结构和证书来加密密码。
有关证书和 DSC 的详细信息，[请阅读此文章](http://aka.ms/certs4dsc)。

要强制使用纯文本密码，资源需要下列配置数据部分中的 `PsDscAllowPlainTextPassword` 关键字：

```powershell
Configuration DomainCredentialExample
{
    param
    (
        [PSCredential] $DomainCredential
    )
    Import-DscResource -ModuleName PSDesiredStateConfiguration

    node localhost
    {
        Group DomainUserToLocalGroup
        {
            GroupName        = 'ApplicationAdmins'
            MembersToInclude = 'contoso\alice'
            Credential       = $DomainCredential
        }
    }
}

$cd = @{
    AllNodes = @(
        @{
            NodeName = 'localhost'
            PSDscAllowPlainTextPassword = $true
        }
    )
}

$cred = Get-Credential -UserName contoso\genericuser -Message "Password please"
DomainCredentialExample -DomainCredential $cred -ConfigurationData $cd
```

> [!NOTE]
> `NodeName` 不等同于星号，必须指定具体的节点名称。

**因为存在重大安全风险，Microsoft 建议避免使用纯文本密码。**

## <a name="domain-credentials"></a>域凭据

再次运行示例配置脚本（加密或不加密），仍然生成警告消息说不推荐将域帐户用于凭据。
使用本地帐户可消除泄露可用于其他服务器的域凭据的可能性。

**对 DSC 资源使用凭据时，应尽可能选择本地帐户而非域帐户。**

如果凭据的 `Username` 属性中有 \' 或 @，则 DSC 会将该凭据视为域帐户。
用户名中域部分的“localhost”、“127.0.0.1”和“::1”除外。

## <a name="psdscallowdomainuser"></a>PSDscAllowDomainUser

在上述 DSC `Group` 资源示例中，查询 Active Directory 域*必须使用*域帐户。
在这种情况下，将 `PSDscAllowDomainUser` 属性添加到 `ConfigurationData` 块中，操作如下：

```powershell
$cd = @{
    AllNodes = @(
        @{
            NodeName = 'localhost'
            PSDscAllowDomainUser = $true
            # PSDscAllowPlainTextPassword = $true
            CertificateFile = "C:\PublicKeys\server1.cer"
        }
    )
}
```

现在配置脚本生成的 MOF 文件将不再出现错误或警告消息。
