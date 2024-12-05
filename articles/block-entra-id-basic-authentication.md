---
title: "PowerShellを使ってEntraIDのベーシック認証をブロックし、セキュリティを強化する"
emoji: "🧱"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Windows","Security","PowerShell","EntraID","Office365"]
published: false
publication_name: "visasq"
---



この記事では、PowerShellを使ってEntraIDのベーシック認証（基本認証）をブロックし、組織のセキュリティを強化する方法を解説します。

## 免責事項
- 本記事は[Securing Office 365 with Okta](https://www.okta.com/resources/whitepaper/securing-office-365-with-okta/)を参考にしています。
- 主にPowerShellのコマンドが古く書き直しています。
- 間違いを見つけていただいた際には是非コメントをいただけると幸いです。

## ⚠️ 重要な注意事項
本記事の設定を適用する前に、以下の影響を必ず確認してください

1. Windowsサインインへの影響
    - ベーシック認証をすべてブロックするとWindowsサインインでパスワードサインインが利用できなくなります。
    - 代替手段として[Web Sign-In](https://learn.microsoft.com/en-us/windows/security/identity-protection/web-sign-in/?tabs=intune)の有効化を検討してください。

2. システムへの影響
    - レガシーアプリケーションが動作しなくなる可能性があります。
    - トークンのRevokeを実行すると、すべてのデバイスで再認証が必要になります。

## 認証方式について

### ベーシック認証とは
ベーシック認証とはユーザー名とパスワードのみを使用する従来型の認証方式です。
セキュリティ機能が限定的でMFAや条件付きアクセスに対応していません。

### モダン認証とは
モダン認証とはベーシック認証より新しい認証です。
多要素認証（MFA）をサポートし、条件付きアクセスポリシーとの連携ができます。

### なぜベーシック認証をブロックするのか
Microsoftはベーシック認証を非推奨とし、モダン認証の使用を推奨しています。


しかしながら執筆時点ではベーシック認証は、非推奨としながらも利用することができ、明示的にブロックしないと両方の認証を併用できる状態になっています。ベーシック認証はパスワードスプレー攻撃に弱く、MFAや条件付きアクセスを回避できてしまうリスクがあります。

### ベーシック認証が利用できるプロトコル（レガシー認証プロトコル）の状況
ベーシック認証が利用できてしまうレガシー認証プロトコルには次のようなものがあります。

| レガシー認証プロトコル | ベーシック認証 | モダン認証 |
| ---------------------- | -------------- | ---------- |
| POP                    | ○              | ☓          |
| IMAP                   | ○              | ☓          |
| Active Sync            | ○              | ○          |
| EWS                    | ○              | ○          |
| MAPI                   | ○              | ○          |
| PowerShell             | ○              | ○          |



## ベーシック認証、レガシー認証プロトコルをブロックする手順
### 1. モダン認証の有効化確認
まず、モダン認証が有効になっていることを確認します。

```powershell
# Exchange Onlineに接続
Connect-ExchangeOnline -UserPrincipalName {your@domain.com}

# 現在の設定を確認
Get-OrganizationConfig | Format-Table -Auto Name,OAuth*

# モダン認証が無効な場合は有効化
Set-OrganizationConfig -OAuth2ClientProfileEnabled $true
```

実行結果の例：
```powershell
Name                           OAuth2ClientProfileEnabled
----                           -------------------------
your@domain.com                True
```

### 2. レガシー認証プロトコルの無効化
特にPOPとIMAPは、モダン認証に対応していないため、必ず無効化が必要です。
不要ならばMAPI、ActiveSync、EWSもDisableにしておくとより安全です。

#### 2.1 現状確認
まず、現在の利用状況を確認します


```powershell
# 利用状況をCSVにエクスポート
$results = Get-EXOCasMailbox -ResultSize Unlimited | Select-Object DisplayName,
    PrimarySmtpAddress,
    PopEnabled,
    ImapEnabled,
    ActiveSyncEnabled,
    LastLogonTime

# デスクトップに保存
$desktop = [Environment]::GetFolderPath("Desktop")
$date = Get-Date -Format "yyyyMMdd"
$filepath = "$desktop\ProtocolEnabledUsers_$date.csv"
$results | Export-Csv -Path $filepath -NoTypeInformation -Encoding UTF8

Write-Host "Results exported to: $filepath" -ForegroundColor Green
```

#### 2.2 プロトコルの無効化
必要なプロトコルを無効化します。

```powershell
# ユーザー一覧の取得
Get-EXOCasMailbox | Select-Object Alias, PrimarySmtpAddress, UserPrincipalName

# 各プロトコルの無効化
Set-CASMailbox <Alias> -PopEnabled $False
Set-CASMailbox <Alias> -ImapEnabled $False
Set-CASMailbox <Alias> -MAPIEnabled $False    # オプション
Set-CASMailbox <Alias> -ActiveSyncEnabled $False  # オプション
Set-CASMailbox <Alias> -EWSEnabled $False    # オプション
```


一括で無効化する場合

```powershell
Get-EXOCasMailbox -ResultSize Unlimited |
    ForEach-Object {
        Write-Host "Disabling POP/IMAP for: $($_.PrimarySmtpAddress)"
        Set-CASMailbox $_.PrimarySmtpAddress -PopEnabled $False -ImapEnabled $False
    }
```

### 3. ベーシック認証をブロックする
ベーシック認証を組織全体でブロックします。
:::message alert
ベーシック認証をすべてブロックをするとWindowsサインインでパスワードサインインが利用できなくなります。
Webサインインを有効にするなど代替手段も併せて検討しましょう。
[Use Web Sign-In To Enable Passwordless Sign-In In Windows | Microsoft Learn](https://learn.microsoft.com/en-us/windows/security/identity-protection/web-sign-in/?tabs=intune)
:::


```powershell
# 認証ポリシーの作成
New-AuthenticationPolicy -Name "Block Basic Authentication"

# ポリシーの確認
Get-AuthenticationPolicy -Identity "Block Basic Authentication"

# ユーザーへの適用
Set-User -Identity <UserId> -AuthenticationPolicy "Block Basic Authentication"

# 即時適用（オプション）
Set-User -Identity <UserId> -STSRefreshTokensValidFrom $([System.DateTime]::UtcNow)

# 組織全体への適用
Set-OrganizationConfig -DefaultAuthenticationPolicy "Block Basic Authentication"
```


### 4. トークンの無効化（必要な場合）
不正アクセスの可能性がある場合は、トークンを無効化します。

#### 4.1 Graph APIへの接続
```powershell
Connect-MgGraph -Scopes "User.ReadWrite.All", "Directory.ReadWrite.All"
```

#### 4.2.A 個別ユーザーのトークン無効化
```powershell
$userId = "user@contoso.com"
$uri = "https://graph.microsoft.com/v1.0/users/$userId/revokeSignInSessions"
Invoke-MgGraphRequest -Method POST -Uri $uri
```

#### 4.2.B 全ユーザーのトークン無効化

```powershell
# Get All Users
$users = Get-MgUser -All

# Revoke token all user
foreach ($user in $users) {
    Write-Host "Revoking tokens for $($user.UserPrincipalName)..." -ForegroundColor Yellow
    $uri = "https://graph.microsoft.com/v1.0/users/$($user.Id)/revokeSignInSessions"
    try {
        Invoke-MgGraphRequest -Method POST -Uri $uri
        Write-Host "Success: $($user.UserPrincipalName)" -ForegroundColor Green
    } catch {
        Write-Host "Failed: $($user.UserPrincipalName) - $($_.Exception.Message)" -ForegroundColor Red
    }
}
```

#### 4.2.C 複数ユーザーのトークン無効化

次のような改行区切りでアドレスを記載したrevoke_account.txtを`C:\Temp\`に用意します。

```revoke_account.txt
akol@contoso.com
tjohnston@contoso.com
kakers@contoso.com
```

以下スクリプトを実行します
```powershell
# log
$logFile = "C:\Temp\RevokeSignIn_$(Get-Date -Format 'yyyyMMdd_HHmmss').log"
$errorLog = "C:\Temp\RevokeSignIn_Errors_$(Get-Date -Format 'yyyyMMdd_HHmmss').log"
$resultsCsv = "C:\Temp\RevokeSignIn_Results_$(Get-Date -Format 'yyyyMMdd_HHmmss').csv"

function Write-Log {
    param($Message)
    $logMessage = "$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss'): $Message"
    Write-Host $logMessage
    Add-Content -Path $logFile -Value $logMessage
}

try {
    # connect Graph API
    Write-Log "Connecting to Microsoft Graph..."
    Connect-MgGraph -Scopes "User.ReadWrite.All"

    $users = Get-Content "C:\Temp\RevokeSignIn\revoke_account.txt"
    Write-Log "Found $($users.Count) users in the file"

    # processMode
    $processMode = Read-Host @"
Select processing mode:
1: Process all users at once
2: Confirm each user individually
3: Preview users list only
Enter your choice (1-3)
"@

    switch ($processMode) {
        "3" {
            Write-Log "Preview mode selected. Users to be processed:"
            $users | ForEach-Object { Write-Log "- $_" }
            Write-Log "Preview completed. No actions taken."
            return
        }
        "2" { $confirmEach = $true }
        "1" { $confirmEach = $false }
        default {
            throw "Invalid choice. Script terminated."
        }
    }

    if (-not $confirmEach) {
        $confirm = Read-Host "Process all $($users.Count) users? (Y/N)"
        if ($confirm -ne "Y") {
            Write-Log "Operation cancelled by user"
            return
        }
    }

    $successCount = 0
    $errorCount = 0
    $results = @()

    foreach ($userId in $users) {
        $userId = $userId.Trim()
        $processed = $false
        $status = "Not Processed"
        $errorMessage = ""

        try {
            Write-Log "Processing user: $userId"

            if ($confirmEach) {
                $confirm = Read-Host "Process $userId? (Y/N)"
                if ($confirm -ne "Y") {
                    Write-Log "Skipped: $userId"
                    $status = "Skipped"
                    continue
                }
            }

            # Cancel a sign-in session
            $uri = "https://graph.microsoft.com/v1.0/users/$userId/revokeSignInSessions"
            Invoke-MgGraphRequest -Method POST -Uri $uri

            Write-Log "Successfully revoked sign-in sessions for: $userId"
            $successCount++
            $status = "Success"
            $processed = $true
        }
        catch {
            $errorCount++
            $errorMessage = $_.Exception.Message
            $status = "Failed"
            Write-Log "ERROR processing $userId : $errorMessage"
            Add-Content -Path $errorLog -Value "$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss'): $userId - $errorMessage"
        }
        finally {
            # Record Results
            $results += [PSCustomObject]@{
                UserEmail = $userId
                ProcessedTime = Get-Date -Format 'yyyy-MM-dd HH:mm:ss'
                Status = $status
                ErrorMessage = $errorMessage
                Processed = $processed
            }
        }
    }

    # Output results to CSV
    $results | Export-Csv -Path $resultsCsv -NoTypeInformation

    # Summary Output
    Write-Log "`n=== Summary ==="
    Write-Log "Total users found: $($users.Count)"
    Write-Log "Successfully processed: $successCount"
    Write-Log "Errors encountered: $errorCount"
    Write-Log "Detailed results saved to: $resultsCsv"

    # Notification if there is an error
    if ($errorCount -gt 0) {
        Write-Log "Error details can be found in: $errorLog"
    }
}
catch {
    Write-Log "CRITICAL ERROR: $($_.Exception.Message)"
}
finally {
    # Graph接続の切断
    Disconnect-MgGraph
    Write-Log "Disconnected from Microsoft Graph"
    Write-Log "Script completed. Check $logFile for full log"
}

# Display Results
if ($results) {
    Write-Log "`nProcessing Results:"
    $results | Format-Table UserEmail, Status, ProcessedTime -AutoSize
}
```


## 補足：Oktaを利用している場合の注意点
:::message
他のIdPでの挙動を確認していませんが、概ね同一の事象が発生するはずです
:::

Oktaをアイデンティティプロバイダーとして利用している場合、以下の点に注意が必要です。

- ベーシック認証使用時にOktaをバイパスしてサインインできる
- 意図しないロックアウトの可能性

### 意図しないロックアウトについて
ベーシック認証を用いた場合、Oktaの設定によってはサインインはできるが、認可要件を満たせていないためOログ上では失敗とみなされてロックアウトする場合があります。

例えばWindowsOSでパスワードによるサインインを複数回実行した場合にOktaがロックアウトします。

これを防ぐにはAuthentication Policyで`Windows-AzureAD-Authentication-Provider`の条件を組む必要があります。詳細は次のドキュメントをご確認ください
https://help.okta.com/oie/ja-jp/content/topics/apps/office365/hybrid-aad-joined-devices-support.htm


## 参考リンク
- [条件付きアクセスを使用してレガシ認証をブロックすることに関する記事 - Microsoft Entra ID | Microsoft Learn](https://learn.microsoft.com/ja-jp/entra/identity/conditional-access/policy-block-legacy-authentication)
- [Exchange Online で基本認証を無効にする | Microsoft Learn](https://learn.microsoft.com/ja-jp/exchange/clients-and-mobile-in-exchange-online/disable-basic-authentication-in-exchange-online)
- [Windows の Web サインイン | Microsoft Learn](https://learn.microsoft.com/ja-jp/windows/security/identity-protection/web-sign-in/?tabs=intune)
- [Enable or disable POP3, IMAP, MAPI, Outlook Web App or ActiveSync in Microsoft 365 - Exchange | Microsoft Learn](https://learn.microsoft.com/en-us/exchange/troubleshoot/user-and-shared-mailboxes/pop3-imap-owa-activesync-office-365)
