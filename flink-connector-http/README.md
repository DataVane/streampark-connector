## flink-connector-http
- 为什么需要 http 异步维表？
> 1. Http LookupTable对于结合 TensorFlow 等算法系统非常有用，这些系统不容易内置 flink。(算法管理更具可控性)
> 2. 现在越来越多的系统支持 Rest api 请求，Http 查表作为 Dimension Table 用例可能是最灵活的方式。
- 底层实现方式
> 采用[Apache-http-client-5](https://hc.apache.org/httpcomponents-client-ga/) 实现的异步`http` 维表

## 🚀 快速上手
```shell
git clone https://github.com/streamxhub/streamx-connector.git
cd streamx-connector/flink-connector-http
# $version_withouttail ： 1.14 1.13 1.12 ; 通过 shim 屏蔽 flink 版本差异
mvn clean install -DskipTests -Dflink.version=$version -Dshim.version=$version_withouttail

```

## 使用须知
**注意，只有 Http Post 方法实现了所有数据类型支持，Http Get 不支持嵌套类型如：map,list,row等。 Http 请求必须返回JSON 样式；Post 请求`Server`端必须支持`application/json`请求样式**

## 🎉 Features
* 异步的http lookuptable
* 支持Get Post 请求，默认自带连接复用
* 支持简单请求限流
* 支持监控请求情况
* 


## 👻 使用

```sql
# 维表定义
# copy_time 为返回数据样例，其他字段除 processtime 外，均为请求参数字段
CREATE TABLE dim (
 test_bigint BIGINT,
 test_decimal DECIMAL(2,2) ,  
 test_float float,  
 test_string string,
 test_row row<a string>, 
 copy_time bigint) WITH (
   'connector' = 'http_table',
   'url' = 'http://127.0.0.1/test',
   'request.method' = 'post'
);

# 源表定义
CREATE TABLE source (
    test_bigint BIGINT,
    test_decimal        DECIMAL(2,2),
    order_time   AS PROCTIME (),
    test_float float,    test_string string,     test_row row<a string>) WITH (
  'connector' = 'datagen'
);

# 使用方式
select 
  * 
from 
  source 
  left join dim FOR SYSTEM_TIME AS OF source.order_time on dim.test_bigint = source.test_bigint 
  and source.test_decimal = dim.test_decimal 
  and source.test_float = dim.test_float 
  and source.test_string = dim.test_string 
  and source.test_row = dim.test_row;
  
# http 服务端接收请求
User-Agent: Apache-HttpAsyncClient/5.1.3 (Java/1.8.0_261)
Content-Length: 319
Content-Type: application/json; charset=UTF-8
Host: 127.0.0.1
Connection: keep-alive

{"test_bigint":-4443229737491929593,"test_decimal":0.87,"test_float":1.3952405E38,"test_string":"406a1d037ceca370306de3b9aa9be285d05bda5853b336d16d217d5e4703101e966008ba6cb979d3b53c14cd512c9f7b9135","test_row":{"a":"a43140c47b71a9b1e2389352c484169fb75c8cd9ccf301fa08e0b60675d3520e1c04dd918331878e9f6960ff9bc8dd11c77c"}}]

# 返回请求体
{"test_bigint":-6708108163846343121,"test_decimal":0.58,"test_float":6.719438E36,"test_string":"1fe27e06816357fb9bbfd0be70e6c02347b0a15e93b75812a47fed32e3f0bbf53a632155575de7612985ed0eb7e3b1bd8a59","copy_time":1,"test_row":{"a":"50b9d825b1eaa37d7e7687857029b1a8d53d0f2d4d94653f7093bbe3695f868e60d9535475f913c9ccf98de6893cee1e1ebc"}}

# 查询结果

+----+----------------------+--------------+-------------------------+--------------------------------+--------------------------------+--------------------------------+----------------------+---------------+--------------------------------+--------------------------------+--------------------------------+----------------------+
| op |          test_bigint | test_decimal |              order_time |                     test_float |                    test_string |                       test_row |         test_bigint0 | test_decimal0 |                    test_float0 |                   test_string0 |                      test_row0 |            copy_time |
+----+----------------------+--------------+-------------------------+--------------------------------+--------------------------------+--------------------------------+----------------------+---------------+--------------------------------+--------------------------------+--------------------------------+----------------------+
| +I |  2278154383963536264 |         0.43 | 2022-04-21 09:52:13.704 |                   2.1050154E38 | 55221d5dee225974b4a30e4b259... | +I[eb30195ac5c656b72b73e092... |  2278154383963536264 |          0.43 |                   2.1050154E38 | 55221d5dee225974b4a30e4b259... | +I[eb30195ac5c656b72b73e092... |                    1 |
| +I |  6168804551817045985 |         0.92 | 2022-04-21 09:52:13.709 |                   2.5027567E38 | cefa9b13dfc042aa0f3920595ed... | +I[e20845eae98fc696579977f2... |  6168804551817045985 |          0.92 |                   2.5027567E38 | cefa9b13dfc042aa0f3920595ed... | +I[e20845eae98fc696579977f2... |                    1 |
| +I |   849041297008809378 |         0.41 | 2022-04-21 09:52:13.713 |                   2.4748103E38 | 21c5a3a68ee5f9a0cfde540f8f7... | +I[98bf4fcbb6b6a1bb7e3bd0a4... |   849041297008809378 |          0.41 |                   2.4748103E38 | 21c5a3a68ee5f9a0cfde540f8f7... | +I[98bf4fcbb6b6a1bb7e3bd0a4... |                    1 |
| +I | -5148624724080265671 |         0.44 | 2022-04-21 09:52:13.716 |                    2.950303E38 | 8051eb7d99bc2eba6a107580146... | +I[950c5f43600752637cf373ae... | -5148624724080265671 |          0.44 |                    2.950303E38 | 8051eb7d99bc2eba6a107580146... | +I[950c5f43600752637cf373ae... |                    1 |
| +I |  5030070719974405429 |         0.21 | 2022-04-21 09:52:13.719 |                    1.409689E38 | 7e0b7dabbc09160c3b7ab62436b... | +I[81b5841437798bbe8ef8d73b... |  5030070719974405429 |          0.21 |                    1.409689E38 | 7e0b7dabbc09160c3b7ab62436b... | +I[81b5841437798bbe8ef8d73b... |   
```

## 连接参数
> 

| 参数名                 | 作用                                                         | 是否必填 | 默认值（样例）        |
| ---------------------- | ------------------------------------------------------------ | -------- | --------------------- |
| connector              | 连接器类型声明                                               | 是       | http_table(唯一值)    |
| url                    | 请求地址                                                     | 是       | http://127.0.0.1/test |
| request.method         | 声明请求类型                                                 | 是       | get , post 两者之一   |
| request.timeout        | 超时配置                                                     | 否       | 30000         ( ms)   |
| request.retry.max      | 失败重试次数上限                                             | 否       | 3                     |
| request.max.interval   | 重试间隙                                                     | 否       | 1000           ( ms)  |
| request.thread         | 请求线程数                                                   | 否       | 1                     |
| request.limit.gap      | 请求限速相关，最大请求堆积量(非严格限制)                     | 否       | 3000                  |
| request.limit.sleep.ms | 请求限速相关，达到堆积量睡眠时间(初始值，内部会根据堆积情况进行调整) | 否       | 3000         （ms）   |

## 维表使用说明
### 维表对数据类型支持

> 只说明特殊类型

1.  `process time` 字段无法参与`join`。
2.  涉及时间相关字段必须转化为时间戳进行交换(防止时区影响，除非时间格式中带有时区信息)；
3.  时间，Decimal 等字段必须一致才能 `join`，并且字段定义不一致引发的 `join`， 不会报错，但是`join ` 字段数据获取失败。

> ## 非法例子
>
> DECIMAL(32,2) join DECIMAL(2,2) 
>
> timestamp(3) join timestamp(9)
4. **Http post 支持所有数据格式，Get 只支持非嵌套格式(`row ,map, array` 不支持)。**

### 维表不支持 Join 的形式

> 嵌套类型 join 比较特殊，以 row 为例，row 有以下特例

1. 原表中定义的 `row join row` 维表中是支持。

```
# 可以这么使用
select *  from source left join dim FOR SYSTEM_TIME AS OF source.proctime on  dim.test_row= source.test_row
```

2. 任何情况下 维表`row` 部分列 `join` 是不支持。（源表 row 部分列，不涉及复杂嵌套，如`3.`二次使用 row 函数都能正常使用）

> `LookupTableSource` 中`LookupContext` 接口描述是**支持**维表嵌套类型部分 `join`； 但是 `LookupContext` 接口的实现类 1.12 中`CommonLookupJoin.lookupFunction`  **标明只支持 top level join ，即不支持嵌套类型的部分列 join；！！！语法不会报错，但是无法提取 join 字段！！！** 

```
# 不可以这么使用，语法不会报错，但是无法获取 join 列信息
select *  from source left join dim FOR SYSTEM_TIME AS OF source.proctime on  dim.test_row.a = source.test_row.a
# 源表 row 部分列是 join 维表的非 row 嵌套列
# 可以 join 
select *  from source left join dim FOR SYSTEM_TIME AS OF source.proctime on  source.test_row.a = dim.test_string
```

3. `Row` 函数构造的列会引发 bug 

> 类似 bug 报告(row 函数多重嵌套也是有问题的)
>
> https://issues.apache.org/jira/browse/FLINK-20922 

```
# 不可以
select  * from source  left join dim FOR SYSTEM_TIME AS OF source.proctime on dim.test_row.a= source.test_row.a
# 不可以
select  * from source  left join dim FOR SYSTEM_TIME AS OF source.proctime on dim.test_row.a= Row(source.`a`)
```

## Http 维表实现调研
   
- 功能点

1. HTTP 支持异步，性能要足够好，排障问题定位要简单
2. 支持 HTTP Get 和 HTTP Post 请求
3. HTTP 支持连接复用，有自动重试，超时，空闲连接自动释放(异步模式下，存在长期空闲连接，或者是服务端长时空闲主动断开连接，client 端无法感知断开等问题)

4. 有一定请求限流，内存控制能力

5. 保留 http 2.0 支持能力，后续能简单升级支持

#### Http 方案对比

|                                                       | 依赖                                                    | 性能                                                 | 可靠性                                          | 开发难度                                                     | 特性                               |
| ----------------------------------------------------- | ------------------------------------------------------- | ---------------------------------------------------- | ----------------------------------------------- | ------------------------------------------------------------ | ---------------------------------- |
| Netty                                                 | Flink 原生                                              | 一般(没有连接复用下，性能较差；连接管理容易出现问题) | 对新手不友好，原生可靠性需要大量 Netty 知识保障 | 难，底层知识点太多，Netty 学习成本比较高，很多异常情况都需要考虑，受限于 Netty 知识，需要自行解决资源回收，异常处理，连接复用等情况（http 1.1 需要自行实现长连接复用，上述问题都是实际开发遇到的问题）； | 内存限制，内存泄漏检测等高级特性   |
| **[Reactor](https://github.com/reactor/reactor)**     | 底层依赖 netty ，为防止与 Flink 依赖冲突必须 shade 处理 | 好                                                   | 比原生 Netty 更加靠谱                           | 一般，没有实际开发测试过                                     | Netty 拥有的特性都有，更加简单易用 |
| [AsyncHttpClient](https://github.com/AsyncHttpClient) | 底层依赖 netty                                          | 好                                                   | 比原生 Netty 更加靠谱，但是不支持 http 2.0      | 一般，没有实际开发测试过                                     |                                    |
| Apache Http client 5                                  | 外部                                                    | 好                                                   | 没有内存泄漏检测等高级特性，项目比较老          | 较简单，文档比较友好。类似 Netty 的 Reactor 模型，并且自动维护长连接，使用简单，完成大量底层的操作。但缺少限流方案 | 默认 Http 1.1 自带长连接和连接复用 |
| OkHttp                                                | 外部                                                    | 最差                                                 | 安卓大面积使用                                  | 简单                                                         |                                    |

