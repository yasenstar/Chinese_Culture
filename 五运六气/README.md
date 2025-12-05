# Create Graph for Chinese Tradition Culture

- [Create Graph for Chinese Tradition Culture](#create-graph-for-chinese-tradition-culture)
  - [Add `天干` node](#add-天干-node)
  - [Add `地支` node](#add-地支-node)
  - [创建`甲子表`，建立天干与地支的关系`配`](#创建甲子表建立天干与地支的关系配)

## Add `天干` node

```cypher
// Load 天干 CSV
LOAD CSV WITH HEADERS FROM 'file:///d://Github//Chinese_Culture//五运六气//csv//天干.csv' AS row
MERGE (t:天干 {id:row.`编号`})
ON CREATE SET
  t.name = row.`天干`,
  t.pinyin = row.`拼音`,
  t.createAt = datetime()
ON MATCH SET
  t.name = row.`天干`,
  t.pinyin = row.`拼音`,
  t.lastUpdated = datetime()
RETURN t
```

![天干](img/天干.png)

## Add `地支` node

```cypher
// Load 地支 CSV
LOAD CSV WITH HEADERS FROM 'file:///d://Github//Chinese_Culture//五运六气//csv//地支.csv' AS row
MERGE (d:地支 {id:row.`编号`})
ON CREATE SET
  d.name = row.`地支`,
  d.pinyin = row.`拼音`,
  d.createAt = datetime()
ON MATCH SET
  d.name = row.`地支`,
  d.pinyin = row.`拼音`,
  d.lastUpdated = datetime()
RETURN d
```

![地支](img/地支.png)

## 创建`甲子表`，建立天干与地支的关系`配`

```cypher
// Load 甲子表 CSV
LOAD CSV WITH HEADERS FROM 'file:///d://Github//Chinese_Culture//五运六气//csv//甲子表.csv' AS row
MATCH (t:天干), (d:地支)
WHERE t.name = row.`天干` AND d.name = row.`地支`
MERGE (t)-[p:配 {id:row.`编号`}]->(d)
ON CREATE SET
  d.createAt = datetime()
ON MATCH SET
  d.lastUpdated = datetime()
RETURN t,p,d
```

![甲子表](img/甲子表.png)