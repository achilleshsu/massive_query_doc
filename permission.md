# Permission 說明 #

###### 撰寫日期:2022/02/09 ######
###### 撰寫版本:minerva-0.1.0 ######
###### 文件撰寫人: achilles ######

---------------------------------------
## sso 前置參數說明 ##

佈署的時候我們是透過參數控制要不要啟動SSO認證，其參數如下表：

|參數名稱| 參數值(舉例) | 說明 |
|---|---|---|
| env.authtype | sso |  若為空值，則不採用SSO認證，採用trino原本方式 |
| env.sso_url | http://api-sso-ensaas.dev001.wise-paas.com/v4.0 | SSO 地址 |
| ssl.additionalConfigProperties.http-server.authentication.type | JWT,PASSWORD | Trion 的驗證方式，jwt 要寫才能判斷 token

在寫SSO權限驗證程式的時候，會必須先要有以下判斷，確保開關正常。
```
String authType = System.getenv("authtype");

if (authType == null || authType.length() == 0) {
    authType = "jwt";
}

if (authType.equals("jwt")) {
    //do original trino jwt do
} else if (authType.equals("sso")) {
    //chech permission by sso
} else {
    log.info("authType is:" + authType)
}
```
這幾行會先讀取環境變數，讀取後判斷是不是空值，若是空值，給予一個參數(如圖中jwt)
，後續用這個 authType 判斷應該執行甚麼流程。

---------------------------------------
## DB 內容一覽 ##

目前 Trino 相關認證權限會用 Postgresql 作為儲存機制(之前是 sqllite )，總共有下列表格。

#### 表格名稱: catalogs 用來記錄現階段的 catalog ####
|欄位名稱 | 說明 |
|---|---|
| name | catalog 名稱 |
| connector | connector 名稱 |
| parameters | 相關參數 |

表格名稱：connectors 用來給UI列表使用
|欄位名稱 | 說明 |
|---|---| 
| connector_name | connector 名稱 |
| display_name | 顯示名稱 |
| svg | 圖案 |
| type | 類別 |
| parameters | 相關參數 |
| default_parameters | Default 參數 |


表格名稱：groups 用來劃分使用者權限用
|欄位名稱 | 說明 |
|---|---|
| group_name | 群組名 |
| user_name | 歸類於這個群組的用戶 |

表格名稱：rules 
|欄位名稱 | 說明 |
|---|---|
| catalog_name | catalog 名稱 |
| subscription_id | 所屬的訂閱號ID(誰建立就是誰的) |

表格名稱：token_info
|欄位名稱 | 說明 |
|---|---|
| token | token 那一長串內容 |
| username | 用戶名稱 |
| expire_time | 過期時間，如果過期會刪除 |

---------------------------------------
## sso username,password 認證 ##

基本上前端通常都是使用 Cookie 帶上 EIToken 來操作 sso ，比較不會帶上 username 和 password 來和 minerva 做認證，但我們也有做這個功能。主要修改位置：
```
Minerva/plugin/trino-password-authenticators/src/main/java/io.trino.plugin.password/file/PasswordStore.java
```
在PasswordStore.java第88行，原先 trino 的判斷是這樣的，會驗證是否符合 Trino 文件設定好的帳號密碼。
```
return cache.getUnchecked(new Credential(user, password));
```
在 PasswordStore.java 中
1. 第 91-189 行，打 SSO API 驗證 username password 有沒有符合資格，若有打 API 失敗會進入 try catch 並回傳 401。
2. 第 122-139 行，打了 sso 的 /auth/native 驗證帳號密碼並且獲取token。
3. 第 143-185 原本會進一步驗證用戶的權限判斷，依照用戶的訂閱號 ID 進而給予可以看到的範圍，這邊註解起來的原因是：
    ### 組成 userMap 這個 HashMap 有困難，沒辦法順利取代 token 驗證那邊的 userMap ###
    這個問題會導致權限沒有辦法正常更新，會維持上一個帳號的權限，因此帳號密碼判斷這邊還需要研究

---------------------------------------
## sso token 認證 ##

前端登入的主要驗證方式，通過帶上 token 驗證，Trino 的佈署參數上要加上 ssl.additionalConfigProperties.http-server.authentication.type=JWT，此時 Trino 就會接受 token 的參數，一般使用 jdbc 連線，配置方法如下：

|參數名稱| 參數值(舉例) | 說明 |
|---|---|---|
| user | ssopassroot@email.com | 用戶的名稱 |
| accessToken | <EIToken> | 那一長串的 token 內容 |
| SSL | true | 啟動SSL |
| SSLVerification | NONE | SSL 的認證方式 |

JWT 驗證的程式位置位於
```
Minerva/core/trino-main/src/main/java/io.trino/server/security/jwt/JwtAuthenticator.java
```
第 84 行 extractPrincipalFromToken 的內容：
1. 第 87-94 行，做了是否使用 SSO 的判斷。
2. 第 100-104 行，先去讀取DB中如果有相同 token 的用戶，返回一個用該用戶做的 map，這個 map 會被拿去跟現在登入的用戶做判斷，若用戶名稱一致，則會放行，這裡讀的是 tokensService 得表。
3. 第 108-127 行，打了 SSO 的 /users/me 驗證 token 的正確性，並且將 response 記錄下來。
4. 第 128-131 行，失敗會傳空值的 username map，後面就會是 401
5. 第 132-155 行，解析 jwt 內容，獲取 username ，還有一些權限
6. 第 156-159 行，判斷是不是 globaladmin，如果是，userMap 放上 <global_admin,username>
7. 第 160-163 行，獲取 subscriptionRole。
8. 第 167-181 行，先去 DB 把現有的 catalog 名稱列出來，(P.S. 因為 Catalog 再購買的時候是需要帶上 subscriptionId 的，所以可以把所有 subscriptionId 拉出來做一個比對)，如果 subscriptionId 相同，繼而判斷他是 subscription admin 還是 user ，userMap 存上 <(catalogName)_admin,username> 或是 <(catalogName)_user,username> 
9. 第 187 行，這邊是另一個 function ，會和 DB 中的參數比較，多的會寫入，少的會刪除，這裡改的是 groupsService 的表
10. 第 192 行，通過驗證的會儲存到 DB 中，這裡儲存的是 tokensService 的表。

