# Springboot集成elasticsearch使用canal监听

# 一、环境搭建（已有环境可跳过）
## 1.1 centos系统环境
前提设置：<font color='red'>调高JVM线程数限制数量</font>
在centos窗口中，修改配置sysctl.conf
```bash
vim /etc/sysctl.conf
```
在最后一行加入如下内容：
```bash
vm.max_map_count=262144 
```
退出保存文件后，启用配置：
```bash
sysctl -p
```
*注：这一步是为了防止启动容器时，报出如下错误：
bootstrap checks failed max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]*

<font color='red'>以下的操作请自行打开对应防火墙端口，或者配置安全组规则，本文中不再赘述。</font>


## 1.2 在centos中 创建对应映射目录 /home/software/elasticsearch/data,以及编写/home/software/elasticsearch/config/下es-single.yml，内容如下
```xml
network.bind_host: 0.0.0.0
http.cors.enabled: true
http.cors.allow-origin: "*"
```
*注：是为了解决其实地址可以访问，以及跨域问题*

## 1.3 在centos中 执行如下命令搭建elasticsearch单例实例
```bash
 chmod 777 /home/software/elasticsearch/data
 docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d -p 9210:9200 -p 9310:9300  -e "discovery.type=single-node"
  -v /home/software/elasticsearch/data:/usr/share/elasticsearch/data 
  -v /home/software/elasticsearch/plugin:/usr/share/elasticsearch/plugin
  -v /home/software/elasticsearch/config/es-single.yml:/usr/share/elasticsearch/config/elasticsearch.yml 
  --name es-single elasticsearch:7.9.3
``` 
安装中文分词
```
docker exec -it es-single /bin/bash
cd plugins/
elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.9.3/elasticsearch-analysis-ik-7.9.3.zip
```

## 1.4 搭建Mysql
准备/home/mysql/conf/my.cnf
开启bin日志
```
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
[mysqld]
log-bin=/var/lib/mysql/mysql-bin
binlog-format=ROW
server_id=5
```

```
docker run -p 3306:3306 --name mysql -v /home/mysql/data:/var/lib/mysql  -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```

注： server_id与canal的server_id请勿冲突
使用命令show variables like 'log_%';进行查看，为ON表明binlog开启

### 1.4.1 创建canal账号
```
CREATE USER canal IDENTIFIED BY 'canal';  
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
FLUSH PRIVILEGES;
```
将配置文件拷入容器中,并重启
```
docker cp /home/mysql/conf/my.cnf mysql:/etc/mysql/my.cnf
docker restart mysql
```


## 1.5 搭建canal
将数据库连接到canal
```
docker run --name mycanal -e canal.auto.scan=false  -e canal.destinations=test  -e canal.instance.master.address=db:3306   -e canal.instance.dbUsername=canal   -e canal.instance.dbPassword=canal    -e canal.instance.connectionCharset=UTF-8  -e canal.instance.tsdb.enable=true  -e canal.instance.gtidon=false    -e canal.admin.port=11110 -e canal.admin.user=admin -e canal.admin.passwd=4ACFE3202A5FF5CF467898FC58AAB1D615029441  -p 11112:11112  -p 11111:11111 -p 11110:11110 --link mysql:db   -d canal/canal-server:v1.1.4
```
注： canal.instance.master.address=db:3306 可以选填外网数据库地址：IP:3306

## 1.6 搭建canal-dapter
下载地址 https://github.com/alibaba/canal/releases/tag/canal-1.1.4
由于这里使用elasticsearch使用的7.9.3版本，但现有canal-adapter版本无法对应上

