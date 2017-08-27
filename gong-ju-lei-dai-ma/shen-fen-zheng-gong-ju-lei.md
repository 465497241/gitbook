```
<?php
/**
 *身份证号处理函数
 */
 
/**

 * 功能：根据身份证号，自动返回生日
 * @param stirng $IDCard 身份证号
 * @return Ambigous <string, NULL>
 */
function getBrithday ($idCard)
{
    if (strlen($idCard) == 18) {
        $birthday = substr($idCard, 6, 4) . '-' .
                 substr($idCard, 10, 2) . '-' .
                 substr($idCard, 12, 2);
    } elseif (strlen($idCard) == 15) {
        $birthday = "19" . substr($idCard, 6, 2) . '-' .
                 substr($idCard, 8, 2) . '-' .
                 substr($idCard, 10, 2);
    } else {
        $birthday = null;
    }
    
    return $birthday;
}

/**
 * 功能：根据身份证号，自动返回性别
 * @param   string    $cid   身份证号
 * @param   number    $comm  返回的性别类型 
 * @return  string    男/女 1/0
 */
function getSex ($cid, $comm = 0)
{ 
    $cid = getIDCard($cid);
    $sexint = (int) substr($cid, 16, 1);
    if ($comm > 0) {
        return $sexint % 2 === 0 ? '女' : '男';
    } else {
        return $sexint % 2 === 0 ? '0' : '1';
    }
}

/**

 * 功能：把15位身份证转换成18位
 * @param string $idCard            
 * @return newid or id
 */
function getIDCard ($idCard)
{
    // 若是15位，则转换成18位；否则直接返回ID
    if (15 == strlen($idCard)) {
        $w = array(7,9,10,5,8,4,2,1,6,3,7,9,10,5,8,4,2,1);
        $a = array("1","0","X","9","8","7","6","5","4","3","2");
        $s = 0;
        $idCard18 = substr($idCard, 0, 6) . "19" . substr($idCard, 6);  
        $idCard18_len = strlen($idCard18);
        
        for ($i = 0; $i < $idCard18_len; $i ++) {
            $s = $s + substr($idCard18, $i, 1) * $w[$i];
        }        
        return $idCard18.$a[$s % 11];
    } else {
        return $idCard;
    }
}

/**
 * 根据身份证号返回年龄
 * @param string $id 身份证号
 * @return string|number
 */
function getAge($id){
    //过了这年的生日才算多了1周岁
    if(empty($id)) return '';
    $date=strtotime(substr($id,6,8));
    //获得出生年月日的时间戳
    $today=strtotime('today');
    //获得今日的时间戳
    $diff=floor(($today-$date)/86400/365);
    //得到两个日期相差的大体年数
    //strtotime加上这个年数后得到那日的时间戳后与今日的时间戳相比
    $age=strtotime(substr($id,6,8).' +'.$diff.'years')>$today?($diff+1):$diff;
    return $age;
}
```



