# php_pcntl_fork
详细介绍下php CGI模式下开始php多线程
* 派生子进程的进程，即父进程，其pid不变；
* 对子进程来说，fork返回给它0,但它的pid绝对不会是0；之所以fork返回0给它，是因为它随时可以调用getpid()来获取自己的pid，而对于父进程来说返回为子进程进程id；
* fork之后，操作系统会复制一个与父进程完全相同的子进程，虽说是父子关系，但是在操作系统看来，他们更像兄弟关系，这2个进程共享代码空间，但是数据空间是互相独立的之间不会相互影响；
* pcntl函数还有一些功能没实验,例如进程状态，根据相应状态进行一些处理，运用多进程可以增加处理数据的效率；
 
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