## 1.7 创建表
在上方数据库中 创建表
```
CREATE TABLE `shop-mini`  (
  `id` mediumint(11) NOT NULL AUTO_INCREMENT COMMENT '商品id',
  `mer_id` int(10) UNSIGNED NOT NULL DEFAULT 0 COMMENT '商户Id(0为总后台管理员创建,不为0的时候是商户后台创建)',
  `image` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '商品图片',
  `slider_image` varchar(2000) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '轮播图',
  `store_name` varchar(128) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '商品名称',
  `store_info` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '商品简介',
  `keyword` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '关键字',
  `bar_code` varchar(15) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '' COMMENT '商品条码（一维码）',
  `cate_id` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '分类id',
  `price` decimal(8, 2) UNSIGNED NOT NULL DEFAULT 0.00 COMMENT '商品价格',
  `vip_price` decimal(8, 2) UNSIGNED NOT NULL DEFAULT 0.00 COMMENT '会员价格',
  `ot_price` decimal(8, 2) UNSIGNED NOT NULL DEFAULT 0.00 COMMENT '市场价',
  `postage` decimal(8, 2) UNSIGNED NOT NULL DEFAULT 0.00 COMMENT '邮费',
  `unit_name` varchar(32) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '单位名',
  `sort` smallint(11) NOT NULL DEFAULT 0 COMMENT '排序',
  `sales` mediumint(11) UNSIGNED NOT NULL DEFAULT 0 COMMENT '销量',
  `stock` mediumint(11) UNSIGNED NOT NULL DEFAULT 0 COMMENT '库存',
  `is_show` tinyint(1) NOT NULL DEFAULT 1 COMMENT '状态（0：未上架，1：上架）',
  `is_hot` tinyint(1) NOT NULL DEFAULT 0 COMMENT '是否热卖',
  `is_benefit` tinyint(1) NOT NULL DEFAULT 0 COMMENT '是否优惠',
  `is_best` tinyint(1) NOT NULL DEFAULT 0 COMMENT '是否精品',
  `is_new` tinyint(1) NOT NULL DEFAULT 0 COMMENT '是否新品',
  `add_time` int(11) UNSIGNED NOT NULL COMMENT '添加时间',
  `is_postage` tinyint(1) UNSIGNED NOT NULL DEFAULT 0 COMMENT '是否包邮',
  `is_del` tinyint(1) UNSIGNED NOT NULL DEFAULT 0 COMMENT '是否删除',
  `mer_use` tinyint(1) UNSIGNED NOT NULL DEFAULT 0 COMMENT '商户是否代理 0不可代理1可代理',
  `give_integral` int(11) DEFAULT 0 COMMENT '获得积分',
  `cost` decimal(8, 2) UNSIGNED NOT NULL DEFAULT 0.00 COMMENT '成本价',
  `is_seckill` tinyint(1) UNSIGNED NOT NULL DEFAULT 0 COMMENT '秒杀状态 0 未开启 1已开启',
  `is_bargain` tinyint(1) UNSIGNED DEFAULT NULL COMMENT '砍价状态 0未开启 1开启',
  `is_good` tinyint(1) NOT NULL DEFAULT 0 COMMENT '是否优品推荐',
  `is_sub` tinyint(1) NOT NULL DEFAULT 0 COMMENT '是否单独分佣',
  `ficti` mediumint(11) DEFAULT 100 COMMENT '虚拟销量',
  `browse` int(11) DEFAULT 0 COMMENT '浏览量',
  `code_path` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '' COMMENT '商品二维码地址(用户小程序海报)',
  `soure_link` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT '' COMMENT '淘宝京东1688类型',
  `video_link` varchar(200) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '' COMMENT '主图视频链接',
  `temp_id` int(11) NOT NULL DEFAULT 1 COMMENT '运费模板ID',
  `spec_type` tinyint(1) NOT NULL DEFAULT 0 COMMENT '规格 0单 1多',
  `activity` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '' COMMENT '活动显示排序0=默认, 1=秒杀，2=砍价，3=拼团',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
```


# 二 canal-adapter源码构建
https://zhuanlan.zhihu.com/p/355162399
将文中版本pom替换7.9.3 构建后替换github下载的client-adapter\launcher\target\canal-adapter\plugin的client-adapter.elasticsearch-1.1.4-jar-with-dependencies.jar

启动程序


# 三 springboot项目集成
pom.xml
```

    <properties>
        <java.version>1.8</java.version>
        <!--  本次 测试使用的elasticsearch是7.9.3      -->
        <elasticsearch.version>7.9.3</elasticsearch.version>
    </properties>

      <!--springboot的elasticsearch服务-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        </dependency>
```

