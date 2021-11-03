# Massive Query 新增的API #

###### 撰寫日期:2021/10/28 ######
###### 撰寫版本:minerva-0.0.5 ######
###### 文件撰寫人: achilles ######

---------------------------------------
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

#### 傳入路徑參數 ####
| 參數名稱 | 內容 | 說明 | 
| --- | --- | --- |
| subscriptionId | 訂閱號ID | 建立後會屬於該用戶，需要訂閱號admin 或global admin 才可以建立。 |

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

---------------------------------------
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

#### 傳入路徑參數 ####
| 參數名稱 | 內容 | 說明 | 
| --- | --- | --- |
| subscriptionId | 訂閱號ID | 需要建立時的訂閱號 admin 或 global admin 才可以刪除。 |

#### 收到回覆 ####
| 狀態馬 | 內容 | 說明 | 
| --- | --- | --- |
| 200 | OK | 正常建立 |
| 404 | Not Found | API路徑錯誤或服務器尚未啟動 |
| 401 | Invalid credentials | 權限不對或是 token 已過期 |
| 500 | Internal server error | DB 連線錯誤或其他問題 |

---------------------------------------

### 查詢 catalogs API ###
#### API 路徑 ####
GET 服務路徑/v1/api/subscriptionId/{subscriptionId}/catalogs

#### 需要 header 整理 ####
| header 參數名稱 | 內容 | 說明 | 
| --- | --- | --- |
| Content-Type | application/json | |
| Authorization | Bearer {ssoToken} | sso 啟用時 |
| Authorization | Basic {base64 user:password} | 當sso沒有啟用時 |
| Cookie | EIToken={ssoToken},WISEUser={userName} | sso 啟用時 |

#### 傳入路徑參數 ####
| 參數名稱 | 內容 | 說明 | 
| --- | --- | --- |
| subscriptionId | 訂閱號ID | 這邊其實不會判斷訂閱號ID，直接會從你登入的用戶判斷你的權限 |

#### 收到回覆 ####
| 狀態馬 | 內容 | 說明 | 
| --- | --- | --- |
| 200 | OK | 正常建立 |
| 404 | Not Found | API路徑錯誤或服務器尚未啟動 |
| 401 | Invalid credentials | 權限不對或是 token 已過期 |
| 500 | Internal server error | DB 連線錯誤或其他問題 |

#### 收到回覆範例 ####
```dtd
["iceberg","system"
        ]
```

---------------------------------------

### SQL API ###
#### API 路徑 ####
PUT 服務路徑/v1/api/execute

#### 需要 header 整理 ####
| header 參數名稱 | 內容 | 說明 | 
| --- | --- | --- |
| Content-Type | application/json | |
| Authorization | Bearer {ssoToken} | sso 啟用時 |
| Authorization | Basic {base64 user:password} | 當sso沒有啟用時 |
| Cookie | EIToken={ssoToken},WISEUser={userName} | sso 啟用時 |

#### body 輸入內容範例 ####
```dtd
show catalogs
```
注意不要有分號(;)
#### 收到回覆 ####
| 狀態馬 | 內容 | 說明 | 
| --- | --- | --- |
| 200 | OK | 正常建立 |
| 404 | Not Found | API路徑錯誤或服務器尚未啟動 |
| 401 | Invalid credentials | 權限不對或是 token 已過期 |
| 500 | Internal server error | DB 連線錯誤或其他問題 |

#### 收到回覆範例 ####
```dtd
[
        {
        "Catalog": "system"
        }
]
```

---------------------------------------

### 查詢 schemas (database) API ###
#### API 路徑 ####
GET 服務路徑/v1/api/catalogs/{catalogName}/schemas
#### 需要 header 整理 ####
| header 參數名稱 | 內容 | 說明 | 
| --- | --- | --- |
| Content-Type | application/json | |
| Authorization | Bearer {ssoToken} | sso 啟用時 |
| Authorization | Basic {base64 user:password} | 當sso沒有啟用時 |
| Cookie | EIToken={ssoToken},WISEUser={userName} | sso 啟用時 |

#### 傳入路徑參數 ####
| 參數名稱 | 內容 | 說明 | 
| --- | --- | --- |
| catalogName | catalog名稱 | 從catalog列表中列出來的名稱 |

#### 收到回覆 ####
| 狀態馬 | 內容 | 說明 | 
| --- | --- | --- |
| 200 | OK | 正常建立 |
| 404 | Not Found | API路徑錯誤或服務器尚未啟動 |
| 401 | Invalid credentials | 權限不對或是 token 已過期 |
| 500 | Internal server error | DB 連線錯誤或其他問題 |

#### 收到回覆範例 ####
```dtd
["achillestest","default","demo","demo2","hive_prod","hospital","hospital2","hospital5","hospital6","hospital7","information_schema","local","performance","spark_catalog","test","test2"
]
```

---------------------------------------

### 查詢 tables API ###
#### API 路徑 ####
GET 服務路徑/v1/api/catalogs/{catalogName}/schemas/{schemaName}/tables

