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

### Запрос для выискивания значения тегов, в которых есть ;
<details>
<summary>Тык</summary>
  
// WARN постепенным добавлением ключей можно скрыть искомые ключи
  
// Например, если у всех точек с somekey есть name, то они не будут обнаружены этим запросом
  
// Однако запрос простой и быстрый по времени, чтобы получить примерный список 
```graphql
[out:json]
[timeout:25];
node[~".*"~".*;.*"]
[!opening_hours]
[!"opening_hours:covid19"]
[!collection_times]
[!"addr:flats"]
[!voltage]
[!"voltage:primary"]
[!utility]
[!waste]
[!cuisine]
[!phone]
[!source]
[!note]
[!fixme]
[!name]
[!old_name]
[!description]
[!inscription]
[!"inscription:1"]
[!"subject:wikidata"]
[!"contact:phone"]
[!vending]
[!brand]
[!"camera:direction"]
[!craft]
[!ref]
[!"toilets:position"]
[!"seamark:buoy_cardinal:colour"]
[!"seamark:buoy_lateral:colour"]
[!"seamark:buoy_isolated_danger:colour"]
[!"seamark:buoy_special_purpose:colour"]
[!"seamark:beacon_lateral:colour"]
[!"seamark:cable_submarine:name"]
[!"seamark:notice:addition"]
[!"seamark:notice:impact"]
[!"seamark:topmark:colour"]
[!sport]
[!level]
[!levels]
[!"building:levels"]
[!clothes]
[!direction]
[!designated]
[!"motor_vehicle:conditional"]
[!"access:conditional"]
[!"was:collection_times"]
[!"was:opening_hours"]
[!operator]
[!shop]
[!material]
[!crossing]
[!barrier]
[!manhole]
[!playground]
[!map_type]
[!fitness_station]
[!colour]
[!amenity]
[!animal]
[!traffic_sign]
[!"traffic_sign:backward"]
[!"traffic_sign:forward"]
[!"addr:housenumber"]
[!"addr:unit"]
[!"communication:mobile_phone"]
[!destination]
[!"destination:backward"]
[!"destination:forward"]
[!"turn:lanes:forward"]
[!"turn:lanes:backward"]
[!"motorcycle:conditional"]
[!"traffic_signals:direction"]
[!"motorcar:conditional"]
[!"restriction:conditional"]
[!"turn:lanes"]
[!"piste:grooming"]
[!"building:material"]
[!"building:cladding"]
[!"building:levelPlan"]
[!start_date]
[!surface]
[!"building:part:use"]
[!"species:ru"]
[!length]
[!motor_vehicle]
[!weather_protection]
[!layer]
[!access]
[!content]
[!product]
[!crop]
[!route_ref]
[!species]
[!"destination:ref"]
[!traffic_calming]
[!information]
[!"generator:source"]
[!network]
[!whitewater]
[!"flag:wikidata"]
[!"flag:type"]
[!"addr:postcode"]
[!traffic_signals]
[!stop]
[!door]
[!kerb]
[!highway]
[!railway]
[!antenna]
[!repeat_on]
[!specality]
[!"disused:ref"]
[!"surveillance:zone"]
[!"fire_hydrant:type"]
[!"destination:lanes"]
[!"healthcare:speciality"]
[!seasonal]
[!man_made]
[!curb]
[!"graffiti:tag"]
[!pipelinemarker]({{bbox}});

out body;
>;
out skel qt;
``` 
  
</details>
  
### Озёра без water=lake
```graphql
(
  way[name~".*ое$"][natural=water][!water]({{bbox}});
  way[name~".*зеро.*"][natural=water][!water]({{bbox}});
);
(._;>;);
out;
```

### Отношения рек без связи с википедией
```graphql
// TODO реки с викидатой, но в отношении
// TODO длинные без отношения
relation[waterway=river][!wikidata]({{bbox}});
out body;
>;
out skel qt;
```

### Подозрительные объекты на со словом яндекс
```graphql
// TODO убрать маркет, такси регулярками
// TODO все типы
node[~".*"~"(я|Я)ндекс"][shop!=outpost][office!=it][amenity!=vending_machine][amenity!=parcel_locker][source!="Яндекс Панорамы"][office!=company][name!="Яндекс.Маркет"];
out;
```
  
