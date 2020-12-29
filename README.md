# Prometheus+Grafana监控部署

### 仓库目录
├── build.sh<br>
├── conf<br>
│   ├── alert<br>
│   │   └── alertmanager.yml -- alertmanager配置文件，配置报警通知接收者<br>
│   ├── nginx<br>
│   │   ├── conf.d<br>
│   │   │   └── vhost<br>
│   │   │       └── localhost.conf    -- nginx访问域名配置文件，配置nginx代理转发<br>
│   │   ├── default_passwd<br>
│   │   ├── htpasswd -- nginx访问密码文件<br>
│   │   └── nginx.conf<br>
│   └── prom<br>
│       ├── prometheus.yml    -- prometheus配置文件，配置prometheus job_name（job_name一旦定下则不能修改，否则grafana展示会报single table错）、抓取地址、抓取时间间隔<br>
│       └── rules.yml         -- prometheus监控规则文件，配置指标监控规则，以及指标监控间隔时间<br>
├── data<br>
│   ├── grafana        -- grafana 数据挂载卷<br>
│   ├── nginx          -- nginx网站根目录<br>
│   └── prom           -- prometheus 数据挂载卷<br>
├── docker-compose.yml<br>
├── log  -- nginx日志目录 <br>
└── README.md          -- 部署必看<br>

### 部署步骤
1、安装docker（如果已安装 docker 则跳过）
```
使用一键安装命令
curl -sSL https://get.daocloud.io/docker | sh

如果实在装不了docker，那就用二进制安装
https://www.cnblogs.com/tchua/p/11102462.html
```


2、配置docker镜像源（如果已安装 docker 则跳过）
```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{ "registry-mirrors": ["https://x7s51j3v.mirror.aliyuncs.com"] }
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

3、安装docker-compose（如果已安装 docker-compose 则跳过）
```
#通过pip安装
yum -y install epel-release #安装PIP
yum -y install python-pip
#升级PIP
pip install --upgrade pip

在用户文件夹下新建.pip文件夹及pip.conf文件，写入
mkdir ~/.pip
vi ~/.pip/pip.conf

[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host=mirrors.aliyun.com

# 安装docker-compose
pip install -U docker-compose==1.23.2

```


4、生成docker-compose配置文件
```
cp ./.env.example ./.env
```

5、设置挂载目录权限并启动docker-compose
```
chmod -R 777 ./data && docker-compose up &
```

6、通过nginx代理访问面板,
- 默认账号：admin
- 默认密码：chexiu168
- prometheus:  http://localhost:3001

- grafana:  http://localhost:3002

- alertmanager:  http://localhost:3003

- pushgateway:  http://localhost:3004

7、如果想重新设置nginx密码
```
    # htpasswd -cm 生成的文件位置 用户名
    htpasswd -cm ./conf/nginx/htpasswd admin
    docker restart nginx容器
```

8、进入grafana面板 http://localhost:3002
- 默认账号：admin
- 默认密码：chexiu168
   
   ![Image text](img/1609157323(1).jpg)
   
   ![Image text](img/1609157350(1).jpg)
   
   prometheus地址：http://172.23.0.4:9090
   
   ![Image text](img/1609157491(1).jpg)
   
   ![Image text](img/1609157525(1).jpg)
   
   ![Image text](img/1609157539(1).jpg)
   
   node_export 编号 8919
   
   ![Image text](img/1609157550(1).jpg)
   
   ![Image text](img/1609157568(1).jpg)
   
   ![Image text](img/1609157591(1).jpg)
   
   ![Image text](img/1609157611(1).jpg)
   
   右上角可以设置图表时间范围
   
   ![Image text](img/1609158624(1).jpg)
   

## Note:
- 默认本仓库的docker-compose中已经启动了node_export监控主机，
如果要监控其他软件，可以参照以下命令，启动docker（注意修改命令行中的连接信息），
再修改conf/prom/prometheus.yml配置要抓取的target节点地址，重启prometheus容器，
即可被prometheus抓取（在prometheus面板中可以看到当前抓取的节点）

- 启动node export， dashboard 编号 8919
```
docker run -d --restart=always --name node-exporter -p 9100:9100  prom/node-exporter
``` 

- 启动mysql export， dashboard 编号 7362
```
docker run -d --restart=always --name mysqld-exporter -p 9104:9104   -e DATA_SOURCE_NAME="root:123456@(192.168.33.10:3306)/"   prom/mysqld-exporter
``` 

- 启动redis export，dashboard 编号 9338
```
docker run -d --restart=always --name redis_exporter -p 9121:9121 oliver006/redis_exporter --redis.addr=redis://192.168.33.10:6379
```

- 启动nginx export，这个需要nginx配置如下 dashboard 编号 12708
location = /stub_status {
   stub_status;
}
```
docker run --restart=always -d -p 9113:9113 nginx/nginx-prometheus-exporter:0.8.0 -nginx.scrape-uri http://192.168.33.10/stub_status
```

- 启动elasticsearch export dashboard 编号 6483
```
docker run --restart=always -d -p 9114:9114 justwatch/elasticsearch_exporter:1.1.0 --es.uri=http://elasticsearch:9200
```

- 启动mongodb export  dashboard 编号 2583,不过这个需要修改下json文件，修改后的图表在graph_json/MongoDB-xxx.json中,导入该json即可
```
docker run --restart=always -d -p 9216:9216 fmet/mongodb_exporter:0.10.0 --mongodb.uri=mongodb://10.10.1.178:27017
```

- 启动rabbitmq export  dashboard 编号 4279
```
docker run --restart=always -d -p 9419:9419 -e RABBIT_URL=http://127.0.0.1:15672 -e RABBIT_USER=guest -e RABBIT_PASSWORD=guest kbudde/rabbitmq-exporter
```

- 更多的dashboard可以在 - https://grafana.com/grafana/dashboards 中搜索



