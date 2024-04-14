Mac和Linux下记得将解释器设定为`/bin/bash`，由于Nacos在Mac/Linux默认是后台启动模式，修改一下它的bash文件，让它变成前台启动，这样IDEA关闭了Nacos就自动关闭了，否则开发环境下很容易忘记关：

```bash
# 注释掉 nohup $JAVA ${JAVA_OPT} nacos.nacos >> ${BASE_DIR}/logs/start.out 2>&1 &
# 替换成下面的
$JAVA ${JAVA_OPT} nacos.nacos
```

