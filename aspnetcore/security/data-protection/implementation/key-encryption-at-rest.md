---
title: ASP.NET Core 中的待用金鑰加密
author: rick-anderson
description: 瞭解待用 ASP.NET Core 資料保護金鑰加密的執行詳細資料。
ms.author: riande
ms.date: 07/16/2018
uid: security/data-protection/implementation/key-encryption-at-rest
ms.openlocfilehash: 52c3137dbe467096364b42430c92aecc7c15e313
ms.sourcegitcommit: 9a129f5f3e31cc449742b164d5004894bfca90aa
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/06/2020
ms.locfileid: "78658386"
---
# <a name="key-encryption-at-rest-in-aspnet-core"></a>ASP.NET Core 中的待用金鑰加密

資料保護系統[預設會採用探索機制](xref:security/data-protection/configuration/default-settings)，以判斷密碼編譯金鑰在待用時的加密方式。 開發人員可以覆寫探索機制，並以手動方式指定如何在待用時加密金鑰。

> [!WARNING]
> 如果您指定明確的[金鑰持續性位置](xref:security/data-protection/implementation/key-storage-providers)，資料保護系統會取消註冊待用的預設金鑰加密機制。 因此，金鑰不會再加密。 我們建議您為生產環境部署[指定明確的金鑰加密機制](xref:security/data-protection/implementation/key-encryption-at-rest)。 本主題將說明待用加密機制選項。

::: moniker range=">= aspnetcore-2.1"

## <a name="azure-key-vault"></a>Azure 金鑰保存庫

若要將金鑰儲存在[Azure Key Vault](https://azure.microsoft.com/services/key-vault/)中，請在 `Startup` 類別中設定具有[ProtectKeysWithAzureKeyVault](/dotnet/api/microsoft.aspnetcore.dataprotection.azuredataprotectionbuilderextensions.protectkeyswithazurekeyvault)的系統：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .PersistKeysToAzureBlobStorage(new Uri("<blobUriWithSasToken>"))
        .ProtectKeysWithAzureKeyVault("<keyIdentifier>", "<clientId>", "<clientSecret>");
}
```

如需詳細資訊，請參閱[設定 ASP.NET Core 資料保護： ProtectKeysWithAzureKeyVault](xref:security/data-protection/configuration/overview#protectkeyswithazurekeyvault)。

::: moniker-end

## <a name="windows-dpapi"></a>Windows DPAPI

**僅適用于 Windows 部署。**

使用 Windows DPAPI 時，會使用[CryptProtectData](/windows/desktop/api/dpapi/nf-dpapi-cryptprotectdata)來加密金鑰材料，然後才保存到儲存體。 DPAPI 是一種適當的加密機制，適用于永遠不會在目前電腦之外讀取的資料（雖然可以將這些金鑰備份到 Active Directory，請參閱[DPAPI 和漫遊設定檔](https://support.microsoft.com/kb/309408/#6)）。 若要設定 DPAPI 靜態金鑰加密，請呼叫其中一個[ProtectKeysWithDpapi](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.protectkeyswithdpapi)擴充方法：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Only the local user account can decrypt the keys
    services.AddDataProtection()
        .ProtectKeysWithDpapi();
}
```

如果呼叫不含任何參數的 `ProtectKeysWithDpapi`，只有目前的 Windows 使用者帳戶可以解密保存的金鑰環。 您可以選擇性地指定電腦上的任何使用者帳戶（而不只是目前的使用者帳戶）能夠解密金鑰環：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // All user accounts on the machine can decrypt the keys
    services.AddDataProtection()
        .ProtectKeysWithDpapi(protectToLocalMachine: true);
}
```

::: moniker range=">= aspnetcore-2.0"

## <a name="x509-certificate"></a>X.509 憑證

如果應用程式分散在多部電腦上，在電腦上散發共用的 x.509 憑證，並將託管的應用程式設定為使用憑證來加密待用金鑰，可能會很方便：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .ProtectKeysWithCertificate("3BCE558E2AD3E0E34A7743EAB5AEA2A9BD2575A0");
}
```

由於 .NET Framework 限制，因此只支援具有 CAPI 私密金鑰的憑證。 如需這些限制的可能因應措施，請參閱下列內容。

::: moniker-end

## <a name="windows-dpapi-ng"></a>Windows DPAPI-NG

**此機制僅適用于 Windows 8/Windows Server 2012 或更新版本。**

從 Windows 8 開始，Windows 作業系統支援 DPAPI-NG （也稱為 CNG DPAPI）。 如需詳細資訊，請參閱[關於 CNG DPAPI](/windows/desktop/SecCNG/cng-dpapi)。

主體會編碼為保護描述項規則。 在下列呼叫[ProtectKeysWithDpapiNG](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.protectkeyswithdpaping)的範例中，只有具有指定 SID 的已加入網域使用者可以解密金鑰環：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Uses the descriptor rule "SID=S-1-5-21-..."
    services.AddDataProtection()
        .ProtectKeysWithDpapiNG("SID=S-1-5-21-...",
        flags: DpapiNGProtectionDescriptorFlags.None);
}
```

另外還有 `ProtectKeysWithDpapiNG`的無參數多載。 使用此便利方法來指定規則 "SID = {CURRENT_ACCOUNT_SID}"，其中*CURRENT_ACCOUNT_SID*是目前 Windows 使用者帳戶的 SID：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Use the descriptor rule "SID={current account SID}"
    services.AddDataProtection()
        .ProtectKeysWithDpapiNG();
}
```

在此案例中，AD 網域控制站負責散發 DPAPI NG 作業所使用的加密金鑰。 目標使用者可以從任何已加入網域的電腦解密已加密的內容（前提是該進程是在其身分識別下執行）。

## <a name="certificate-based-encryption-with-windows-dpapi-ng"></a>以憑證為基礎的加密與 Windows DPAPI-NG

如果應用程式是在 Windows 8.1/Windows Server 2012 R2 或更新版本上執行，您可以使用 Windows DPAPI-NG 來執行以憑證為基礎的加密。 使用規則描述元字串 "CERTIFICATE = HashId： THUMBPRINT"，其中*指紋*是憑證的十六進位編碼 SHA1 指紋：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .ProtectKeysWithDpapiNG("CERTIFICATE=HashId:3BCE558E2...B5AEA2A9BD2575A0",
            flags: DpapiNGProtectionDescriptorFlags.None);
}
```

任何指向此存放庫的應用程式都必須在 Windows 8.1/Windows Server 2012 R2 或更新版本上執行，才能解密金鑰。

## <a name="custom-key-encryption"></a>自訂金鑰加密

如果不適合使用內建機制，開發人員可以藉由提供自訂[IXmlEncryptor](/dotnet/api/microsoft.aspnetcore.dataprotection.xmlencryption.ixmlencryptor)來指定自己的金鑰加密機制。
