
# Restful LDAP Manager

LDAP Server 操作用の<b>「実験的」</b>な REST Server。  
*エントリの登録・更新・削除をトリガーに CSV ファイルへエントリを出力  
*動作環境は Windows Server 2019  

1. [機能](/機能.md) 
1. [インストール](/インストール.md) 
1. [Windows サービス](/Windowsサービス.md) 
1. [実行方法](/実行方法.md) 
1. [REST API](https://aoyanagi.github.io/Restful-LDAP-Manager-Release/rest-api.html)
1. [変更履歴](/変更履歴.md) 

# 呼び出し方法

PowerShell
```PowerShell
try {
    # ログイン
    $json = @{
        "bind_dn"       = "cn=manager,dc=maybework,dc=local"
        "bind_password" = "password"
    } | ConvertTo-Json -Compress -Depth 10
    $body = [System.Text.Encoding]::UTF8.GetBytes($json)
    $response = invoke-restmethod -Uri "http://localhost:8080/v1/sessions" -Method Post -ContentType 'application/json'-Body $body
    $session_id = $response.session_id

    # エントリの検索
    $headers = @{ "SESSION-ID" = $session_id }
    $queryParameter = "?scope=subtree&filter=" + [System.Web.HttpUtility]::UrlEncode("(&(cn=10001)(objectClass=inetOrgPerson))")
    $response = invoke-restmethod -Uri "http://localhost:8080/v1/ldap/ou=people,dc=maybework,dc=local$queryParameter" -Method Get -ContentType 'application/json' -Headers $headers
    $response

    # ログアウト
    $headers = @{ "SESSION-ID" = $session_id }
    invoke-restmethod -Uri "http://localhost:8080/v1/sessions" -Method Delete -ContentType 'application/json' -Headers $headers > $null
}
catch {
    Write-Error ($_.Exception.Message)
}
```
出力結果
```
dn           : cn=10001,ou=people,dc=maybework,dc=local
displayname  : 山田 太郎
givenname    : taro
objectclass  : {top, inetOrgPerson}
userpassword : {SHA512}sQnzu7wkTrgkQZF+0G1hi5AI3Qmzvv0bXgc5THBqi7mAsdd4Xll27ASbRt9fEyavWi6m0QP9B8lThf+rDKy8hg==
cn           : 10001
sn           : yamada
title        : Manager
```

# マッピングによる LDAP Server の更新
設定ファイルに記述されたマッピングを評価して LDAP Server のエントリを登録・更新する。
リクエストの属性値をそのままエントリに設定する、または PowerShell の実行結果をエントリに設定する２パターン。
- エントリの登録時に REST API のリクエスト cn を LDAP Server のエントリ cn にそのまま設定を行う。
- エントリの登録・更新時に REST API のリクエスト sn と givenName の間にスペースを挿入して LDAP Server のエントリ displayName に設定を行う。
- エントリの登録・更新時に LDAP Server のエントリ description に現在日時の設定を行う。
- PowerShell は複数行の記述が可能

ServerConfig.xml
```XML
    <!-- マッピング一覧 -->
    <mapping_list>
        <mapping add="true" mod="false">
            <!-- リクエストの属性値をそのまま設定する場合は false、PowerShell による加工の場合は true を設定する。-->
            <formula>false</formula>

            <!-- リクエストの属性名または式 -->
            <source>cn</source>

            <!-- LDAP サーバーに設定する属性名 -->
            <target>cn</target>
        </mapping>

        <mapping add="true" mod="true">
            <!-- リクエストの属性値をそのまま設定する場合は false、PowerShell による加工の場合は true を設定する。-->
            <formula>true</formula>

            <!-- リクエストの属性名または式 -->
            <source>$sn + ' ' + $givenName</source>

            <!-- LDAP サーバーに設定する属性名 -->
            <target>displayName</target>
        </mapping>
        
        <mapping add="true" mod="true">
            <!-- リクエストの属性値をそのまま設定する場合は false、PowerShell による加工の場合は true を設定する。-->
            <formula>true</formula>

            <!-- リクエストの属性名または式 -->
            <source>(Get-Date).ToString('yyyy-MM-dd hh:mm:ss')</source>

            <!-- LDAP サーバーに設定する属性名 -->
            <target>description</target>
        </mapping>
    </mapping_list>
</config>
```