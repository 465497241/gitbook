```php
/**
 * 
 * 根据两点的经纬度计算距离
 * @author zhengshaozhuo
 * @version 2015-08-10
 */
function calc_distance_by_pos($o_lat, $o_lng, $t_lat, $t_lng){
	$pk = 180 / 3.14169;
	$_a1 = $o_lat / $pk;
	$_a2 = $o_lng / $pk;
	$_b1 = $t_lat / $pk;
	$_b2 = $t_lng / $pk;

	$t1 = cos($_a1) * cos($_a2) * cos($_b1) * cos($_b2);
	$t2 = cos($_a1) * sin($_a2) * cos($_b1) * sin($_b2);
	$t3 = sin($_a1) * sin($_b1);

	$tt = acos($t1 + $t2 + $t3);
	return (int)(6366000 * $tt);
}
```