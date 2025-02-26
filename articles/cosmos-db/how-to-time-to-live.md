---
title: 了解如何在 Azure Cosmos DB 中配置和管理生存时间
description: 了解如何在 Azure Cosmos DB 中配置和管理生存时间
author: rockboyfor
ms.service: cosmos-db
ms.topic: sample
origin.date: 05/23/2019
ms.date: 07/29/2019
ms.author: v-yeche
ms.openlocfilehash: 6b3352ae0692df5e05aff4cc7a230a52aa420ad0
ms.sourcegitcommit: 021dbf0003a25310a4c8582a998c17729f78ce42
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/26/2019
ms.locfileid: "68514292"
---
# <a name="configure-time-to-live-in-azure-cosmos-db"></a>在 Azure Cosmos DB 中配置生存时间

在 Azure Cosmos DB 中，可以选择在容器级别配置生存时间 (TTL)，也可以在为容器完成设置之后，在项级别重写生存时间。 可以通过 Azure 门户或特定于语言的 SDK 配置容器的 TTL。 可以通过 SDK 配置项级别 TTL 重写。

## <a name="enable-time-to-live-on-a-container-using-azure-portal"></a>使用 Azure 门户在容器上启用生存时间

通过以下步骤在没有到期时间的容器上启用生存时间。 启用此项即可在项级别重写 TTL。 也可通过输入非零值（代表秒）来设置 TTL。

