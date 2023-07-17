# kvcluster

主要設計用 Statefulset 來管理分片化的 JetStream Cluster，為了方便說明參考以下表格

## Port usage
對每一個 Pod 而言，總共會暴露三個端口，分別有各自的用處。

|         | port |
|---------|------|
| client  | 4222 |
| cluster | 6222 |
| http    | 8222 |

* client: 用來服務客戶端
* cluster: 專門用來溝通群集內部 Raft State Machine Replication
* http: 提供一些當前伺服器資訊

## Sharding cluster design
首先使用者可以單純修改 Statefulset 的數量來達到水平擴展，特別注意必須是三的整數倍，如果沒有依照此設定會有分片無法正常運作。

設計上 Statefulset 會以三個 Pod 為單位各自行形成一個 Raft Group，並且以此類推，下表會說明一個三個分片設計，一共九個 Pod 的設計，當然使用者可以自行決定要擴展到多少分片。


| Pod Id      | Shard Id/Cluster | Routes                                                                                                                                                           |
|-------------|------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| kvcluster-0 | shard-0          | kvcluster-0.kvcluster.default.svc.cluster.local:6222, kvcluster-1.kvcluster.default.svc.cluster.local:6222, kvcluster-2.kvcluster.default.svc.cluster.local:6222 |
| kvcluster-1 | shard-0          | kvcluster-0.kvcluster.default.svc.cluster.local:6222, kvcluster-1.kvcluster.default.svc.cluster.local:6222, kvcluster-2.kvcluster.default.svc.cluster.local:6222 |
| kvcluster-2 | shard-0          | kvcluster-0.kvcluster.default.svc.cluster.local:6222, kvcluster-1.kvcluster.default.svc.cluster.local:6222, kvcluster-2.kvcluster.default.svc.cluster.local:6222 |
| kvcluster-3 | shard-1          | kvcluster-3.kvcluster.default.svc.cluster.local:6222, kvcluster-4.kvcluster.default.svc.cluster.local:6222, kvcluster-5.kvcluster.default.svc.cluster.local:6222 |
| kvcluster-4 | shard-1          | kvcluster-3.kvcluster.default.svc.cluster.local:6222, kvcluster-4.kvcluster.default.svc.cluster.local:6222, kvcluster-5.kvcluster.default.svc.cluster.local:6222 |
| kvcluster-5 | shard-1          | kvcluster-3.kvcluster.default.svc.cluster.local:6222, kvcluster-4.kvcluster.default.svc.cluster.local:6222, kvcluster-5.kvcluster.default.svc.cluster.local:6222 |
| kvcluster-6 | shard-2          | kvcluster-6.kvcluster.default.svc.cluster.local:6222, kvcluster-7.kvcluster.default.svc.cluster.local:6222, kvcluster-8.kvcluster.default.svc.cluster.local:6222 |
| kvcluster-7 | shard-2          | kvcluster-6.kvcluster.default.svc.cluster.local:6222, kvcluster-7.kvcluster.default.svc.cluster.local:6222, kvcluster-8.kvcluster.default.svc.cluster.local:6222 |
| kvcluster-8 | shard-2          | kvcluster-6.kvcluster.default.svc.cluster.local:6222, kvcluster-7.kvcluster.default.svc.cluster.local:6222, kvcluster-8.kvcluster.default.svc.cluster.local:6222 |

