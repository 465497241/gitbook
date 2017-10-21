```php
<?php     
    /***
    * 生成excle文件
    ***/
    $time = time(); 
    $filename = date("Y年m月d日h点m分s秒", $time).'问卷数据';
    $rows =  Sp_Looks_Vote::downvote();
    $file = $filename.".csv";
    header('Content-Description: File Transfer');
    header('Content-Type: application/octet-stream');  
    header('Content-Disposition: attachment; filename='.basename($file));
    header('Content-Transfer-Encoding: binary');
    header('Expires: 0');
    header('Cache-Control: must-revalidate, post-check=0, pre-check=0');
    header('Pragma: public');

    $tabletitle .= "购买地址,常常购买,喜欢模特,内容如何,印象如何,是否购买,感觉如何,购买方式,购买方式建议,吸引点,改进建议";

    $conter = iconv('utf-8','gbk',$tabletitle)."\n";

    echo $conter;die;



    /***
    * 生成word文件
    ***/

    header("Content-Type:   application/msword");        
    header("Content-Disposition:   attachment;   filename=doc.doc");        
    header("Pragma:   no-cache");        
    header("Expires:   0");        

    $output    =   '<table border="1" cellspacing="2" cellpadding="2"  width="90%" align="center">';        
    $output   .=   '<tr bgcolor="#cccccc"><td   align="center">图片</td></tr>';        
    $output   .=   '<tr bgcolor="#f6f7fa"><td><span style="color:#FF0000;"><strong>下面是一张图片</strong></span& gt;</td></tr>';        
    $output   .=   '<tr><td align="center"><img src="http://zi.csdn.net/48260_2.gif"></td></tr>';        
    $output   .=   '</table>';  

    echo   $output;   

  ?>
```



