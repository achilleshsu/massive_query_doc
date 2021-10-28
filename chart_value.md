# massive query helm chart 參數說明 #

#### 編輯日期:2021/10/27 ####
#### chart 版本號: 0.2.2 ####

#### values.yaml ####
| 參數 | 說明 |
|---|---|
| image.repository | 鏡像拉取地址 |
| image.pullPolicy | 拉取策略 IfNotPresent |
| image.tag | 版本號 |
| imagePullSecrets.name | 拉取鏡像所需密碼 |
| server.workers | worker節點數量 |
| additionalConfigProperties | config的額外參數 |
| additionalConfigProperties.http-server.http.port | 記住這邊永遠設為 8080，這代表的是內部溝通的通道，請看下圖說明|
| service.type | 預設使用 ClusterIP |
| service.port | 使用8443時請將ssl.enabled設為true，否則請設為8080 |
| ssl.enabled | 設為true時請將service.port設為8443，否則請設為8080 |
| ssl.additionalConfigProperties | 當 ssl enabled 為 ture 時會使用到的參數，其中authentication.type可選擇一種或兩種都使用 | 
| env.authtype | 設定是否要使用 sso 認證，當s sl.enable 為 ture 時請寫 sso，反之請寫 jwt | 
| env.sso_url | 設定 sso API 位置，當 ssl.enable 為 false 時，此參數不生效 | 
| additionalCatalogs | 新版本這邊的設定無效，請透過API | 
| ingress | 對外連線的API端口設定 | 
| resources | minerva 使用的資源大小，請注意設定此區域的參數不可小於 jvm 的大小 |

#### 叢集結點示意圖 ####
![叢集結點示意圖](.\picture\mierva.JPG)

#### template/configmap-coordinator.yaml ####
#### template/configmap-worker.yaml ####
| 參數 | 說明 |
|---|---|
| data | 這裡存放了各種 minerva 要使用的設定，有很多是預先設定好的，並不用修正，但如果有特別需求可以修改這裡的設定，coordinator 的設定必須和 worker 設定相同 |
| node.properties | 節點的一般環境配置，包含資料夾目錄位置|
| jvm.config | java jvm 的設置，包含了緩存到disk 的功能|
| config.properties | minerva 的一般設定，基本不用修改|
| log.properties | minerva 的 log 設定位置 |
| jwks.json | 這是存放jwk 的文件配置，必須要有文件，但內容可設定為空值 |
| password-authenticator.properties | 帳號密碼認證的配置內容 |
| password.db | 儲存帳號密碼的位置，這邊儲存了一組 admin/admin 得帳號密碼，當authtype 設為jwt 或 password的時候可以使用 admin/admin的帳號密碼登入 |
| access-control.properties | 配置登入認證的規則文件位置、刷新時間秒數 |
| rules.json | 設置使用者登入規則的地方，新版會改由自動讀取DB內容，沒有使用時請留空"{}" |
| group-provider.properties | 配置群組的規則文件位置、刷新時間秒數 |
| groups.txt | 設置群組規則，新版會改由自動讀取DB內容，沒有使用時請留空 |
| cacert.pem | 設置憑證的地方，建議與平台使用相同憑證，否則須要信任憑證才可以連接 |