store-product-setting.json
```
{
  "number_of_shards": "3",
  "number_of_replicas": "0",
  "analysis": {
    "analyzer": {
      "ik_en_analyzer": {
        "type": "custom",
        "tokenizer": "ik_max_word",
        "filter": [
          "my_pinyin"
        ]
      }
    },
    "filter": {
      "my_pinyin": {
        "type": "pinyin",
        "keep_separate_first_letter": false,
        "keep_full_pinyin": true,
        "keep_original": true,
        "limit_first_letter_length": 16,
        "lowercase": true,
        "remove_duplicated_term": true
      }
    }
  }
}

```

store-product-mapping.json
```
{
  "properties": {
    "id": {
      "type": "long"
    },
    "merId": {
      "type": "integer"
    },
    "store_name": {
      "type": "text",
      "analyzer": "ik_en_analyzer"
    },
    "store_info": {
      "type": "text",
      "analyzer": "ik_en_analyzer"
    },
    "keyword": {
      "type": "text",
      "analyzer": "ik_en_analyzer"
    },
    "image": {
      "type": "text"
    },
    "slider_image": {
      "type": "text"
    },
    "bar_code": {
      "type": "text"
    },
    "cate_id": {
      "type": "text"
    },
    "price": {
      "type": "float"
    },
    "vip_price": {
      "type": "float"
    },
    "ot_price": {
      "type": "float"
    },
    "postage": {
      "type": "float"
    },
    "unit_name": {
      "type": "text"
    },
    "sort": {
      "type": "integer"
    },
    "sales": {
      "type": "integer"
    },
    "stock": {
      "type": "integer"
    },
    "is_show": {
      "type": "boolean"
    },
    "is_hot": {
      "type": "boolean"
    },
    "is_benefit": {
      "type": "boolean"
    },
    "is_best": {
      "type": "boolean"
    },
    "is_new": {
      "type": "boolean"
    },
    "add_time": {
      "type": "integer"
    },
    "is_postage": {
      "type": "boolean"
    },
    "is_del": {
      "type": "boolean"
    },
    "mer_use": {
      "type": "boolean"
    },
    "give_integral": {
      "type": "integer"
    },
    "cost": {
      "type": "float"
    },
    "is_seckill": {
      "type": "boolean"
    },
    "is_bargain": {
      "type": "boolean"
    },
    "is_good": {
      "type": "boolean"
    },
    "is_sub": {
      "type": "boolean"
    },
    "ficti": {
      "type": "integer"
    },
    "browse": {
      "type": "integer"
    },
    "code_path": {
      "type": "text"
    },
    "soure_link": {
      "type": "text"
    },
    "video_link": {
      "type": "text"
    },
    "temp_id": {
      "type": "integer"
    },
    "spec_type": {
      "type": "boolean"
    },
    "activity": {
      "type": "text"
    }
  }
}

```


