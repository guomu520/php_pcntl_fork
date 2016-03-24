# php socket
简单的socket小实例
server 代码
server.php
 ```php
<?php
/*
小实例  socket 服务  主进程负责监控  子 多进程处理请求 支持自动平滑重启
*/

ini_set('display_errors', 1);
error_reporting(E_WARNING | E_ERROR | E_ALL);
$server=stream_socket_server("tcp://0.0.0.0:6666",$error,$errstr);
$pids= array(); // 进程PID数组
$son_max_num=20;//每个子进程接收请求的最大次数  超过20次自动平滑重启   还可以加上信号处理 增加 异常处理
while(1){
   for($i=0;$i<10;$i++){
       if(isset($pids[$i])){
            $son_pid=pcntl_waitpid($pids[$i],$status,WNOHANG);
            if($son_pid!=$pids[$i]){
                continue;
             }
        }
        $pids[$i] = pcntl_fork();
        if($pids[$i]==-1){//创建子进程失败
            unset($pids[$i]);
        }
        if($pids[$i]==0){
            $son_num=0;
            while(1){
              if($son_num>=$son_max_num) exit;
              $conn =@stream_socket_accept($server);
              //增加请求处理超时时间 未增加
              $response="";
              if($conn==false) continue;
              $son_num++;
              $request=fread($conn,1024);//是否接收完 可以自定义处理 这里暂时忽略
              $response.=$request."连接第".$i."个进程";
              fwrite($conn,$response);
              fclose($conn);
           }
        }
   }
}
 ```
 client端代码
 client.php
  ```php
  <?php
ini_set('display_errors', 1);
error_reporting(E_WARNING | E_ERROR | E_ALL);
$fp = stream_socket_client("tcp://127.0.0.1:6666", $errno, $errstr);
if (!$fp) {
    echo "ERROR: $errno - $errstr<br />\n";
} else {
    fwrite($fp, "php");
     echo fread($fp,1024);
    fclose($fp);
}
   ```
   
   启动服务 /home/service/php/bin/php server.php &
   测试服务/home/service/php/bin/php client.php
   或者 telnet 127.0.0.1 6666  测试服务
   