1. 登录到 [Azure 门户](https://portal.azure.cn/)。

2. 创建新的 Azure Cosmos 帐户或选择现有的帐户。

3. 打开“数据资源管理器”窗格  。

4. 选择一个现有的容器，将其展开并修改以下值：

    * 打开“规模和设置”窗口。 
    * 在“设置”下找到“生存时间”。  
    * 选择“启用(无默认值)”或选择“启用”，然后设置一个 TTL 值  
    * 单击“保存”  以保存更改。

    ![在 Azure 门户中配置生存时间](./media/how-to-time-to-live/how-to-time-to-live-portal.png)
- 当 DefaultTimeToLive 为 null 时，生存时间为“关”
- 当 DefaultTimeToLive 为 -1 时，“生存时间”设置为“开”（无默认值）
- 当 DefaultTimeToLive 具有任何其他整数值（0 除外）时，“生存时间”设置为“开”

## <a name="enable-time-to-live-on-a-container-using-sdk"></a>使用 SDK 在容器上启用生存时间

<a name="dotnet-enable-noexpiry"></a>
### <a name="net-sdk"></a>.NET SDK

```csharp
// Create a new collection with TTL enabled and without any expiration value
DocumentCollection collectionDefinition = new DocumentCollection();
collectionDefinition.Id = "myContainer";
collectionDefinition.PartitionKey.Paths.Add("/myPartitionKey");
collectionDefinition.DefaultTimeToLive = -1; //(never expire by default)

DocumentCollection ttlEnabledCollection = await client.CreateDocumentCollectionAsync(
    UriFactory.CreateDatabaseUri("myDatabaseName"),
    collectionDefinition,
    new RequestOptions { OfferThroughput = 20000 });
```

## <a name="set-time-to-live-on-a-container-using-sdk"></a>使用 SDK 在容器上设置生存时间

<a name="dotnet-enable-withexpiry"></a>
### <a name="net-sdk"></a>.NET SDK

若要在容器上设置生存时间，需提供一个非零正数来指示时间段（以秒为单位）。 在项的上次修改的时间戳 (`_ts`) 过后，将会删除容器中的所有值，具体取决于配置的 TTL 值。

```csharp
// Create a new collection with TTL enabled and a 90 day expiration
DocumentCollection collectionDefinition = new DocumentCollection();
collectionDefinition.Id = "myContainer";
collectionDefinition.PartitionKey.Paths.Add("/myPartitionKey");
collectionDefinition.DefaultTimeToLive = 90 * 60 * 60 * 24; // expire all documents after 90 days

DocumentCollection ttlEnabledCollection = await client.CreateDocumentCollectionAsync(
    UriFactory.CreateDatabaseUri("myDatabaseName"),
    collectionDefinition,
    new RequestOptions { OfferThroughput = 20000 });
```

<a name="nodejs-enable-withexpiry"></a>
### <a name="nodejs-sdk"></a>NodeJS SDK

```javascript
const containerDefinition = {
          id: "sample container1",
        };

async function createcontainerWithTTL(db: Database, containerDefinition: ContainerDefinition, collId: any, defaultTtl: number) {
      containerDefinition.id = collId;
      containerDefinition.defaultTtl = defaultTtl;
      await db.containers.create(containerDefinition);
}
```

## <a name="set-time-to-live-on-an-item"></a>在某个项上设置生存时间

除了在容器上设置默认的生存时间，还可以为项设置生存时间。 在项级别设置生存时间将重写该容器中项的默认 TTL。

* 若要在项上设置 TTL，则需要提供一个非零正数，该数字表示自项上次的修改时间戳 (`_ts`) 之后，项过期所经过的时间段（以秒为单位）。

* 如果项没有 TTL 字段，则默认情况下，设置到容器的 TTL 将应用到项。

* 如果在容器级别禁用了 TTL，在项上的 TTL 字段会被忽略，直到在容器上再次启用 TTL。

<a name="portal-set-ttl-item"></a>
### <a name="azure-portal"></a>Azure 门户

通过以下步骤在项上启用生存时间：

1. 登录到 [Azure 门户](https://portal.azure.cn/)。

2. 创建新的 Azure Cosmos 帐户或选择现有的帐户。

3. 打开“数据资源管理器”窗格  。

4. 选择一个现有的容器，将其展开并修改以下值：

    * 打开“规模和设置”窗口。 
    * 在“设置”下找到“生存时间”。  
    * 选择“启用(无默认值)”或选择“启用”，然后设置一个 TTL 值。   
    * 单击“保存”  以保存更改。

5. 接下来导航到要为其设置生存时间的项，添加 `ttl` 属性，然后选择“更新”。  

    ```json
    {
    "id": "1",
    "_rid": "Jic9ANWdO-EFAAAAAAAAAA==",
    "_self": "dbs/Jic9AA==/colls/Jic9ANWdO-E=/docs/Jic9ANWdO-EFAAAAAAAAAA==/",
    "_etag": "\"0d00b23f-0000-0000-0000-5c7712e80000\"",
    "_attachments": "attachments/",
    "ttl": 10,
    "_ts": 1551307496
    }
    ```

<a name="dotnet-set-ttl-item"></a>
### <a name="net-sdk"></a>.NET SDK

```csharp
// Include a property that serializes to "ttl" in JSON
public class SalesOrder
{
    [JsonProperty(PropertyName = "id")]
    public string Id { get; set; }
    [JsonProperty(PropertyName="cid")]
    public string CustomerId { get; set; }
    // used to set expiration policy
    [JsonProperty(PropertyName = "ttl", NullValueHandling = NullValueHandling.Ignore)]
    public int? ttl { get; set; }

    //...
}
// Set the value to the expiration in seconds
SalesOrder salesOrder = new SalesOrder
{
    Id = "SO05",
    CustomerId = "CO18009186470",
    ttl = 60 * 60 * 24 * 30;  // Expire sales orders in 30 days
};
```

<a name="nodejs-set-ttl-item"></a>
### <a name="nodejs-sdk"></a>NodeJS SDK

```javascript
const itemDefinition = {
          id: "doc",
          name: "sample Item",
          key: "value", 
          ttl: 2
        };
```

## <a name="reset-time-to-live"></a>重置生存时间

可以通过在项上执行写入或更新操作，重置项的生存时间。 写入或更新操作会将 `_ts` 设置为当前时间，要到期的项的 TTL 就会重启。 若要更改项的 TTL，可以更新相关字段，就像更新任何其他字段那样。

<a name="dotnet-extend-ttl-item"></a>
### <a name="net-sdk"></a>.NET SDK

```csharp
// This examples leverages the Sales Order class above.
// Read a document, update its TTL, save it.
response = await client.ReadDocumentAsync(
    "/dbs/salesdb/colls/orders/docs/SO05"),
    new RequestOptions { PartitionKey = new PartitionKey("CO18009186470") });

Document readDocument = response.Resource;
readDocument.ttl = 60 * 30 * 30; // update time to live
response = await client.ReplaceDocumentAsync(readDocument);
```

## <a name="turn-off-time-to-live"></a>禁用生存时间

如果已在项上设置生存时间，并且不再想要该项过期，则可以获取该项，删除 TTL 字段并替换服务器上的项。 从项中删除 TTL 字段时，分配给容器的默认 TTL 值会应用到项。 将 TTL 值设置为 -1 可以防止项过期，并且不会从容器继承 TTL 值。

<a name="dotnet-turn-off-ttl-item"></a>
### <a name="net-sdk"></a>.NET SDK

```csharp
// This examples leverages the Sales Order class above.
// Read a document, turn off its override TTL, save it.
response = await client.ReadDocumentAsync(
    "/dbs/salesdb/colls/orders/docs/SO05"),
    new RequestOptions { PartitionKey = new PartitionKey("CO18009186470") });

Document readDocument = response.Resource;
readDocument.ttl = null; // inherit the default TTL of the collection

response = await client.ReplaceDocumentAsync(readDocument);
```

## <a name="disable-time-to-live"></a>禁用生存时间

若要在容器上禁用生存时间并阻止后台进程查找过期项，则应删除容器上的 `DefaultTimeToLive` 属性。 删除此属性不同于将其设置为 -1。 将它设置为 -1 时，添加到容器的新项将永久生存。但是，可以在容器中的特定项上重写此值。 从容器中删除 TTL 属性后项就不会过期，即使这些项已显式重写了以前的默认 TTL 值。

<a name="dotnet-disable-ttl"></a>
### <a name="net-sdk"></a>.NET SDK

```csharp
// Get the collection, update DefaultTimeToLive to null
DocumentCollection collection = await client.ReadDocumentCollectionAsync("/dbs/salesdb/colls/orders");
// Disable TTL
collection.DefaultTimeToLive = null;
await client.ReplaceDocumentCollectionAsync(collection);
```

## <a name="next-steps"></a>后续步骤

在以下文章中详细了解生存时间：

* [生存时间](time-to-live.md)

<!-- Update_Description: update meta properties， wording update -->