---------------------------------------
## sso Cookie 認證 ##

通常 UI 會是用 cookie 帶入認證，所以需要有個地方把 sso 特有的 cookie EIToken 解出來。
cookie 讀取的程式位置位於
```
Minerva/core/trino-main/src/main/java/io.trino/server/security/AbstractBearerAuthenticator.java
```
第 72 行 extractToken 的內容：
1. 第 76-79 行，是否使用 SSO 判斷。
2. 第 81-91 行，使用 Trino 原先的讀取方式。
3. 第 93-100 行，先對 header 取 AUTHORIZATION 的內容看看，看是不是可以拿到 token，如果無法，則為 null。
4. 第 102-117 行，嘗試找 cookie 裡面的 EIToken，並把 token 加上 bearer 做成 header 。
5. 第 120-132 行，採用了 Trino 的舊的 code ，會把參數 header 裡面的值轉換為 token 並回傳。

---------------------------------------
## 權限驗證map製作 ##

Trino 的權限管理可以用來限制某些 group 或是某些用戶只能看到某個 catalog ，他判斷權限機制的地方位於何處還不太曉得，但我製作權限管理的時候是參考他已存在的驗證機制來達成驗證方法。

#### 權限驗證map製作的程式位置位於
```
Minerva/lib/trino-plugin-toolkit/src/main/java/io.trino.plugin.base/security/FileBasedSystemAccessControl.java
```
Trino 既有的方法是透過 json mapping 把權限區隔開來，其實就是會在程式中組成一個 hashMap ，只要有辦法透過程式把這個 rule 的 hashMap 組好，就可以確保權限驗證沒有問題。

第 249 行 SystemAccessControl 得內容：
1. 第 254-257 行，是否使用 SSO 判斷。
2. 第 262-287 行，定義好最基本的 rule ，catalog 區塊 system 固定打開(dbeaver需要用)，預設 group:global_admin 和 user:admin 可以訪問全部 catalog、schema 和 table 的所有操作權限。
3. 第 284 行注意，凡是有 _user 的 group 都只能查詢，不能寫入。
4. 第 280 行，是要給 special 權限的用戶，可以把某個欄位遮蔽。
5. 第 288-292 行，把上面的 rule 轉為 jsonArray 比較好處理。
6. 第 294-314 行，讀取 DB 的內容，把相對應的權限加上去，注意：擁有者的 group 會屬於 catalogName_admin ，一般使用者的 group 會屬於 catalogName_user 
7. 第 316-332 行，將更新過後的內容，組合起來並做成 Map 往後傳遞下去 
8. 注意：這邊做權限判斷得時間點，就是用戶下指令使用DB得時候會最做的判斷，所以如果有新建的catalog，就會要在相對應的資料庫中，把 catalogName,subscriptionId 加上去，反之，如果有 catalog 刪除，就需要再對應的表格中將符合的 catalogName,subscriptionId 欄位刪除。

groups <-- 定義當前用戶屬於什麼 group，舉例：
```
global_admin:ssopassroot@email.com
iceberg_admin: demo-subadmin@test.com
iceberg_user: demo-subuser@test.com
mysql_admin: demo-othadmin@test.com
mysql_user: demo-othuser@test.com
```
rults <-- 定義 group 有什麼權限，舉例：
```
{
  "catalogs": [
    {
      "user": "admin",
      "allow": "all"
    },
    {
      "group": "global_admin",
      "allow": "all"
    },
    {
      "catalog": "system",
      "allow": "none"
    },
    {
      "group": "(iceberg_).*",
      "catalog": "iceberg",
      "allow": "all"
    },
    {
      "group": "(mysql_).*",
      "catalog": "mysql",
      "allow": "all"
    }
  ],
  "schemas": [
    {
      "group": "admin",
      "owner": true
    },
    {
      "group": "global_admin",
      "owner": true
    },
    {
      "group": "iceberg_admin",
      "catalog": "iceberg",
      "owner": true
    },
    {
      "group": "mysql_admin",
      "catalog": "mysql",
      "owner": true
    },
    {
      "owner": false
    }
  ],
    "tables": [
    {
      "group": "admin",
      "privileges": ["SELECT", "INSERT", "DELETE", "OWNERSHIP", "GRANT_SELECT"]
    },
    {
      "group": "global_admin",
      "privileges": ["SELECT", "INSERT", "DELETE", "OWNERSHIP", "GRANT_SELECT"]
    },
    {
      "group": "iceberg_admin",
      "privileges": ["SELECT", "INSERT", "DELETE", "OWNERSHIP", "GRANT_SELECT"]
    },
	{
	  "privileges": ["SELECT"]
	}
  ]
}

```