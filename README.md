# 不用再找了，支付宝自动收取能量、自动偷能量、超级简单的系统教程在这里，华为荣耀V20亲测可用

### 关键信息
作者：狐仙小妲己  
#### [教程地址 ](https://m.lizhiweike.com/channel2/887975)
源码地址：https://github.com/Xiao-DaJi/alipay_autojs

### 相关背景
你还在为自动收取蚂蚁森林能量而发愁吗？狐仙小妲己教大家一个超级的简单的方法来收取能量----使用auto.js收取能量，无需手机root，华为荣耀V20亲测可用。

虽说网上有很多收取能量的教程，但是它们都不够系统，不够全面。直接参考是没办法使用的，今天狐仙小妲己就把踩过的坑做成了一个视频，给大家全面系统的讲解如何使用auto.js收取支付宝蚂蚁森林能量。

一步一步由简入深讲解怎么实现，从auto.js软件的安装下载，到手机的权限设置，再到脚本代码的讲解都有详细的讲解，除此之外其中还穿插了很多爱华为荣耀V20上实操的视频，让DIY学习者更加快速get技巧。

![img]( https://github.com/Xiao-DaJi/alipay_autojs/blob/master/pic/1.png)

### auto.js简介
auto.js其实就是一款安卓的APP，是国内的一个大神的一款APP软件，用来实现自动操作手机，帮忙我们做一些重复的工作，跟按键精灵、脚本精灵一类的软件类似，但是按键精灵和脚本精灵需要手机root才能实现功能，如果你不想或者不知道如何root，那么autojs是你最好的选择。

### 实现流程图

![img]( https://github.com/Xiao-DaJi/alipay_autojs/blob/master/pic/2.png)


![img]( https://github.com/Xiao-DaJi/alipay_autojs/blob/master/pic/%E5%BD%95%E5%88%B6_2020_05_28_13_28_20_709.gif)

### 源码解析
 > ==代码看不懂？这里有视频教程：https://m.lizhiweike.com/channel2/887975 ，加入QQ群后，狐仙小妲己会为你解答疑惑==

```
/* 描述：这是一个autojs自动收集支付宝蚂蚁森林能量的js脚本，实现每天定时收取自己的能量和好友的能量
 * 作者：狐仙小妲己
 * 说明：代码有部分是参考网上写的，大部分是自己实现的
 * 时间：2020/05/17
 * 版本：V1.0
 * github地址：https://github.com/Xiao-DaJi/alipay_autojs.git
 * 教程地址：想要系统的了解整个过程，如何安装autojs，如何在js上跑这个脚本，请参考我的教程https://m.lizhiweike.com/channel2/887975
 */



 //解锁密码，需要自行设置，设置方法参考我的教程
var unlockCode=[
    205,1443, //1
    539,2086, //0
    219,1888, //7
    529,1882, //8
    219,1888, //7
    525,1673 //5
];

var screen_width = device.width;  //设置屏幕的宽度，像素值
var screen_height = device.height; //设置屏幕的高度，像素值


//1、在屏幕上打印“开始自动收集能量”
toast("开始自动收集能量");

//2、解锁屏幕，输入解锁密码
unlock();
sleep(2000);



//3、创立一个线程，截图提示，后台点击立即开始（华为手机申请手机截图权限时，需要手动点击“立即开始”），为了方便，我们开辟一个线程来点击
var thread = threads.start(function() 
{ 
   while (!click("立即开始")); 
});

//4、音量“-”g关闭脚本
threads.start(function(){
    //在子线程中调用observeKey()从而使按键事件处理在子线程执行
    events.observeKey();
    events.on("key_down", function(keyCode, events){
        //音量键关闭脚本
        if(keyCode == keys.volume_down){
            toast("您选择退出脚本！")
            sleep(2000);
            exit();
        }
    });
});

//4、请求截图权限，因为后面的收集能量需要用到手机截图
prepareThings();

//5、关闭3创建的那个线程，后面不要用到就关闭
thread.interrupt();



//进入主程序
mainEntrence();

//程序主入口
function mainEntrence()
{
    var ret = false;
    var step = 0;
    var tryCnt=0;
    var sucessFlag = false;

    //下面几个环节如果出错，运行3次机会，3次失败的话退出去，手机失败
    //while(tryCnt<=3)
    //{
        //sleep(2000);
       // tryCnt++;
    //}

    //如果有1步出错，尝试3次，3次都不成功能就退出
    while(tryCnt <=3)
    {
        switch(step)
        {
            case 0:
                //1、打开支付宝软件
                ret = openAlipay();

                if(ret == true){ 
                    step = 1; 
                    sucessFlag = true;
                }
                else { 
                    step = 3;
                    tryCnt++;
                    sucessFlag = false;
                }
                break;
            case 1:
                //2、进入到蚂蚁森林界面，并收集我自己的能量
                ret = enterMyAntForest();

                if(ret == true){ 
                    step = 2; 
                    sucessFlag = true;
                }
                else { 
                    step = 3;
                    tryCnt++;
                    sucessFlag = false;
                }
                break;
            case 2:
                //3、进入到好友的能量主页
                enterRank();
                enterOthers();
                step = 3;
                sucessFlag = true;
                break;
            case 3:
                //4、收集完成，返回到收集主页面，等待手机自动锁屏
                whenComplete();

                if(sucessFlag == true)
                {
                    tryCnt = 8;//退出while(tryCnt <=3)循环
                }
                else
                {
                    step = 0;
                }
                break;
        }      
    }
    
}

//解锁
function unlock()
{
    //判断当前屏幕是不是亮着的，是亮的话就不解锁了
    if(!device.isScreenOn())
    {
        //点亮屏幕
        device.wakeUp();
        sleep(1000);

        //滑动一下手机，进入到输入解锁密码的界面
        swipe(500, 0, 500, 1900, 1000);
        sleep(1000);

        //输入屏幕解锁密码，其他密码请自行修改
        click(unlockCode[0],unlockCode[1]);
        sleep(500);

        click(unlockCode[2],unlockCode[3]);
        sleep(500);

        click(unlockCode[4],unlockCode[5]);
        sleep(500);

        click(unlockCode[6],unlockCode[7]);
        sleep(500);

        click(unlockCode[8],unlockCode[9]);
        sleep(500);

        click(unlockCode[10],unlockCode[11]);
        sleep(500);
   
    }
}


//打开支付宝
function openAlipay()
{
    launchApp("支付宝");

    toastLog("等待支付宝启动");

    var i=0;

    //等待出现“扫一扫”这个控件
    while (!textEndsWith("扫一扫").exists() && i<=5)
    {
        sleep(2000);
        i++;
    }

    toastLog("第"+i+"次尝试进入支付宝主页");

    if(i>=5)
    {
        toastLog("没有找到支付宝首页");
        sleep(1000);

        //如果10秒还没检测到“扫一扫”出现，可能支付宝在其他页面，比如理财、口碑、朋友等，这时尝试点击“首页”按钮
        clickByTextDesc("首页",0);
        return false;
    }

    return true;
}

//从支付宝主页进入蚂蚁森林我的主页
function enterMyAntForest()
{
    //五次尝试蚂蚁森林入
    var i=0;

    //滑动，确保点击不出错
    swipe(screen_width*0.5,screen_height*0.5,screen_width*0.5,screen_height*0.25,500);
    sleep(500);
    swipe(screen_width*0.5,screen_height*0.25,screen_width*0.5,screen_height*0.5,500);

    //根据控件的text属性来确认支付宝的界面有没有被打开
    while (!textEndsWith("蚂蚁森林").exists() && i<=5)
    {
        sleep(1000);
        i++;   
    } 

    //点击进入蚂蚁森林
    clickByTextDesc("蚂蚁森林",0);
    
    toast("已经进入蚂蚁森林了。。。");

    //等待进入自己的主页,10次尝试
    sleep(3000);

    i=0;
    //现在的支付宝没有地图这个控件，改成通知
    while (!textEndsWith("背包").exists() && !textEndsWith("通知").exists() && i<=10){
        sleep(1000);
        i++;
    }

    toastLog("第"+ i +"次尝试进入自己主页");

    if(i>=10)
    {
        toastLog("进入自己能量主页失败");
        return false;
    }
    else
    {
        toastLog("进入蚂蚁森林页面成功");
    }

    //收自己能量
    //clickByTextDesc("克",0);

    //由于支付宝的更新，需要用这种方法收集能量，之前的API不可用
    for(var row=screen_height*0.256;row<screen_height*0.376;row+=80)
        for(var col=screen_width*0.185;col<screen_width*0.815;col+=80){
            click(col,row);
            }

    toastLog("自己能量收集完成");
    sleep(100);

    return true;
}

//点击控件函数
function clickByTextDesc(energyType, paddingY)
{
    var clicked = false;
    if(descEndsWith(energyType).exists()){
        descEndsWith(energyType).find().forEach(function(pos){
            var posb=pos.bounds();
            if(posb.centerX()<0 || posb.centerY()-paddingY<0){
                return false;
            }
            //toastLog(pos.id());
            var str = pos.id();
            if(str != null){
                if(str.search("search") == -1){
                    click(posb.centerX(),posb.centerY()-paddingY);
                     //toastLog("get it 1");
                     clicked = true;   
                }
            }else{
                click(posb.centerX(),posb.centerY()-paddingY);
                //toastLog("get it 2");
                clicked = true;
            }
            sleep(100);
        });
    }
    
    if(textEndsWith(energyType).exists() && clicked == false){
        textEndsWith(energyType).find().forEach(function(pos){
            var posb=pos.bounds();
            if(posb.centerX()<0 || posb.centerY()-paddingY<0){
                return false;
            }
            //toastLog(pos.id());
            var str = pos.id();
            if(str != null){
                if(str.search("search") == -1){
                    click(posb.centerX(),posb.centerY()-paddingY); 
                    //toastLog("get it 3"); 
                    clicked = true;  
                }
            }else{
                click(posb.centerX(),posb.centerY()-paddingY);
                //toastLog("get it 4");
                clicked = true;
            }
            sleep(100);
        });
    }
    
    return clicked;
}


//进入蚂蚁森林的排行榜
function enterRank()
{
    toastLog("进入排行榜");

    sleep(2000);
    swipe(screen_width*0.5,screen_height*0.8,screen_width*0.5,screen_height*0.1,500);
    sleep(500);
    swipe(screen_width*0.5,screen_height*0.8,screen_width*0.5,screen_height*0.1,500);


    toastLog("查看更多好友");
    sleep(500);

    clickByTextDesc("查看更多好友",0);
       
    //等待排行榜主页出现
    sleep(3000);
    return true;
}


//获取截图
function getCaptureImg()
{ 
    var img0 = captureScreen();

    sleep(100);
    if(img0==null || typeof(img0)=="undifined")
    {
        toastLog("截图失败,脚本退出");
        exit();
    }
    else
    {
       return img0;
    }
}

//从排行榜获取可收集好友的点击位置
function  getHasEnergyfriend(type) 
{
    var img = getCaptureImg();
    var p=null;

    if(type==1)
    {
        // 寻找可以手机能量的好友，有小手的，区分倒计时和可收取能量的小手，这里是核心代码，支付宝更新了或者不同手机分辨需要做些微调，详细方法请参考我的教程
       p = images.findMultiColors(img, "#ffffff",[[0, -35, "#1da06e"],[0, 23, "#1da06e"]], {
            region: [1043,200 , 1, 2010]
        });
    }


    if(p!=null)
    {
        //toastLog("找到了可收取能量的好友");
        return p;
    }
    else 
    {
        //toastLog("此页没有找到可收能量的好友");
        return null;
    }
}

//判断是否收取能量结束
function  canItEnd() 
{
    var img = getCaptureImg();
    var p=null;

    //判断底部是否到了“没有更多了”的了字
     p = images.findMultiColors(img, "#999999",[[0, 4, "#F5F5F5"],[0, 10, "#999999"]], {
        region: [620,2035 , 1, 264]
    });

    if(p!=null)
    {
        return true;
    }
    else 
    {
        return false;
    }

}
//在排行榜页面,循环查找可收集好友
function enterOthers()
{
    var i=1;
    var ePoint=getHasEnergyfriend(1);
    
    //不断滑动，查找好友
    while(ePoint==null)
    {

        swipe(screen_width*0.5,screen_height*0.7,screen_width*0.5,screen_height*0.1,500);
        sleep(2000);

        toastLog("开始查找");
        ePoint=getHasEnergyfriend(1);
        i++;

        //如果连续15次都未检测到可收集好友,无论如何停止查找 
        if(i>3)
        {
            if(canItEnd() == true)
            {
                toastLog("所有好友收集完成");
                return true;
            }
            if(i>15)
            {
                toastLog("程序可能出错,连续"+i+"次未检测到可收集好友");
                return false;
            }
        }
    }
    
    //找到好友
    //进入好友页面,10次尝试
    click(ePoint.x,ePoint.y+20);
    sleep(3000);
    i=0;

    while (!textEndsWith("TA收取你").exists() && !textEndsWith("你收取TA").exists() && i<=10)
    {
        sleep(1000);
        i++;
    }

    toastLog("第"+i+"次尝试进入好友主页");

    if(i>=10)
    {
        toastLog("进入好友能量主页失败");
        return false;
    }
    
    //收能量
    //clickByTextDesc("克",0);

    for(var row=screen_height*0.256;row<screen_height*0.376;row+=80)
        for(var col=screen_width*0.185;col<screen_width*0.815;col+=80){
            click(col,row);
            }

    //等待返回好友排行榜
    back();

    sleep(1000);
    //返回排行榜成功，继续收取剩余好友能量
    enterOthers();

}

function myEnergyTime()
{
    var now =new Date();
    var hour=now.getHours();
    var minu=now.getMinutes();
    var mytime=morningTime.split(":");
    
    if(mytime[0]==hour && (mytime[1]==minu || mytime[1]==minu-1) )
    {
        return true;
    }
    else
    {
        return false;
    }   
}

//获取权限和设置参数
function prepareThings()
{
    //请求截图权限，只需要请求一次
    if(!requestScreenCapture())
    {
        toastLog("请求截图失败,脚本退出");
        exit();
    }

    sleep(3000);
}

//结束后返回主页面
function whenComplete() 
{
    toastLog("结束");
    back();
    sleep(1500);
    back();
    sleep(1500);
    back();
}

var morningTime = "07:18";//自己运动能量生成时间
var startTime = "07:00";
var endTime = "7:35";
```

