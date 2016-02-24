# pcntl_signal
详细介绍下php pcntl_signal信号处理
* PHP的函数无法直接注册到操作系统信号设置中，所以pcntl信号需要依赖tick机制。接受信号用declare或者pcntl_signal_dispatch函数（php5.3版本以上支持pcntl_signal_dispatch）。pcntl_signal的实现原理是，触发信号后先将信号加入一个队列中。然后在PHP的ticks回调函数中不断检查是否有信号，如果有信号就执行PHP中指定的回调函数，如果没有则跳出函数
 
 ```php
<?php
        //declare(ticks=1);
        $bWaitFlag = TRUE; // 是否等待进程结束
        $intNum= 3; // 进程总数
        $pids= array(); // 进程PID数组
        while(1){  ///守护进程形式
        for($i= 0; $i<$intNum; $i++)
        {
          if($bWaitFlag)  //是否等待 进程执行完在进行下一次循环
            {
                if(isset($pids[$i])){
                     pcntl_waitpid($pids[$i], $status,WUNTRACED);//查看子进程是否执行完如果没执行完 等待
                     //或者写shell命令查看pid进程是否存在 如果存在直接 continue 不存在继续向下执行
                 }
            }
           $pids[$i] = pcntl_fork();// 产生子进程，而且从当前行之下开试运行代码，而且不继承父进程的数据信息
           switch ($pids[$i]){
                 case '-1'://创建子进程失败
                      echo"couldn't fork". "\n";
                      break;
                 case  '0'://创建子进程成功 并且代表此进程为子进程
                      echo"\n"."第".$i."子个进程 -> ". time(). "\n";
                      sleep(10);
                      exit(0);//子进程要exit否则会进行递归多进程，父进程不要exit否则终止多进程
                      break;
                 default://代表此进程为主进程
                      echo $pids[$i]."parent"."$i -> " . time(). "\n";
                      break;
           }

        }
}
```


