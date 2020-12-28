---
title: php-verificationCode
date: 2017-06-27 00:00:00
tags: ["php"]
categories: ["记录"]
draft: true
---

# 前后端分离，php验证码处理

> 在[自己的项目](https://github.com/yiiouo/todo-vue)，登录注册要用到验证码功能，而这个项目是接后端分离的。因为我是一边学php一边写接口，所以写得不怎么好，继续努力吧。

## 使用GD生成验证码

这个我google了下，看了下代码，很简单就能实现了。

    $string = "abcdefghijklmnopqrstuvwxyz0123456789";
    $str = "";
    
    for($i=0;$i<6;$i++){
      //随机获取验证码中的数字
      $pos = rand(0,35);
      $str .= $string{$pos};
    }
    
    $img_handle = Imagecreate(80, 20);  //图片大小80X20
    $back_color = ImageColorAllocate($img_handle, 255, 255, 255); //背景颜色（白色）
    $txt_color = ImageColorAllocate($img_handle, 0,0, 0);  //文本颜色（黑色）
      
    //加入干扰线
    for($i=0;$i<3;$i++)
    {
      $line = ImageColorAllocate($img_handle,rand(0,255),rand(0,255),rand(0,255));
      Imageline($img_handle, rand(0,15), rand(0,15), rand(100,150),rand(10,50), $line);
    }
    //加入干扰象素
    for($i=0;$i<200;$i++) 
    {
      $randcolor = ImageColorallocate($img_handle,rand(0,255),rand(0,255),rand(0,255));
      Imagesetpixel($img_handle, rand()%100 , rand()%50 , $randcolor);
    }
      
    Imagefill($img_handle, 0, 0, $back_color);             //填充图片背景色
    ImageString($img_handle, 28, 10, 0, $str, $txt_color);//水平填充一行字符串
    header("Content-type: image/jpg"); //生成验证码图片    
    /*
      比如这个接口链接是http://localhost:80/api/code.php，直接在img中的src给予链接，就会显示出验证码了。
     */
    Imagepng($img_handle);//显示图片

因为这是前后端分离的，都是用token进行验证登录状态的，没有用到session（至于为什么，我还没去了解），所以要给一个标识id，知道用户发送的验证码对应的是后台中的哪一个。

## 将图片转成base64，发送给前端

上面GD生成的验证码，如果前端用ajax去获取的话，会一堆乱码，前端要用到`FileReader`对象的`readAsDataURL`方法，将它转成图片。但是这样就获取不了获取到的验证码的标识id了。

所以我想将GD库生成的验证码图片，转成base64，这样就可以组合成个数组，发给前端了。

谷歌了下，找到了方法：
    
    ob_start (); 
      //$img_handle，就是gd生成的图片
      imagejpeg ($img_handle);
      $image_data = ob_get_contents (); 
    
    ob_end_clean (); 
    //生成base64图片
    $image_data_base64 = 'data:image/base64;base64'.base64_encode ($image_data);

这样就能生成了base64的图片，是字符串来的，所以最后再组装下数据结构，就可以发送给前端了：

    //生成此验证码图片的id, uniqid()方法生成验证码的标识id
    echo json_encode(array(
    	'pic'=>$image_data_base64,
    	'id' => uniqid(),
    	'status' => 0
    ))

## 验证码答案和标识id存放在哪里？

这个问题困扰了我，可以存在数据库，txt文件，cookie，缓存。。。还有其他我就不知道了。缓存这个好像是最好的，后面再研究下。然后如果是读写数据库和txt文件的话，感觉会浪费性能，所以先用了cookie去存储，后面再去了解缓存是怎么操作的。
    //id是验证码的标识，pwd是验证码的答案
    setcookie("code", json_encode(array('id'=>$id , 'pwd'=>$pwd)), time()+3600);

要判断的时候获取下cookie再去判断就好了。


# 总结

多尝试点不会的技术吧...