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

  
<details>
<summary>Аналогичное для кладбищ</summary>
  
```graphql
area
  ["boundary"="administrative"]
  ["name"="Воронежская область"]
->.b;

(
  node(area.b)
  	["place"="village"];
)->.c;

(
  wr(area.b)["landuse"="cemetery"]->.build;
) -> .build;

(
  node(around.build:4000)
  	["place"="village"];
) -> .d;

(.c; - .d;)->.result;

.result out center;
```
</details>

<details>
<summary>Аналогичное для лежачих поличейских около школ</summary>
  
```graphql
area
  ["boundary"="administrative"]
  ["name"="Липецкая область"]
->.b;

(
  node(area.b)
    ["amenity"="school"];
)->.c;

(
  way[traffic_calming](around.c:2000)->.build; // <- !
) -> .build;

(
  node(around.build:1000)
    ["amenity"="school"];
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

#### Имена рек, требующие исправления. v4 
```graphql
area
  ["boundary"="administrative"]
  ["name"="Центральный федеральный округ"]
->.b;

way(area.b)[waterway]["name"~"^[Рр](е[ч]?ка|\\.|уч).*[^аяй]$"];

out center;
```
  
#### Имена рек, требующие исправления. v5
```graphql
area
  ["boundary"="administrative"]
  ["name"="Центральный федеральный округ"]
->.b;

way(area.b)[waterway]["name"~"^[Рр].*?\\..*$"];

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

### Попытка найти нетривиальные знаки Уступи дорогу без направления
```graphql
// тривиальные — на перекрёстках дорог с разным статусом. Но почему-то не всегда работает:(
area
  ["boundary"="administrative"]
  ["name"="Воронежская область"]
->.b;

way[highway][!oneway](area.b)->.h;
.h > -> .h;
node.h["highway"="give_way"][!direction](area.b)->.c;

(
  way[highway=trunk](around.с:100);
  way[highway=primary](around.с:100);
  way[highway=secondary](around.с:100);
)->.w;

.w > -> .w;

node.h(around.w:100)->.d;

(.c; - .d;)->.res;

.res out body;
```
  
### Для шаурмы и шашлыка на русском
```graphql
node[cuisine~"шаурма|шава|шашлык", i];
out body;
>;
out skel qt;
```
  
### Длинные ЛЭП без вольтажа
```graphql
{{geocodeArea::Russia}}->.a;
(
  way["power"="line"][!voltage](area.a)(if: length() > 100000);
);
out body;
>;
out skel qt;
```
  
### Вероятно ошибочные островки безопасности
```graphql
area["boundary"="administrative"]["name"="Россия"]->.b;
way["crossing:island"="yes"](area.b)(if: length() > 60);
(._;>;);
out;
```
  
  
### fixme=continue не на концах линий
```graphql
node[fixme=continue]({{bbox}});
(._;<;);
(._;>;)->.x;

node.x[fixme=continue]->.a;
node(w.x:1,-1)[fixme=continue]->.b;
(.a; - .b;);

out geom;
```

### Длинные footway=crossing + поиск в нескольких округах

```graphql
(
  {{geocodeArea:"Северо-Западный федеральный округ"}};
  {{geocodeArea:"Центральный федеральный округ"}};
  {{geocodeArea:"Южный федеральный округ"}};
  {{geocodeArea:"Северо-Кавказский федеральный округ"}};
  {{geocodeArea:"Приволжский федеральный округ"}};
)->.b;
way[footway=crossing](area.b)(if: length() > 150);
(._;>;);
out;
```

### Геометрия с слишком острыми углами

```graphql
way({{bbox}})(if:lrs_in(1,per_vertex(angle() < -175 || angle() > 175)));
out geom;
```

### Названия улиц на точках
```graphql
{{geocodeArea:"Russian Federation"}}->.a;
node["name"~"улица"][highway!=bus_stop][!public_transport][highway!=platform](if: count_tags() == 1)(area.a);
out geom;
```

### Самые крайние остановки

```graphql
[timeout:250];
{{geocodeArea:"Russian Federation"}}->.a;
node[highway=bus_stop](area.a)->.b;

node.b(if:lat() == b.min(lat())); out;
node.b(if:lon() == b.min(lon())); out;
node.b(if:lat() == b.max(lat())); out;
node.b(if:lon() == b.max(lon())); out;
```


#### Попытка оптимизации

```graphql
[timeout:250];
{{geocodeArea:"Россия"}}->.a;
node[highway=bus_stop](area.a)->.b;

make node ::geom = pt(b.min(lat()), b.min(lon()))->.mins;
make node ::geom = pt(b.max(lat()), b.max(lon()))->.maxs;

node.b(if:lat() == mins.min(lat())); out;
node.b(if:lon() == mins.min(lon())); out;
node.b(if:lat() == maxs.max(lat())); out;
node.b(if:lon() == maxs.max(lon())); out;
```
