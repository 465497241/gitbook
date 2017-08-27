```
<?php 
	/**
	* 正则获取页面title 
	* 自带判断https/http
	* @author 无邪<m13883121541@163.com>
	* @return array('title'=>title,'url'=>url) 一个关联数组，包含title,url
	*/
	function curlGetTitle($url=""){
		if(!$url)return 'url为空';
		$curl = curl_init();
	    curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
	    curl_setopt($curl, CURLOPT_TIMEOUT, 500); 

	    preg_match("/^(http:\/\/|https:\/\/).*$/",$url,$match);
		if($match['1']!='https://' && $match[1]!='http://'){
			return '这不是一个链接';
		}
		//判断https加ssl验证 ,不验证ssl
		if($match[1]=='https://'){
			curl_setopt($curl, CURLOPT_FOLLOWLOCATION,1);
			curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, false);
	    	curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, false);
		}
	    curl_setopt($curl, CURLOPT_URL, $url);
	    $data = curl_exec($curl);
	    // curl_error();
	    curl_close($curl);  
		$pos = strpos($data,'utf-8');

		if($pos===false){$data = iconv("gbk","utf-8",$data);}
		preg_match("/<title>(.*)<\/title>/i",$data, $title);
		if(empty($title) || !isset($title)){ // title为空
			return '该页面没有title！';
		}
		$res = array('title'=>$title[1],'url'=>$url);
		return $res;

	}
	$url = 'https://baidu.com';
	var_dump(curlGetTitle($url));

```



