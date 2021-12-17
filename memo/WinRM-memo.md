# WinRM-memo

## Windows Sandboxで検証する方法

1. Windows Sandbox側
    1. Powershellを立ち上げる
    1. 以下実行

        ```PowerShell
        > Enable-PSRemoting
        ```

    1. Windows DefenderでInboudのグループ名="Windows リモート管理"をすべて有効化する
    1. コマンドプロンプトでWDAGUtilityAccountアカウントのパスワードを適当に変更する

        ```Batchfile
        net user WDAGUtilityAccount hoge
        ```

    1. IPを確認

1. localhost側
    1. WinRMが有効になっていることを確認
    1. 管理者モードでPowershellを立ち上げる
    1. Windows Sandbox側のport 5985にアクセスできることを確認
    1. 以下実行

        ```PowerShell
        > set-item wsman:localhost\client\trustedhosts -value *
        > Invoke-Command  -ComputerName "Windows Sandbox側のIPアドレス" -Credential (Get-Credential) -ScriptBlock {echo hoge}
        ```

1. 後始末（localhost側）
    1. 以下実行しエントリが残っているか確認

        ```PowerShell
        > get-item wsman:localhost\client\trustedhosts
        ```

    1. エントリが存在する場合は以下実行

        ```PowerShell
        > Clear-Item WSMan:\localhost\Client\TrustedHosts
        ```

    1. WinRM無効化
        
        ```PowerShell
        > Disable-PSRemoting
        ```
    
    1. 必要に応じて

        * Windows Defenderの"Windows リモート管理"を無効化
        * WinRMサービスを停止
        * ...など

