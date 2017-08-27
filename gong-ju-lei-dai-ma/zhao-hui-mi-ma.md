```
<?php

/**
* 找回密码_步骤
** 判断帐号类型
** 手机发送验证码||邮箱发送验证码
** 修改密码 
**/
public function findpass()
{
    $step = I('get.step');
    if($step == 1 || $step == null || !$step){  //step1
        if(IS_POST){
            $account = I('post.account') ? trim(I('post.account')) : '';
            if(!$account) $this->error('帐号不能为空！');
            $M = M('member');
            //验证帐号是否存在
            $Account = $M->field('mtel,email')->where(" mtel = '$account' or email = '$account' ")->select();
            if(!$Account) $this->error('账户未注册');
            $this->assign('account',$account);
            $type = 0;  //账号类型：0空，1手机，-1邮箱
            //分辨帐号类型
            if(preg_match('/^1[3456789]\d{9}$/',$account)) $type = 1;
            if(preg_match('/^[A-Za-z\d]+([-_.][A-Za-z\d]+)*@([A-Za-z\d]+[-.])+[A-Za-z\d]{2,4}$/',$account)) $type = -1;
            //跳转步骤二
            $this->assign('type',$type);
            $this->step = 2;
            $this->display();
        }else{  //!IS_POST
            $this->step = 1;
            $this->display();
        }
    }elseif($step == 2 ){   //step2
        if(IS_POST){
            $account = I('post.account') ? trim(I('post.account')) : '';
            $code = I('post.code') ? trim(I('post.code')) : '';
            if(!$code) $this->error('验证码不能为空！');
            if(!$account) $this->error('帐号不能为空！');
            if($scode = authcode(cookie('code'),'DECODE')){
                $scode = explode('#',$scode);
                if($scode[2] != $account)  $this->error('验证码无效！');
                if(time() - $scode[1] > 5*60)  $this->error('验证码已过期！');
                if($scode[0] != $code) $this->error('验证码错误！');
                //跳转步骤三
                $this->assign('account',$account);
                $this->step = 3;
                $this->display();
            }else  $this->error('验证码无效！');
            
        }else{  //!IS_POST
            $this->error('非法操作！');
        }
    }elseif($step == 3 ){   //step3
        if(IS_POST){
            $account = I('post.account') ? trim(I('post.account')) : '';
            $orgpass = I('post.orgpass') ? trim(I('post.orgpass')) : '';
            $compass = I('post.compass') ? trim(I('post.compass')) : '';
            if(!$account) $this->error('帐号不能为空！');
            if(!$orgpass) $this->error('不能设置空密码！');
            if(!$compass) $this->error('确认密码不能为空！');
            if($compass != $orgpass) $this->error('密码不一致！');
            $M = M('member');
            $oldpass = $M->field('password,mid')->where(" mtel = '$account' or email = '$account' ")->find();
            $data['password'] = md5($compass);
            $data['mid'] = $oldpass['mid'];
            if($data['password'] == $oldpass['password']) $this->error('旧密码与新密码一致');
            if($M->save($data) !== false){
                $this->success('修改密码成功','Index/index');
            }else $this->error('修改密码失败');
        }else{  //!IS_POST
            $this->error('非法操作！');
        }
    }else $this->error('非法操作！');
}

public function sendVeifyCode()
{
    $account = I('post.account') ? trim(I('post.account')) : '';
    if(!$account) $this->error('帐号不能为空！');
    $yzm = I('post.yzm') ? trim(I('post.yzm')) : '';
    $captcha = cookie('captcha');
    //验证码检测
    if(!$yzm || !$captcha) $this->error('验证码不能为空！');
    $captcha = authcode(cookie('captcha'),'DECODE');
    $captchaArr = explode('#',$captcha);
    if(count($captchaArr) != 2 || $captchaArr[0] != $yzm) $this->error('验证码错误！');
    if($captchaArr[1] < time() - 60*5) $this->error('验证码超时！');
    //分辨帐号类型
    $type = 0;  //账号类型：0空，1手机，-1邮箱
    if(preg_match('/^1[3456789]\d{9}$/',$account)) $type = 1;
    if(preg_match('/^[A-Za-z\d]+([-_.][A-Za-z\d]+)*@([A-Za-z\d]+[-.])+[A-Za-z\d]{2,4}$/',$account)) $type = -1;
    //账号是否存在
    $M = M('member');
    if($type > 0){  //手机
        //发送短信验证
        (sendCode($account,$yzm))?$this->success('手机验证码发送成功！'):$this->error('手机验证码发送失败！');
    }elseif($type < 0){ //邮箱
        //发送邮箱验证
        $title = "找回密码（系统邮件，请勿回复）";
        $code = rand(100000,999999);
        $content = "<div id='qm_con_body'><div id='mailContentContainer' class='qmbox qm_con_body_content qqmail_webmail_only' style=''><table style='font-size:14px; font-family:Arial, Helvetica, sans-serif; line-height:170%;' width='100%' cellspacing='0' cellpadding='0' border='0'><tbody><tr><td><p align='left'>亲爱的用户<strong>".$this->userinfo[mname]."</strong>，您好： </p><p align='left'>&nbsp;&nbsp;&nbsp; 验证码</p><p align='left'>&nbsp;&nbsp;&nbsp; <u>".$code."</u></p><p align='left'>有效期为5分钟。如非本人操作，请忽略本邮件。</p></td></tr></tbody></table><br><hr><br>这只是一封系统自动发出的邮件，请不要直接回复。<style type='text/css'>.qmbox style, .qmbox script, .qmbox head, .qmbox link, .qmbox meta {display: none !important;}</style></div></div>";
        if(sendMail($account,$title,$content)){
            $code = authcode($code.'#'.time().'#'.$account, 'ENCODE');
            cookie('captcha',null);
            cookie('code',$code);
            $this->success('邮箱验证码发送成功！');
        }else $this->error('邮箱验证码发送失败！');
    }
}

function sendCode($phone,$yzm){
    $captcha = cookie('captcha');
    if(!$phone) return '手机号码不能为空';
    if(!preg_match('/^1[3456789]\d{9}$/',$phone)) return '手机号码格式不正确';
    if(!$yzm) return '验证码不能为空';
    $captcha = authcode(cookie('captcha'),'DECODE');
    $captchaArr = explode('#',$captcha);
    if(count($captchaArr) != 2 || $captchaArr[0] != $yzm) return '验证码错误';
    if($captchaArr[1] < time() - 60*5) return '验证码超时';

    $code = rand(100000,999999);
    $con = file_get_contents('http://domain.com/api/GetCode.php?tel='.$phone.'&code='.$code);
    if($con == '100'){
        $code = authcode($code.'#'.time().'#'.$phone, 'ENCODE');
        cookie('captcha',null);
        cookie('code',$code);
        return true;
    }else return '发送失败';
}


function sendMail($to, $title, $content) {

    Vendor('PHPMailer.PHPMailerAutoload');     
    $mail = new PHPMailer(); //实例化
    $mail->IsSMTP(); // 启用SMTP
    $mail->Host=C('MAIL_HOST'); //smtp服务器的名称
    $mail->Port = C('SMTP_PORT');  //邮件发送端口
    $mail->SMTPSecure = 'ssl';
    $mail->SMTPAuth = C('MAIL_SMTPAUTH'); //启用smtp认证
    $mail->Username = C('MAIL_USERNAME'); //发件人邮箱名
    $mail->Password = C('MAIL_PASSWORD') ; //邮箱密码
    $mail->From = C('MAIL_FROM'); //发件人地址
    $mail->FromName = C('MAIL_FROMNAME'); //发件人姓名
    $mail->AddAddress($to,"尊敬的客户");
    $mail->WordWrap = 50; //设置每行字符长度
    $mail->IsHTML(C('MAIL_ISHTML')); // 是否HTML格式邮件
    $mail->CharSet=C('MAIL_CHARSET'); //设置邮件编码
    $mail->Subject =$title; //邮件主题
    $mail->Body = $content; //邮件内容
    $mail->AltBody = "这是一个纯文本的身体在非营利的HTML电子邮件客户端"; //邮件正文不支持HTML的备用显示
    $mail->SMTPDebug = 0; // 关闭SMTP调试功能
    // 1 = errors and messages
    // $mail->ErrorInfo;
    // 2 = messages only
    if ($mail->send()) {
        return true;
    } else {
        // return $mail->ErrorInfo;
        return false;
    }
    
}
```



