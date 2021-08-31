
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
# ログイン
$json = @{
    "bind_dn"       = "cn=manager,dc=maybework,dc=local"
    "bind_password" = "password"
} | ConvertTo-Json -Compress -Depth 10
$body = [System.Text.Encoding]::UTF8.GetBytes($json)
$response = invoke-restmethod -Uri "http://localhost:8080/v1/sessions" -Method Post  -ContentType 'application/json'-Body $body
$session_id = $response.session_id

# エントリの検索
$headers = @{ "SESSION-ID" = $session_id }
$queryParameter = "?scope=subtree&filter=" + [System.Web.HttpUtility]::UrlEncode("(&(cn=10001)(objectClass=inetOrgPerson))")
$response = invoke-restmethod -Uri "http://localhost:8080/v1/ldap/ou=people,dc=maybework,dc=local$queryParameter" -Method Get  -ContentType 'application/json' -Headers $headers
$response

# ログアウト
$headers = @{ "SESSION-ID" = $session_id }
invoke-restmethod -Uri "http://localhost:8080/v1/sessions" -Method Delete -ContentType 'application/json' -Headers $headers > $null
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
