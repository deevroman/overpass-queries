# overpass-queries
Некоторые (бес)полезные запросы к [Overpass-turbo](https://overpass-turbo.eu/) ([зеркало от mail.ru](https://maps.mail.ru/osm/tools/overpass/))


### Сёла без домов
```graphql
area
  ["boundary"="administrative"]
  ["name"="Смоленская область"]
->.b;

(
  node(area.b)
    ["place"~"village"];
)->.c;

foreach .c -> .d (
  (way[building](around.d:500); .d;);
  node._(if:count(ways) == 0);
  out;
);

```

<details>
<summary>Более быстрый, но менее гибкий запрос</summary>
https://gis.stackexchange.com/questions/407903/places-near-which-there-are-no-buildings
  
```graphql
area
  ["boundary"="administrative"]
  ["name"="Смоленская область"]
->.b;

(
  node(area.b)
    ["place"~"village"];
)->.c;

(
  way[building](around.c:500)->.build; // <- !
) -> .build;

(
  node(around.build:500)
    ["place"~"village"];
) -> .d;

(.c; - .d;)->.result;

.result out center;
```

</details>

### Висячие подъезды 
```graphql
{{geocodeArea:"Центральный федеральный округ"}}->.b;

(
node(area.b)[entrance]; > -> .ent;
)->.ent;

(
node.ent; < -> .w;
)->.w;

(
way.w; > -> .w;
)->.w;

( node(area.b)[entrance]; - node.w; );
out meta;
```

### Имена рек, требующие исправления. v3 https://maproulette.org/browse/challenges/24018
```graphql
area
  ["boundary"="administrative"]
  ["name"="Центральный федеральный округ"]
->.b;

way(area.b)[waterway]["name"~"^[Рр](ека|\\.).*[^яй]$"];

out center;
```

### Имена рек, требующие исправления. v4 
```graphql
area
  ["boundary"="administrative"]
  ["name"="Центральный федеральный округ"]
->.b;

way(area.b)[waterway]["name"~"^[Рр](е[ч]?ка|\\.|уч).*[^аяй]$"];

out center;
```
  
### Фильтрация по длине значения ключа
```graphql
node["addr:flats"~".{150,}"];

out body;
>;
out skel qt;
```

  
### Граффити без artwork_type
```graphql
node
  [name~".*раффити.*"][!artwork_type][tourism=artwork];
out;
```
