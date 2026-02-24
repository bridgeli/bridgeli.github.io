---
title: Redis GeoHash 的一个小示例
author: Bridge Li
type: post
date: 2019-05-19T09:38:46+00:00

categories:
  - Java
  - Redis
tags:
  - GeoHash
  - LBS
  - 地理位置

---
上周产品经理提了一个类似于 LBS 的应用，第一时间想到了忘记了之前什么时候看 Redis 的 API，发现 Redis 自 3.2 版本之后，新增了一类关于地理位置相关的 API，于是拿来测试一下，发现特别好用，写一个小例子作为笔记。

首先需要说明的是，由于我们公司的 JDK 的版本是 1.7，所以我采用的 spring-data-redis 的版本是：1.8.20.RELEASE，最新二点几的版本已经不支持 JDK 1.7，而一点几和二点几的版本的 API 有略微的差异（下面会说明，还有一点点我的小感悟），废话不多说，直接看例子：

```

@Override  
public Long geoAdd(String key, List<Entity> entities) {  
redisTemplate.delete(key);  
GeoOperations geoOperations = redisTemplate.opsForGeo();  
Map<String, Point> map = new HashMap<>();  
Point point = null;  
for (Entity entity : entities) {  
point = new Point(entity.getLongitude(), entity.getLatitude());  
map.put(gson.toJson(entity), point);  
}

Long add = geoOperations.geoAdd(key, map);

return add;

}

```

参数就两个很简单，一个是 key，一个是数据集，我们将在这个集合中找出符合条件的数据，需要说明的是：

1. 第一行我先调用了一个 delete 方法，是将上次放进去的数据删除，因为这个命令是 add，也就是新增，但是 redis 并没有提供直接删除这个 key 的命令（有一个 remove 的方法，但是需要传入删除哪些数据，也就是不能只给一个 key，把这个 key 对应的数据都删除，个人感觉不太好用），当然你也可以在计算后取得相应的数据之后删除，个人感觉都一样，不要忘记清理数据就行，另一个方法就是设置过期时间，也都行；  
2. 为什么要清理数据？因为这个 add，不同的情况，放进去的数据应该不同的，如果 entities 已经发生变更，而一直 add，那么数据将会乱掉，所以先把之前的数据删掉再说；  
3. 我个人采用的是把数据放到了 Map 中，其中 key 是对象序列化之后的 json 串，目的是为了下面找到对应的数据之后，直接反序列化成对象进行返回，当然也可以采用其他的方案，还有就是 add 还有一些其他的 API，这个大家可以自己看文档，选择合适的就行;

数据放进去之后就是计算了，我们的需求就是算一个人旁边几公里内有多少符合条件的数据，代码如下：

```

@Override  
public Page<Entity> geoRadius(String key, Double latitude, Double longitude, Integer distance, String sort, Integer pageNo, Integer pageSize) {  
GeoOperations geoOperations = redisTemplate.opsForGeo();  
Circle circle = new Circle(new Point(latitude, longitude), new Distance(distance / 1000, RedisGeoCommands.DistanceUnit.KILOMETERS));  
RedisGeoCommands.GeoRadiusCommandArgs geoRadiusCommandArgs = RedisGeoCommands.GeoRadiusCommandArgs.newGeoRadiusArgs();  
geoRadiusCommandArgs = geoRadiusCommandArgs.includeCoordinates().includeDistance();  
if ("DESC".equals(sort)) {  
geoRadiusCommandArgs.sortDescending();  
} else {  
geoRadiusCommandArgs.sortAscending();  
}

GeoResults<RedisGeoCommands.GeoLocation<String>> radiusGeo = geoOperations.geoRadius(key, circle, geoRadiusCommandArgs);  
List<GeoResult<RedisGeoCommands.GeoLocation<String>>> list = radiusGeo.getContent();  
List<Entity> entities = null;  
Integer size = null;  
if (CollectionUtils.isNotEmpty(list)) {

int limit = pageNo * pageSize;  
size = list.size();  
if (limit > size) {  
limit = size;  
}

list = list.subList((pageNo &#8211; 1) * pageSize, limit);  
entities = new ArrayList<>(pageSize);

for (GeoResult<RedisGeoCommands.GeoLocation<String>> geoLocationGeoResult : list) {  
RedisGeoCommands.GeoLocation<String> geoLocationGeoResultContent = geoLocationGeoResult.getContent();

Distance distance1 = geoLocationGeoResult.getDistance();  
double value = distance1.getValue();  
value = (double) Math.round(value * 100) / 100;

String name = geoLocationGeoResultContent.getName();  
Entity entity = gson.fromJson(name, Entity.class);  
entity.setDistance(value);

entities.add(entity);  
}  
}  
Page<entity> page = new Page<>();  
page.setData(entities);  
page.setTotal(size);

return page;  
}

```

这段代码需要说明的是：

1. 参数 key 就是之前放进去数据的 key(这不废话吗？)，然后是当前人的经纬度，多远以内的数据，由近到远排序还是由远到近的排序方式，以及分页数据；  
2. 例子中距离多远以内的，传进来的是米，但是后面返回的是千米，所以除以 1000 转了一次，根据业务而定；  
3. 默认是正序，也就是由近到远，但是也支持由远到近  
4. 如果仅仅是排序，也就是对所有的数据由远到近或者由近到远，那么距离就应该是无穷远，Integer.MAX_VALUE；  
5. 有一个 geoRadiusCommandArgs.limit(); 方法，其实就是取 top N，应该挺常用的，但这次不适合我们的应用  
6. list = list.subList((page &#8211; 1) * pageSize, limit); 的意思是说，只需要取 pageSize 个数据进行反序列化就好了，而上面那个判断，是为了防止最后一页下表越界；  
7. value = (double) Math.round(value * 10) / 10; 数据距离中心点的距离，四舍五入，根据需求就好；  
8. 就是对应的数据反序列化以及组装返回了

经过以上，就可以做一个基于 LBS 的应用了，很简单吧？需要补充说明的是：

1. redis 还有好几个其他的 GeoHash 相关的命令和 API，自行查询文档就好；  
2. redis 这个算法，其实就是基于 GeoHash，关于这个算法已经实现，网上有很多资料，有比较详细的说明，感兴趣的自行查阅相关文档就好；  
3. spring-data-redis 一点几和二点几，你说相关的 API 有区别，有什么区别啊？区别很简单就是一点几的方法，都是 geo 打头，例如 geoAdd、geoRadius 等，而二点几版本直接是：add、radius 等，其实我们根据命令 GeoOperations geoOperations = redisTemplate.opsForGeo(); 得到的肯定是操作 geo 相关的命令，这个时候方法再以 geo 打头不是有点画蛇添足吗？但是这个给我们什么提示呢？我们很多时候，根据 id 查询的是，不用写 entityService.getEntityById()，因为理论上这个时候我们肯定是查询 Entity，所以直接写 entityService.getById()，这就告诉我们命名的时候需要仔细斟酌，见名知意的情况下越短越好。

参考资料：  
1. http://redisdoc.com/geo/index.html 官方 API 是最好的学习资料  
2. https://mygodccl.iteye.com/blog/2374978