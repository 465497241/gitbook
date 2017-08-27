```
<?php
namespace Common\Common;
class ImageEdit{

    /**
     * [上传图片并生成缩略图]
     * @param  [type] $imgName [图片提交的表单名称]
     * @param  [type] $dirName [图片上传的二级目录]
     * @param  array  $thumb   
     * [缩略图生成,一个数组,传几个值生成几个图片]
     * @return [type]          [description]
     */
    static public function uploadOne($imgName, $dirName, $thumb = array())
    {
        // 上传LOGO
        if(isset($_FILES[$imgName]) && $_FILES[$imgName]['error'] == 0)
        {
            $ic = C('IMAGE_CONFIG');
            $upload = new \Think\Upload(array(
                'rootPath' => $ic['rootPath'],
                'maxSize' => $ic['maxSize'],
                'exts' => $ic['exts'],
            ));// 实例化上传类
            $upload->savePath = $dirName . '/'; // 图片二级目录的名称
            // 上传时指定一个要上传的图片的名称，否则会把表单中所有的图片都处理，之后再想其他图片时就再找不到图片了
            $info   =   $upload->upload(array($imgName=>$_FILES[$imgName]));
            if(!$info)
            {
                return array(
                    'ok' => 0,
                    'error' => $upload->getError(),
                );
            }
            else
            {
                $ret['ok'] = 1;
                $ret['images'][0] = $logoName = $info[$imgName]['savepath'] . $info[$imgName]['savename'];
                // 判断是否生成缩略图
                if($thumb)
                {
                    $image = new \Think\Image();
                    // 循环生成缩略图
                    foreach ($thumb as $k => $v)
                    {
                        $ret['images'][$k+1] = $info[$imgName]['savepath'] . 'thumb_'.$k.'_' .$info[$imgName]['savename'];
                        // 打开要处理的图片
                        $image->open($ic['rootPath'].$logoName);
                        $image->thumb($v[0], $v[1])->save($ic['rootPath'].$ret['images'][$k+1]);
                    }
                }
                return $ret;
            }
        }
    }
    /*******************************************************
     * 上传图片并生成缩略图使用方法
     * 用法:
     * $ret=uploadOne('logo','Goods',array(
     * 			array(600,600),
     * 			array(300,300),
     * 			array(100,100),
     * 
     * ));
     * 返回值:
     * if($ret['ok']==1){
     * 		$ret['images'][0]; //原图地址
     * 		$ret['images'][1]; //第一个缩略图地址
     * 		$ret['images'][2]; //第二个缩略图地址
     * 		$ret['images'][3]; //第三个缩略图地址
     * }else{
     * 		$this->error=$ret['error'];
     * 		return false;
     * }
     ****************************************************/

    /**
     * [delImage 删除图片函数]
     * @param  [type] $img [图片文件名数组]
     * @return [type]      [description]
     */
    static public function delImage($img){
        foreach ($img as $k => $v) {
             unlink(C('IMAGE_CONFIG')['rootPath'].$v);
        }
    }

    /**
     * [showImage 显示图片函数]
     * @param  [type] $url   [图片名称]
     * @param  string $width [显示宽度]
     * @param  string $heigt [显示高度]
     * @return [type]        [description]
     */
    function showImage($url,$width='',$heigt=''){
    	$pt=C('IMAGE_CONFIG');
    	if($width)
    		$width="width='{$width}'";
    	if($heigt)
    		$height="height='{$height}'";
    	echo "<img $width $height src='{$pt['viewPath']}$url' />";
    }

}
```



