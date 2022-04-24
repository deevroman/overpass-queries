# overpass-queries
Некоторые (бес)полезные запросы к Overpass-turbo


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

### Имена рек Имена рек, требующие исправления. v4 
```graphql
area
  ["boundary"="administrative"]
  ["name"="Центральный федеральный округ"]
->.b;

way(area.b)[waterway]["name"~"^[Рр](е[ч]?ка|\\.|уч).*[^аяй]$"];

out center;
```

