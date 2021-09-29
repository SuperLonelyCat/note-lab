#### JVM最佳参数配置

**注：在一个服务器的JVM进程占用的总内存一般建议不超多总内存的80%，总内存包括堆、元空间、堆外内存以及CodeCache等。**

##### 常用参数

|          JVM参数          |     参数说明     | 1c2g | 1c2g  | 4c8g | 8c16g |
| :-----------------------: | :--------------: | :--: | :---: | :--: | :---: |
|           -Xms            |  初始堆内存大小  |  1g  | 2560m |  4g  |  10g  |
|           -Xmx            |  最大堆内存大小  |  1g  | 2560m |  4g  |  10g  |
|           -Xmn            |  新生代空间大小  | 500m | 1200m |  2g  |  5g   |
|     -XX:MetaspaceSize     |  初始元空间大小  | 128m | 256m  | 384m | 512m  |
|   -XX:MaxMetaspaceSize    |  最大元空间大小  | 128m | 256m  | 384m | 512m  |
|  -XX:MaxDirectMemorySize  | 最大堆外内存大小 | 256m | 256m  |  1g  |  1g   |
| -XX:ReservedCodeCacheSize |  CodeCache大小   | 64m  | 128m  | 256m | 256m  |

##### 高级参数

|                   JVM参数                    |                           参数说明                           |
| :------------------------------------------: | :----------------------------------------------------------: |
|        -XX:-OmitStackTraceInFastThrow        |                保留NPE空指针异常的StackTrace                 |
|             -XX:SurvivorRatio=10             | Survivor1:Survivor2:Eden=1:1:10，即一个survivor大小为新生代的1/12 |
|           -XX:+UserConcMarkSweepGC           |                       Old区使用CMS GC                        |
|     -XX:CMSMaxAbortablePrecleanTime=5000     |              CMS GC回收超时时间，避免GC时间太久              |
|        -XX:+CMSClassUnloadingEnabled         | 支持CMS GC时对元空间的Class和ClassLoader进行GC，否则要等FullGC时才卸载Class |
|    -XX:CMSInitiatingOccupancyFraction=80     | Old区达到80%时触发CMS GC，如果不设置，JVM会自适应，效果不好  |
|      -XX:+UseCMSInitiatingOccupancyOnly      | 只以CMSInitiatingOccupancyFraction设定的阈值为准，不加这个参数CMSInitiatingOccupancyFraction不会生效 |
|       -XX:+HeapDumpOnOutOfMemoryError        |                    OOM时自动jmap dump内存                    |
|       -Xloggc:/home/admin/logs/gc.log        |                      OOM时dump内存位置                       |
|             -XX:+PrintGCDetails              |                        打印GC详细信息                        |
|            -XX:+PrintGCDateStamps            |                   将GC时间戳更改为直观时间                   |
|          -XX:+UseGCLogFileRotation           |                      开启GC日志文件轮询                      |
|            -XX:GCLogFileSize=50M             |                      每个GC日志文件容量                      |
|           -XX:NumberOfGCLogFiles=5           |                     保留的GC日志文件个数                     |
|           -Djava.awt.headless=true           | headless模式在没有显示器鼠标和键盘等设备的情况下正常运行，服务端建议开启 |
| -Dsun.net.client.defaultConnectTimeout=10000 |  socket连接超时时间，影响所有基于URLConnection的Socket链接   |
|  -Dsun.net.client.defaultReadTimeout=30000   |   socket读超时时间，影响所有基于URLConnection的Socket链接    |
|       -XX:+ExplicitGCInvokesConcurrent       |           调用System.gc()时触发CMS GC而不是Full GC           |
|      -XX:ParallelGCThreads=${CPU_COUNT}      |                   并发GC线程数（Young GC）                   |
|    -Dfile.encoding=${JAVA_FILE_ENCODING}     |                 文件默认编码，推荐使用UTF-8                  |
|         -XX:+CMSScavengeBeforeRemark         |   执行CMS GC的remark阶段之前先Young GC一次，缩短remark时间   |
