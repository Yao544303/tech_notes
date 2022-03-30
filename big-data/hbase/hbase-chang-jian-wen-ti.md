# HBase常见问题

## Hbase 协处理器写es，只考虑写，没考虑写失败的情况，写代码要注意异常情况

* 写失败
* 写错误
* 写一半等

## 协处理器可能有的坑

因协处理器设计不当导致regionserver停工，没法启动，可以先在配置文件hbase-site.xml文件中将hbase.coprocessor.abortonerror设为false，再启动hbase，但此时你无法将协处理器卸载，若要将其卸载，需创建backup-master，创建方法为，新建配置文件：backupmasters，在此文件中键入backup-master的hostname
