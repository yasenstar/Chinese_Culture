# Create Graph for Chinese Tradition Culture

- [Create Graph for Chinese Tradition Culture](#create-graph-for-chinese-tradition-culture)
  - [Create `阴阳` node](#create-阴阳-node)
  - [Add `天干` node](#add-天干-node)
  - [Add `地支` node](#add-地支-node)
  - [Refactor 天干地支 to Match 阴阳](#refactor-天干地支-to-match-阴阳)
  - [创建`甲子表`，建立天干与地支的关系`配`](#创建甲子表建立天干与地支的关系配)
  - [Query 天干地支](#query-天干地支)
  - [添加 五行、五方、五季](#添加-五行五方五季)
  - [创建 十干五行配合 关系](#创建-十干五行配合-关系)
  - [创建 十二支月建五行所属 关系](#创建-十二支月建五行所属-关系)
  - [十干化运：对于天干到五行之配属关系，添加五行到天干之“主”关系](#十干化运对于天干到五行之配属关系添加五行到天干之主关系)
  - [目前为止的schema](#目前为止的schema)
- [对“五运”建模](#对五运建模)
  - [对“五行”node添加“五运”Label](#对五行node添加五运label)
  - [十干化运](#十干化运)
  - [Refactor 分开 五行 和 五运](#refactor-分开-五行-和-五运)
- [Tool](#tool)

## Create `阴阳` node

```cypher
MERGE (y1:阴阳 {id:1, name:"阳"})
MERGE (y2:阴阳 {id:2, name:"阴"})
```

## Add `天干` node

```cypher
// Load 天干 CSV
LOAD CSV WITH HEADERS FROM 'file:///d://Github//Chinese_Culture//五运六气//csv//天干.csv' AS row
MERGE (t:天干 {id:row.`编号`})
ON CREATE SET
  t.name = row.`天干`,
  t.pinyin = row.`拼音`,
  t.yinyang = row.`阴阳`,
  t.createAt = datetime()
ON MATCH SET
  t.name = row.`天干`,
  t.pinyin = row.`拼音`,
  t.yinyang = row.`阴阳`,
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
  d.yinyang = row.`阴阳`,
  d.createAt = datetime()
ON MATCH SET
  d.name = row.`地支`,
  d.pinyin = row.`拼音`,
  d.yinyang = row.`阴阳`,
  d.lastUpdated = datetime()
RETURN d
```

![地支](img/地支.png)

## Refactor 天干地支 to Match 阴阳

```cypher
// 阳干属阳
MATCH (t:天干),(y:阴阳)
WHERE t.yinyang CONTAINS "阳" AND y.name = "阳"
MERGE (t)-[s:属]->(y)
RETURN t,s,y
```

```cypher
// 阴干属阴
MATCH (t:天干),(y:阴阳)
WHERE t.yinyang = "阴干" AND y.name = "阴"
MERGE (t)-[s:属]->(y)
RETURN t,s,y
```

```cypher
// 阳支属阳
MATCH (d:地支),(y:阴阳)
WHERE d.yinyang CONTAINS "阳" AND y.name = "阳"
MERGE (d)-[s:属]->(y)
RETURN d,s,y
```

```cypher
// 阴支属阴
MATCH (d:地支),(y:阴阳)
WHERE d.yinyang CONTAINS "阴" AND y.name = "阴"
MERGE (d)-[s:属]->(y)
RETURN d,s,y
```

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

## Query 天干地支

```cypher
MATCH (t:天干)-[p:配]->(d:地支)
RETURN t,p,d
```

## 添加 五行、五方、五季

```cypher
// Load 五行 CSV
LOAD CSV WITH HEADERS FROM 'file:///d://Github//Chinese_Culture//五运六气//csv//五行.csv' AS row
MERGE (w:五行 {id:row.`编号`})
ON CREATE SET
  w.name = row.`五行`,
  w.createAt = datetime()
ON MATCH SET
  w.name = row.`五行`,
  w.lastUpdated = datetime()
RETURN w
```

```cypher
// Load 五方 CSV
LOAD CSV WITH HEADERS FROM 'file:///d://Github//Chinese_Culture//五运六气//csv//五方.csv' AS row
MERGE (w:五方 {id:row.`编号`})
ON CREATE SET
  w.name = row.`五方`,
  w.createAt = datetime()
ON MATCH SET
  w.name = row.`五方`,
  w.lastUpdated = datetime()
RETURN w
```

```cypher
// Load 五季 CSV
LOAD CSV WITH HEADERS FROM 'file:///d://Github//Chinese_Culture//五运六气//csv//五季.csv' AS row
MERGE (w:五季 {id:row.`编号`})
ON CREATE SET
  w.name = row.`五季`,
  w.createAt = datetime()
ON MATCH SET
  w.name = row.`五季`,
  w.lastUpdated = datetime()
RETURN w
```

## 创建 十干五行配合 关系

```cypher
// Load 十干五行配合表 CSV
LOAD CSV WITH HEADERS FROM 'file:///d://Github//Chinese_Culture//五运六气//csv//十干五行配合表.csv' AS row
MATCH (t:天干), (w1:五行), (w2:五方), (w3:五季)
WHERE t.name = row.`天干` AND w1.name = row.`五行` AND w2.name = row.`五方` AND w3.name = row.`五季`
MERGE (t)-[p1:配]->(w1)
MERGE (w1)-[p2:配]->(w2)
MERGE (w2)-[p3:配]->(w3)
RETURN t,p1,p2,p3,w1,w2,w3
```

![十干五行配合](img/十干五行配合.png)

## 创建 十二支月建五行所属 关系

```cypher
// Load 十二支月建五行所属 CSV
LOAD CSV WITH HEADERS FROM 'file:///d://Github//Chinese_Culture//五运六气//csv//十二支月建五行所属.csv' AS row
MATCH (d:地支),(w:五行)
WHERE d.name = row.`地支` AND w.name = row.`五行`
SET d.monthNumber = row.`月份`, d.monthName = row.`月名`
MERGE (d)-[s:属]->(w)
RETURN d,s,w
```

![十二支月建五行所属](img/十二支月建五行所属.png)

## 十干化运：对于天干到五行之配属关系，添加五行到天干之“主”关系

```cypher
MATCH (t:天干)-[r:配]->(w:五行)
MERGE (w)-[z:主 {source: "十二支月建五行所属"}]->(t)
RETURN t,r,w,z
```

![天干和五行](img/天干与五行.png)

## 目前为止的schema

![schema01](img/schema01.png)

# 对“五运”建模

## 对“五行”node添加“五运”Label

```cypher
MATCH (w:五行)
SET w:五运
RETURN w
```

![五运](img/五运.png)

## 十干化运

素问五运行大论：“土主甲己，金主乙庚，水主丙辛，木主丁壬，火主戊癸。”

```cypher
// Load 十干化运 CSV
LOAD CSV WITH HEADERS FROM 'file:///d://Github//Chinese_Culture//五运六气//csv//十干化运.csv' AS row
MATCH (t:天干), (w:五运)
WHERE t.name = row.`天干` AND w.name = row.`五运`
MERGE (w)-[z:主 {source: "十干化运"}]->(t)
RETURN w,z,t
```

Note: add "source" per relationship.

使用下面的查询可以看到现在的关系“主”有不同的配合：

```cypher
MATCH p=()-[:`主`]->() RETURN p
```

![十干配运](img/十干配运.png)

|m.name|n.name|z.source |
|------|------|---------|
|土     |戊     |十二支月建五行所属|
|土     |己     |十二支月建五行所属|
|土     |甲     |十干化运     |
|土     |己     |十干化运     |
|木     |甲     |十二支月建五行所属|
|木     |乙     |十二支月建五行所属|
|木     |丁     |十干化运     |
|木     |壬     |十干化运     |
|水     |壬     |十二支月建五行所属|
|水     |癸     |十二支月建五行所属|
|水     |丙     |十干化运     |
|水     |辛     |十干化运     |
|火     |丙     |十二支月建五行所属|
|火     |丁     |十二支月建五行所属|
|火     |戊     |十干化运     |
|火     |癸     |十干化运     |
|金     |庚     |十二支月建五行所属|
|金     |辛     |十二支月建五行所属|
|金     |乙     |十干化运     |
|金     |庚     |十干化运     |

## Refactor 分开 五行 和 五运

```cypher
// 从五行中删除“五运”标签
MATCH (w:五运) REMOVE w:五运
```

再CSV源文件中区分五行和五运，如叫“土行”对应“土运”，重新运行“五行”，“五方”，“五季”创建语句，名称更新包含了后缀。

![新五行](img/新五行.png)

对如下“土行”与“天干”的“主”的关系，清除来自“十干化运”的链接，为后面建立“土运”的关系做准备：

![土行的主](img/土行的主.png)

```cypher
MATCH (w:五行)-[z:主 {source: "十干化运"}]->(m)
DELETE z
```

运行以后，“土行”的“主”关系为：

![土行的主-新](img/土行的主-新.png)

---

# Tool

- [Convert CSV to Markdown Table](https://www.convertcsv.com/csv-to-markdown.htm)

Last updated at 12/5/2025, 8:26:37 PM 