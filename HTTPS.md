前排提示：需要准备一个域名

#### 1、安装宝塔面板

（理论上完全不需要宝塔面板这个东西，但是我不会用Linux和Nginx，所以。。。实际操作过程中可以根据具体情况自行选择安不安装）



宝塔面板的安装教程见https://www.bt.cn/bbs/thread-19376-1-1.html ，安装好后，会给你登录网址和默认账号默认密码（此处注意提前放通8888端口以及你可能会用到的端口）

#### 2、安全性设置

成功登陆面板后先进入“面板设置”，修改面板端口（提前放通）、安全入口、面板用户和面板密码

#### 3、安装Nginx

进入“软件商店”，搜索Nginx并安装

#### 4、新建站点

进入“网站”选项，点击添加站点，在域名处填写你的域名，比如我填写的是tf.example.com:81(由于转户口的问题，导致我的服务器无法备案，不能使用443端口，如果想用443端口请自行测试，要用到的端口需要提前放通)，域名填完后什么都不用管，直接提交

#### 5、创建证书

可以使用宝塔面板自带的创建证书功能（点一下你的网站名，再点一下SSL），也可以自己创建证书后导入。

#### 6.修改Nginx配置

点击你的网站名，再点击配置文件，首先把最前面的listen 81改成listen 81 ssl（如果你用的端口不是81，此处的数字也会相应变化，如果用443端口则不需要这一步操作），然后在`#SSL-START SSL相关配置，请勿删除或修改下一行带注释的404规则`这一行之前插入以下代码（注意把域名替换为自己的）

```
    resolver 8.8.8.8;

    location ~ .*
    {
        proxy_set_header Host 127.0.0.1:8501;
        proxy_pass http://127.0.0.1:8501;
        proxy_set_header Accept-Encoding "";
        #使用网页端需要加上以下代码
        if ($request_method = 'OPTIONS') {
				add_header 'Access-Control-Allow-Origin' '*';
				add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
				add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
				add_header 'Access-Control-Max-Age' 1728000;
				add_header 'Content-Type' 'text/plain charset=UTF-8';
				add_header 'Content-Length' 0;
				return 204;
      	}
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Allow-Headers X-Requested-With;
        add_header Access-Control-Allow-Methods GET,POST,OPTIONS;
        #使用网页端需要加上以上代码
        sub_filter "127.0.0.1:8501" "tf.example.com:81";
        sub_filter_once off;
        sub_filter_types *;
    }
```

点击保存。

#### 7、测试

至此，设置完成，你可以通过之前填写的域名访问TensorFlow Serving（Docker也要开着），推荐在https://www.sojson.com/http/test.html 进行测试