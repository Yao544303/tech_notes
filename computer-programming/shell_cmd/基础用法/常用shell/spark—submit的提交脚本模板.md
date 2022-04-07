# spark—submit的提交脚本模板
```
#!/bin/bash
#######################################
#脚本说明：
#这是一个spark—submit的提交脚本模板
#
#Create By: ych
#Email: yao544303963@gmail.com
#######################################
PRO="jiangsu"
DATE="20180114"
MAIN="com.dtwave.dpi.user_dpi_tag"
JAR="lion01181931.jar"
INPUT="/user/vendorszry/ych/dthour/${PRO}/${DATE}*/"
OUTPUT="/user/vendorszry/ych/dt/tag/dpitag/${PRO}/${DATE}"
OUT1="${OUTPUT}/result_00"
OUT2="${OUTPUT}/result_hotel"
OUT3="${OUTPUT}/result_ecomm"

hdfs dfs -rm -r ${OUTPUT}
rm log_dpi_tag.log
nohup spark-submit --master yarn-cluster  --name dpi_tag_${PRO}_${DATE} --driver-memory 6g --executor-memory 6g --queue vendor.ven27  --class ${MAIN} ${JAR}  ${INPUT} ${OUT1} ${OUT2} ${OUT3} /user/vendorszry/ych/dtconfig/applabel_01171730txt  /user/vendorszry/ych/dtconfig/appagesex_01151420.txt   /user/vendorszry/ych/dtconfig/hotel_firstbrand.txt  /user/vendorszry/ych/dtconfig/hotel_secondbrand.txt  /user/vendorszry/ych/dtconfig/dianshang.txt  >> log_dpi_tag.log 2>&1 & 
```