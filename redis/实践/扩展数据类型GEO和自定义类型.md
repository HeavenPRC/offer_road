基于位置的信息服务（Location-Based Service, LBS）,LBS应用访问的数据是和人或物关联的一组经纬度信息，而且要能查到相邻的经纬度。

hash可以做到很好的储存，但是不擅长做范围查询操作。

Sorted set 权重只能保存一个浮点数，涉及多维度的就不适合保存排序



GeoHash 编码

```
GEOADD cars:locations 116.034579 39.030452 33
```



```
GEORADIUS cars:locations 116.054579 39.030452 5 km ASC COUNT 10
```

### redis还支持自定义的数据类型和操作命令

