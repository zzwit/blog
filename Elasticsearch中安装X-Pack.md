
## Elasticsearch中安装X-Pack

[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/installing-xpack-es.html)

1.可选：如果要在无法访问Internet的计算机上安装X-Pack：

    手动下载X-Pack zip文件： （sha512） https://artifacts.elastic.co/downloads/packs/x-pack/x-pack-6.2.4.zip

    注意
    Elasticsearch，Kibana和Logstash的插件包含在同一个zip文件中。如果您已下载此文件以在其中一个产品上安装X-Pack，则可以重复使用同一文件。

    将zip文件传输到脱机计算机上的临时目录。（不要将文件放在Elasticsearch插件目录中。）

2 .bin/elasticsearch-plugin install从ES_HOEM群集中的每个节点上 运行：

    bin/elasticsearch-plugin 安装x-pack 

    注意
    如果您使用的 是Elasticsearch 的DEB / RPM分发，请使用超级用户权限运行安装。

    插件安装脚本需要直接访问Internet才能下载和安装X-Pack。如果您的服务器无法访问Internet，请指定您下载到临时目录的X-Pack zip文件的位置。

    ```
    bin/elasticsearch-plugin file://path/to/file/x-pack-6.2.4.zip
    ```

3.确认您要授予X-Pack其他权限。



