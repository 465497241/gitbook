# 微信获取access_token

```php
<?php
namespace app\home\controller;
use think\Controller;
define("APPID", "");
define('SECRET', "");
class WeChat extends Controller {
    /**
	*获取access_token
	*/
	public function getAccessToken()
	{
		$appid=APPID;
		$secret=SECRET;
		//access_token暂存地址在和入口文件同级目录下
		$token_file='access_token.txt';
		//判断access_token文件是否已经过期，未过期直接返回文件内容
		$life_time=7200;
		if (file_exists($token_file) && time()-filemtime($token_file)<$life_time) {
			return file_get_contents($token_file);
		}

		//若是文件过期根据URL重新获取access_token
		$url="https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=$appid&secret=$secret";
		//发送GET请求
		$result=$this->request($url);
		if (!$result) {
			return false;
		}
		//获取响应返回结果
		$arr=json_decode($result,true);
		//将access_token保存在文件中
		file_put_contents($token_file, $arr['access_token']);
		return $arr['access_token'];
	}

	/*
	*发送GET请求方法
	*@param string $url URL
	*@param bool $ssl  是否为https协议
	*@return string   响应主体内容 
	*/

	private function request($url,$data=null){
		$curl=curl_init();
        curl_setopt($curl, CURLOPT_URL, $url);

        //设定为不验证证书和host
        curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, FALSE);
        curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, FALSE);

        if(!empty($data)){
            curl_setopt($curl, CURLOPT_POST, true);
            curl_setopt($curl, CURLOPT_POSTFIELDS, $data);
        }

        // 将curl_exec()获取的信息以文件流的形式返回，而不是直接输出
        curl_setopt($curl,CURLOPT_RETURNTRANSFER,true);

        $output=curl_exec($curl);
        if (false===$output) {
        	echo "<br/>",curl_error($curl),"<br/>";
        	return false;
        }
        curl_close($curl);
        return $output;
	}
}
```
