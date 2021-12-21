# GitHubメモ書き

## httpsで複数アカウントを使い分ける

### 参考  

https://docs.github.com/en/get-started/getting-started-with-git/caching-your-github-credentials-in-git

https://qiita.com/shiena/items/fc7783a82d59be5ff259

### 前提

GitHubはSSHではなくHTTPSを推奨している。

### インストール

Git Credential Managerが必要。  
Git for Windowsをインストールする際、Git Credential Managerもあわせて入れる。  

* メインアカウント

普通にVSCodeでgit cloneすればWindowsの資格情報に情報が作成される。

* サブアカウント

任意フォルダでterminalを開き下記実行。  

```console
> git -c credential.namespace=sub clone https://github.com/[name]/[repository].git
```

namespace=sub でWindowsの資格情報にメインとは別で作成される。  

```console
> git config credential.namespace sub
```

必要に応じて以下も実行。

```console
> git config --local user.name [your username]
> git config --local user.email [your email]
```