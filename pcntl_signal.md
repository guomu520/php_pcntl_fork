# pcntl_signal
详细介绍下php pcntl_signal信号处理
* PHP的函数无法直接注册到操作系统信号设置中，所以pcntl信号需要依赖tick机制。接受信号用declare或者pcntl_signal_dispatch函数（php5.3版本以上支持pcntl_signal_dispatch）。pcntl_signal的实现原理是，触发信号后先将信号加入一个队列中。然后在PHP的ticks回调函数中不断检查是否有信号，如果有信号就执行PHP中指定的回调函数，如果没有则跳出函数
 
 ```php
<?php
function shutdown()
{
    echo PHP_EOL.'shutdown'.PHP_EOL;
}

register_shutdown_function('shutdown');

//使用ticks需要PHP 4.3.0以上版本 接收信号
//declare(ticks = 1);
//或者用 pcntl_signal_dispatch 接收信号  比上面效率高
//信号处理函数
////下面执行exit 也会调用shutdown函数
function sig_handler($signo)
{
     switch ($signo) {
        case SIGTERM:
            // 处理kill
            echo PHP_EOL.'killsys';
            exit;
            break;
        case SIGHUP:
            //处理SIGHUP信号
            break;
        case SIGINT:
            //处理ctrl+c
            echo PHP_EOL.'ctrl+c';
            exit;
            break;
        default:
            echo PHP_EOL.'other';
            exit;
            // 处理所有其他信号
           break;
     }
}
//安装信号处理器
pcntl_signal(SIGTERM, "sig_handler");
pcntl_signal(SIGHUP,  "sig_handler");
pcntl_signal(SIGINT,  "sig_handler");
while(true){
    //echo posix_getpid();
    sleep(10);//也是一次注册操作如果有信号会立刻返回
    // do something
    //信号处理
    pcntl_signal_dispatch(); // 接收到信号时，调用注册的signalHandler()比declare效率更高 php5.3版本以上支持
}
```


