```php
<?php
namespace app\home\controller;
use think\Controller;
define("APPID", "");
define('SECRET', "");
class WeChat extends Controller {

	//获取用户基本信息
	public function getUser()
	{	
		$appid=APPID;
		$secret=SECRET;
		// access_token	调用接口凭证,这里通过code换取的是一个特殊的网页授权access_token,与基础支持中的access_token（该access_token用于调用其他接口）不同
		// openid	普通用户的标识，对当前公众号唯一
		//避免重复发送授权链接。已经获取用户信息，刷新页面，获得重新返回该页面，导致相同的授权链接再次请求，产生错误。
		//把获取的用户信息保存，每次发送请求前，检查是否已经获取，如果已经获取用户信息，就不再请求。
		//提示errcode":40163,"errmsg":"code been used。说明code被使用过一次了，code只能用一次。
		//判断是否发送获取code连接
		if (input('code')) {
			//获取到code判断code是否已经被使用过
			if (!cache('code_array')){
				//通过code获取access_token
            	$code = input('code');
            	$url = "https://api.weixin.qq.com/sns/oauth2/access_token?appid=$appid&secret=$secret&code=$code&grant_type=authorization_code";
            	$result=$this->request($url);
				if (!$result) {
					return false;
				}
				$arr=json_decode($result,true);
				cache('code_array',$arr,7200);
			
			}
			//获取access_token，openid
			$code_array=cache('code_array');
			$access_token=$code_array['access_token'];
			$openid=$code_array['openid'];
			$url="https://api.weixin.qq.com/sns/userinfo?access_token=$access_token&openid=$openid&lang=zh_CN";
			$result=$this->request($url);
			if (!$result) {
				return false;
			}
            //$arr 用户信息数组
			$arr=json_decode($result,true);			
            return $arr;
			
		}else{
			//发送获取code连接
			$url="https://open.weixin.qq.com/connect/oauth2/authorize?appid=$appid&redirect_uri=*****网站地址*****&response_type=code&scope=snsapi_userinfo&state=1#wechat_redirect";
			//发送GET请求
			header("Location: ".$url);
			exit();//必要
		 }

	
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