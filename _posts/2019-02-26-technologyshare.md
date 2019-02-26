---
layout: post
title: 大数据部技术分享第四期
icon: star-o
---

2019年2月月22日13:00--17:00朱小杰老师和沈志宏老师分享关于“大数据流水线系统piflow0.5”、“融合型图数据管理引擎PiDB0.1”两方面的技术知识。

![](https://github.com/cas-bigdatalab/piflow/blob/master/doc/piflow.png) 是一个简单易用，功能强大的大数据流水线系统，即大数据ETL工具
## 介绍内容

- [Features](#特性)
- [Architecture](#架构)
- [Requirements](#环境要求)
- [Getting Started](#使用)


## 特性

- 简单易用
  - 提供所见即所得页面配置流水线
  - 监控流水线状态
  - 查看流水线日志
  - 检查点功能
- 可扩展性
  - 支持用户自定义开发组件
- 性能优越
  - 基于分布式计算引擎Spark开发
- 功能强大
  - 提供100+数据处理组件
  - 包括 spark、mllib、hadoop、hive、hbase、solr、redis、memcache、elasticSearch、jdbc、mongodb、http、ftp、xml、csv、json等.

## 架构
![](https://github.com/cas-bigdatalab/piflow/blob/master/doc/architecture.png)
## 环境要求
* JDK 1.8 or newer
* Apache Maven 3.1.0 or newer
* Git Client (used during build process by 'bower' plugin)
* spark-2.1.0
* hadoop-2.6.0

## 使用

Build工程:
`git clone https://github.com/cas-bigdatalab/piflow.git`
`mvn clean package -Dmaven.test.skip=true`

          [INFO] Replacing original artifact with shaded artifact.
          [INFO] Replacing /opt/project/piflow/piflow-server/target/piflow-server-0.9.jar with /opt/project/piflow/piflow-server/target/piflow-server-0.9-shaded.jar
          [INFO] ------------------------------------------------------------------------
          [INFO] Reactor Summary:
          [INFO]
          [INFO] piflow-project ..................................... SUCCESS [  4.602 s]
          [INFO] piflow-core ........................................ SUCCESS [ 56.533 s]
          [INFO] piflow-bundle ...................................... SUCCESS [02:15 min]
          [INFO] piflow-server ...................................... SUCCESS [03:01 min]
          [INFO] ------------------------------------------------------------------------
          [INFO] BUILD SUCCESS
          [INFO] ------------------------------------------------------------------------
          [INFO] Total time: 06:18 min
          [INFO] Finished at: 2018-12-24T16:54:16+08:00
          [INFO] Final Memory: 41M/812M
          [INFO] ------------------------------------------------------------------------

运行Piflow Server：
- configure config.properties

      #server ip and port
      server.ip=10.0.86.191
      server.port=8002
      h2.port=50002

      #spark and yarn config
      spark.master=yarn
      spark.deploy.mode=cluster
      yarn.resourcemanager.hostname=10.0.86.191
      yarn.resourcemanager.address=10.0.86.191:8032
      yarn.access.namenode=hdfs://10.0.86.191:9000
      yarn.stagingDir=hdfs://10.0.86.191:9000/tmp/
      yarn.jars=hdfs://10.0.86.191:9000/user/spark/share/lib/*.jar
      yarn.url=http://10.0.86.191:8088/ws/v1/cluster/apps/

      #hive config
      hive.metastore.uris=thrift://10.0.86.191:9083

      #piflow jar path
      piflow.bundle=/opt/piflowServer/piflow-server-0.9.jar

      #checkpoint hdfs path
      checkpoint.path=hdfs://10.0.86.89:9000/piflow/checkpoints/
- you can run piflow server on intellij
  - main class is cn.piflow.api.Main
  - remember to set SPARK_HOME
- you can run piflow server as follows:
  - download piflowServer:***
  - edit config.properties
  - run start.sh

运行Piflow Web：
  - todo

使用方式：

- 命令行
  - flow config example


        {
          "flow":{
          "name":"test",
          "uuid":"1234",
          "checkpoint":"Merge",
          "stops":[
          {
            "uuid":"1111",
            "name":"XmlParser",
            "bundle":"cn.piflow.bundle.xml.XmlParser",
            "properties":{
                "xmlpath":"hdfs://10.0.86.89:9000/xjzhu/dblp.mini.xml",
                "rowTag":"phdthesis"
            }
          },
          {
            "uuid":"2222",
            "name":"SelectField",
            "bundle":"cn.piflow.bundle.common.SelectField",
            "properties":{
                "schema":"title,author,pages"
            }

          },
          {
            "uuid":"3333",
            "name":"PutHiveStreaming",
            "bundle":"cn.piflow.bundle.hive.PutHiveStreaming",
            "properties":{
                "database":"sparktest",
                "table":"dblp_phdthesis"
            }
          },
          {
            "uuid":"4444",
            "name":"CsvParser",
            "bundle":"cn.piflow.bundle.csv.CsvParser",
            "properties":{
                "csvPath":"hdfs://10.0.86.89:9000/xjzhu/phdthesis.csv",
                "header":"false",
                "delimiter":",",
                "schema":"title,author,pages"
            }
          },
          {
            "uuid":"555",
            "name":"Merge",
            "bundle":"cn.piflow.bundle.common.Merge",
            "properties":{
              "inports":"data1,data2"
            }
          },
          {
            "uuid":"666",
            "name":"Fork",
            "bundle":"cn.piflow.bundle.common.Fork",
            "properties":{
              "outports":"out1,out2,out3"
            }
          },
          {
            "uuid":"777",
            "name":"JsonSave",
            "bundle":"cn.piflow.bundle.json.JsonSave",
            "properties":{
              "jsonSavePath":"hdfs://10.0.86.89:9000/xjzhu/phdthesis.json"
            }
          },
          {
            "uuid":"888",
            "name":"CsvSave",
            "bundle":"cn.piflow.bundle.csv.CsvSave",
            "properties":{
              "csvSavePath":"hdfs://10.0.86.89:9000/xjzhu/phdthesis_result.csv",
              "header":"true",
              "delimiter":","
            }
          }
        ],
        "paths":[
          {
            "from":"XmlParser",
            "outport":"",
            "inport":"",
            "to":"SelectField"
          },
          {
            "from":"SelectField",
            "outport":"",
            "inport":"data1",
            "to":"Merge"
          },
          {
            "from":"CsvParser",
            "outport":"",
            "inport":"data2",
            "to":"Merge"
          },
          {
            "from":"Merge",
            "outport":"",
            "inport":"",
            "to":"Fork"
          },
          {
            "from":"Fork",
            "outport":"out1",
            "inport":"",
            "to":"PutHiveStreaming"
          },
          {
            "from":"Fork",
            "outport":"out2",
            "inport":"",
            "to":"JsonSave"
          },
          {
            "from":"Fork",
            "outport":"out3",
            "inport":"",
            "to":"CsvSave"
          }
        ]
      }
    }
  - curl -0 -X POST http://10.0.86.191:8002/flow/start -H "Content-type: application/json" -d 'this is your flow json'
- 界面piflow web
  ![](https://github.com/cas-bigdatalab/piflow/blob/master/doc/piflow_web.png)