#### 需要 header 整理 ####
| header 參數名稱 | 內容 | 說明 | 
| --- | --- | --- |
| Content-Type | application/json | |
| Authorization | Bearer {ssoToken} | sso 啟用時 |
| Authorization | Basic {base64 user:password} | 當sso沒有啟用時 |
| Cookie | EIToken={ssoToken},WISEUser={userName} | sso 啟用時 |

#### 傳入路徑參數 ####
| 參數名稱 | 內容 | 說明 | 
| --- | --- | --- |
| catalogName | catalog名稱 | 從catalog列表中列出來的名稱 |
| schemaName | schema(database)名稱 | 從schemas列表中列出來的名稱 |

#### 收到回覆 ####
| 狀態馬 | 內容 | 說明 | 
| --- | --- | --- |
| 200 | OK | 正常建立 |
| 404 | Not Found | API路徑錯誤或服務器尚未啟動 |
| 401 | Invalid credentials | 權限不對或是 token 已過期 |
| 500 | Internal server error | DB 連線錯誤或其他問題 |

#### 收到回覆範例 ####
```dtd
["cchtest","sstest","table1","test","test1","test22"
]
```

---------------------------------------

### 查詢 columns API ###
#### API 路徑 ####
GET 服務路徑/v1/api/catalogs/{catalogName}/schemas/{schemaName}/tables/{tableName}/columns

#### 需要 header 整理 ####
| header 參數名稱 | 內容 | 說明 | 
| --- | --- | --- |
| Content-Type | application/json | |
| Authorization | Bearer {ssoToken} | sso 啟用時 |
| Authorization | Basic {base64 user:password} | 當sso沒有啟用時 |
| Cookie | EIToken={ssoToken},WISEUser={userName} | sso 啟用時 |

#### 傳入路徑參數 ####
| 參數名稱 | 內容 | 說明 | 
| --- | --- | --- |
| catalogName | catalog名稱 | 從catalog列表中列出來的名稱 |
| schemaName | schema(database)名稱 | 從schemas列表中列出來的名稱 |
| tableName | table名稱 | 從tables列表中列出來的名稱 |

#### 收到回覆 ####
| 狀態馬 | 內容 | 說明 | 
| --- | --- | --- |
| 200 | OK | 正常建立 |
| 404 | Not Found | API路徑錯誤或服務器尚未啟動 |
| 401 | Invalid credentials | 權限不對或是 token 已過期 |
| 500 | Internal server error | DB 連線錯誤或其他問題 |

#### 收到回覆範例 ####
```dtd
[
    {
        "Column": "ordert",
        "Type": "bigint",
        "Extra": "",
        "Comment": ""
    },
    {
        "Column": "name",
        "Type": "varchar",
        "Extra": "",
        "Comment": ""
    }
]
```

---------------------------------------

### 從table中查詢幾筆資訊 ###
#### API 路徑 ####
GET 服務路徑/v1/api/catalogs/{catalogName}/schemas/{schemaName}/tables/{tableName}/columns/{limitCount}

#### 需要 header 整理 ####
| header 參數名稱 | 內容 | 說明 | 
| --- | --- | --- |
| Content-Type | application/json | |
| Authorization | Bearer {ssoToken} | sso 啟用時 |
| Authorization | Basic {base64 user:password} | 當sso沒有啟用時 |
| Cookie | EIToken={ssoToken},WISEUser={userName} | sso 啟用時 |

#### 傳入路徑參數 ####
| 參數名稱 | 內容 | 說明 | 
| --- | --- | --- |
| catalogName | catalog名稱 | 從catalog列表中列出來的名稱 |
| schemaName | schema(database)名稱 | 從schemas列表中列出來的名稱 |
| tableName | table名稱 | 從tables列表中列出來的名稱 |
| limitCount | 返回限制數量 | 從 table 中搜尋 limitCount 筆數的資料 |

#### 收到回覆 ####
| 狀態馬 | 內容 | 說明 | 
| --- | --- | --- |
| 200 | OK | 正常建立 |
| 404 | Not Found | API路徑錯誤或服務器尚未啟動 |
| 401 | Invalid credentials | 權限不對或是 token 已過期 |
| 500 | Internal server error | DB 連線錯誤或其他問題 |

#### 收到回覆範例 ####
```dtd
[
    {
        "custkey": 10807630,
        "name": "Customer#010807630",
        "address": "U,Ze2JfT1IPeAp8D7j0RA1Qs",
        "nationkey": 6,
        "phone": "16-329-921-9489",
        "acctbal": 1735.74,
        "mktsegment": "FURNITURE",
        "comment": "bove the ironic, final theodolites. fu"
    },
    {
        "custkey": 10807631,
        "name": "Customer#010807631",
        "address": "0CIFLgeqKwl8hf2N0urSUT",
        "nationkey": 0,
        "phone": "10-447-549-1289",
        "acctbal": 9510.98,
        "mktsegment": "BUILDING",
        "comment": "ular escapades cajole quickly. carefully bold ideas haggl"
    }
]
```