```
package com.elk.elktcp.entity;

import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.*;

import java.io.Serializable;
import java.math.BigDecimal;

@Document(indexName = "store-product")
@Mapping(mappingPath = "/json/store-product-mapping.json")
@Setting(settingPath = "/json/store-product-setting.json")
public class StoreProduct1 implements Serializable {

    @Id
    private Integer id;

    @Field(name="mer_id")
    private Integer merId;

    @Field(name="image")
    private String image;

    @Field(name="slider_image")
    private String sliderImage;

    @Field(name="store_name")
    private String storeName;

    @Field(name="store_info")
    private String storeInfo;

    @Field(name="keyword")
    private String keyword;

    @Field(name="bar_code")
    private String barCode;

    @Field(name="cate_id")
    private String cateId;

    @Field(name="price")
    private BigDecimal price;

    @Field(name="vip_price")
    private BigDecimal vipPrice;

    @Field(name="ot_price")
    private BigDecimal otPrice;

    @Field(name="postage")
    private BigDecimal postage;

    @Field(name="mer_id")
    private String unitName;

    @Field(name="sort")
    private Integer sort;

    @Field(name="sales")
    private Integer sales;

    @Field(name="stock")
    private Integer stock;

    @Field(name="is_show")
    private Boolean isShow;

    @Field(name="is_hot")
    private Boolean isHot;

    @Field(name="is_benefit")
    private Boolean isBenefit;

    @Field(name="is_best")
    private Boolean isBest;

    @Field(name="is_new")
    private Boolean isNew;

    @Field(name="add_time")
    private Integer addTime;

    @Field(name="mis_postage")
    private Boolean isPostage;

    @Field(name="id_del")
    private Boolean isDel;

    @Field(name="mer_use")
    private Boolean merUse;

    @Field(name="give_integral")
    private Integer giveIntegral;

    @Field(name="cost")
    private BigDecimal cost;

    @Field(name="is_seckill")
    private Boolean isSeckill;

    @Field(name="is_bargain")
    private Boolean isBargain;

    @Field(name="is_good")
    private Boolean isGood;

    @Field(name="is_sub")
    private Boolean isSub;

    @Field(name="ficti")
    private Integer ficti;

    @Field(name="browse")
    private Integer browse;

    @Field(name="code_path")
    private String codePath;

    @Field(name="mer_id")
    private String soureLink;

    @Field(name="video_link")
    private String videoLink;

    @Field(name="temp_id")
    private Integer tempId;

    @Field(name="spec_type")
    private Boolean specType;

    @Field(name="activity")
    private String activity;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public Integer getMerId() {
        return merId;
    }

    public void setMerId(Integer merId) {
        this.merId = merId;
    }

    public String getImage() {
        return image;
    }

    public void setImage(String image) {
        this.image = image;
    }

    public String getSliderImage() {
        return sliderImage;
    }

    public void setSliderImage(String sliderImage) {
        this.sliderImage = sliderImage;
    }

    public String getStoreName() {
        return storeName;
    }

    public void setStoreName(String storeName) {
        this.storeName = storeName;
    }

    public String getStoreInfo() {
        return storeInfo;
    }

    public void setStoreInfo(String storeInfo) {
        this.storeInfo = storeInfo;
    }

    public String getKeyword() {
        return keyword;
    }

    public void setKeyword(String keyword) {
        this.keyword = keyword;
    }

    public String getBarCode() {
        return barCode;
    }

    public void setBarCode(String barCode) {
        this.barCode = barCode;
    }

    public String getCateId() {
        return cateId;
    }

    public void setCateId(String cateId) {
        this.cateId = cateId;
    }

    public BigDecimal getPrice() {
        return price;
    }

    public void setPrice(BigDecimal price) {
        this.price = price;
    }

    public BigDecimal getVipPrice() {
        return vipPrice;
    }

    public void setVipPrice(BigDecimal vipPrice) {
        this.vipPrice = vipPrice;
    }

    public BigDecimal getOtPrice() {
        return otPrice;
    }

    public void setOtPrice(BigDecimal otPrice) {
        this.otPrice = otPrice;
    }

    public BigDecimal getPostage() {
        return postage;
    }

    public void setPostage(Boolean postage) {
        isPostage = postage;
    }

    public Boolean getDel() {
        return isDel;
    }

    public void setDel(Boolean del) {
        isDel = del;
    }

    public Boolean getMerUse() {
        return merUse;
    }

    public void setMerUse(Boolean merUse) {
        this.merUse = merUse;
    }

    public Integer getGiveIntegral() {
        return giveIntegral;
    }

    public void setGiveIntegral(Integer giveIntegral) {
        this.giveIntegral = giveIntegral;
    }

    public BigDecimal getCost() {
        return cost;
    }

    public void setCost(BigDecimal cost) {
        this.cost = cost;
    }

    public Boolean getSeckill() {
        return isSeckill;
    }

    public void setSeckill(Boolean seckill) {
        isSeckill = seckill;
    }

    public Boolean getBargain() {
        return isBargain;
    }

    public void setBargain(Boolean bargain) {
        isBargain = bargain;
    }

    public Boolean getGood() {
        return isGood;
    }

    public void setGood(Boolean good) {
        isGood = good;
    }

    public Boolean getSub() {
        return isSub;
    }

    public void setSub(Boolean sub) {
        isSub = sub;
    }

    public Integer getFicti() {
        return ficti;
    }

    public void setFicti(Integer ficti) {
        this.ficti = ficti;
    }

    public Integer getBrowse() {
        return browse;
    }

    public void setBrowse(Integer browse) {
        this.browse = browse;
    }

    public String getCodePath() {
        return codePath;
    }

    public void setCodePath(String codePath) {
        this.codePath = codePath;
    }

    public String getSoureLink() {
        return soureLink;
    }

    public void setSoureLink(String soureLink) {
        this.soureLink = soureLink;
    }

    public String getVideoLink() {
        return videoLink;
    }

    public void setVideoLink(String videoLink) {
        this.videoLink = videoLink;
    }

    public Integer getTempId() {
        return tempId;
    }

    public void setTempId(Integer tempId) {
        this.tempId = tempId;
    }

    public Boolean getSpecType() {
        return specType;
    }

    public void setSpecType(Boolean specType) {
        this.specType = specType;
    }

    public String getActivity() {
        return activity;
    }

    public void setActivity(String activity) {
        this.activity = activity;
    }

    public void setPostage(BigDecimal postage) {
        this.postage = postage;
    }

    public String getUnitName() {
        return unitName;
    }

    public void setUnitName(String unitName) {
        this.unitName = unitName;
    }

    public Integer getSort() {
        return sort;
    }

    public void setSort(Integer sort) {
        this.sort = sort;
    }

    public Integer getSales() {
        return sales;
    }

    public void setSales(Integer sales) {
        this.sales = sales;
    }

    public Integer getStock() {
        return stock;
    }

    public void setStock(Integer stock) {
        this.stock = stock;
    }

    public Boolean getShow() {
        return isShow;
    }

    public void setShow(Boolean show) {
        isShow = show;
    }

    public Boolean getHot() {
        return isHot;
    }

    public void setHot(Boolean hot) {
        isHot = hot;
    }

    public Boolean getBenefit() {
        return isBenefit;
    }

    public void setBenefit(Boolean benefit) {
        isBenefit = benefit;
    }

    public Boolean getBest() {
        return isBest;
    }

    public void setBest(Boolean best) {
        isBest = best;
    }

    public Boolean getNew() {
        return isNew;
    }

    public void setNew(Boolean aNew) {
        isNew = aNew;
    }

    public Integer getAddTime() {
        return addTime;
    }

    public void setAddTime(Integer addTime) {
        this.addTime = addTime;
    }
}

```

