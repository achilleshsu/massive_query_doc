# Massive Query 新增的API #

###### 撰寫日期:2021/10/28 ######
###### 撰寫版本:minerva-0.0.3 ######
###### 文件撰寫人: achilles ######

### 新增 catalog API ###
#### API 路徑 ####
POST 服務路徑/v1/api/subscriptionId/{subscriptionId}/catalogs
#### 需要 header 整理 ####
| header 參數名稱 | 內容 | 說明 | 
| --- | --- | --- |
| Content-Type | application/json | |
| Authorization | Bearer {ssoToken} | sso 啟用時 |
| Authorization | Basic {base64 user:password} | 當sso沒有啟用時 |
| Cookie | EIToken={ssoToken},WISEUser={userName} | sso 啟用時 |
#### body 輸入內容範例 ####
iceberg
```dtd
{
  "catalog": "iceberg",
  "connector": "iceberg",
  "parameters": {
    "hive.metastore.uri": "thrift://172.17.21.204:9083",
    "hive.s3.path-style-access": "true",
    "hive.s3.endpoint": "http://172.22.23.161:8080",
    "hive.s3.aws-access-key": "XXXXXXXXXXXXXXXXXX",
    "hive.s3.aws-secret-key": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
    "hive.metastore.thrift.delete-files-on-drop": "true",
    "iceberg.max-partitions-per-writer": "10000"
  }
}
```
hana
```dtd
{
  "catalog": "hana",
  "connector": "hana",
  "parameters": {
    "connection-url":"jdbc:sap://172.20.3.81:36315",
    "connection-user":"XXXX",
    "connection-password":"XXXXXXXXXXX",
    "hana.include-system-tables":"true"
  }
}
```
postgresql
```dtd
{
  "catalog": "postgres",
  "connector": "postgresql",
  "parameters": {
    "connection-url":"jdbc:postgresql://61.219.26.55:5432/postgres",
    "connection-user":"XXXXXXXXXXX",
    "connection-password":"XXXXXXXXXXXXXXX"
  }
}
```
#### 收到回覆 ####
| 狀態馬 | 內容 | 說明 | 
| --- | --- | --- |
| 200 | OK | 正常建立 |
| 404 | Not Found | API路徑錯誤或服務器尚未啟動 |
| 401 | Unauthorized | 權限不對或是 token 已過期 |
| 500 | Internal server error | DB 連線錯誤或其他問題 |

### 刪除 catalog API ###
#### API 路徑 ####
DELETE 服務路徑/v1/api/subscriptionId/{subscriptionId}/catalogs/{catalogsName}
#### 需要 header 整理 ####
| header 參數名稱 | 內容 | 說明 | 
| --- | --- | --- |
| Content-Type | application/json | |
| Authorization | Bearer {ssoToken} | sso 啟用時 |
| Authorization | Basic {base64 user:password} | 當sso沒有啟用時 |
| Cookie | EIToken={ssoToken},WISEUser={userName} | sso 啟用時 |

#### 收到回覆 ####
| 狀態馬 | 內容 | 說明 | 
| --- | --- | --- |
| 200 | OK | 正常建立 |
| 404 | Not Found | API路徑錯誤或服務器尚未啟動 |
| 401 | Unauthorized | 權限不對或是 token 已過期 |
| 500 | Internal server error | DB 連線錯誤或其他問題 |