启动项目，可以看到elasticsearch可以看到对应索引。

插入数据
```
INSERT INTO `shop-mini`.`eb_store_product`(`id`, `mer_id`, `image`, `slider_image`, `store_name`, `store_info`, `keyword`, `bar_code`, `cate_id`, `price`, `vip_price`, `ot_price`, `postage`, `unit_name`, `sort`, `sales`, `stock`, `is_show`, `is_hot`, `is_benefit`, `is_best`, `is_new`, `add_time`, `is_postage`, `is_del`, `mer_use`, `give_integral`, `cost`, `is_seckill`, `is_bargain`, `is_good`, `is_sub`, `ficti`, `browse`, `code_path`, `soure_link`, `video_link`, `temp_id`, `spec_type`, `activity`) VALUES (1, 0, 'https://img.alicdn.com/imgextra/i1/2089100916/O1CN01REh0cF1IdZS1D9FEU_!!2089100916.jpg', '[\"https://img.alicdn.com/imgextra/i1/2089100916/O1CN01REh0cF1IdZS1D9FEU_!!2089100916.jpg\",\"https://img.alicdn.com/imgextra/i2/2089100916/O1CN01JEpOqu1IdZR5Dwrvq_!!2089100916-0-lubanu-s.jpg\",\"https://img.alicdn.com/imgextra/i3/2089100916/O1CN01sCKNY71IdZR5DyHMG_!!2089100916-0-lubanu-s.jpg\",\"https://img.alicdn.com/imgextra/i1/2089100916/O1CN01OWmqbR1IdZQyjAWLB_!!2089100916-0-lubanu-s.jpg\",\"https://img.alicdn.com/imgextra/i1/2089100916/O1CN01PujBC01IdZR1wPKpq_!!2089100916-0-lubanu-s.jpg\"]', '【24期免息】Dyson戴森吹风机Supersonic HD03紫红色家用护发礼物', '新一代戴森吹风机，快速干发，无需过高温度', '【24期免息】Dyson戴森吹风机Supersonic HD03紫红色家用护发礼物', '', '246,248,258,488,276', 2990.00, 0.00, 3990.00, 0.00, '件', 0, 1, 996, 0, 0, 0, 0, 0, 1612321153, 0, 0, 0, 1000, 3990.00, 0, NULL, 0, 0, 0, 44, '', '', '', 2, 1, '0, 1, 2, 3');
```
然后可以在elasticsearch查询
http://IP:9200/store-product/_search