# Chatito

## 简介
Chatito可以根据定义好的DSL（领域定义语言）文件`.chatito`来生成对话机器人的训练测试数据集。

## 安装
Chatito支持`Node.js v8.11.2 LTS`或更高的版本  
`
npm i chatitio --save
`

## 如何编辑Chatito文件
###示例：  
```
%[greet]('training': '2')
    ~[hi] @[name?] ~[whatsUp?]

~[hi]
    hi
    hey

@[name]
    Janis
    Bob

~[whatsUp]
    whats up
    how is it going
```
这个示例的目的是为`greet`这个意图生成两个训练样本。

共有三种实体，`intent`、`slot`、`alias(别名)`。  

- intent
以`%`开始，是生成预料的起点也是生成对话的标签，不允许有重复的`intent`。
- slot
以`@`开始。一个`slot`可以包含多个`alias`。
- alias
以`~`开始。

### 在Rasa中的使用：
```
%[some intent]('training': '1')
    @[some slot]

@[some slot]
    ~[some slot synonyms]

~[some slot synonyms]('synonym': 'true')
    synonym 1
    synonym 2
```
Rasa适配器的一个特定行为是，当`slot`定义语句仅包含一个`alias`时，生成的Rasa数据集将会把这个别名映射为同义词。
在示例中，生成的Rasa数据集将包含`synonym 1`和`synonym 2`的`entity_synonyms`映射到`some slot synonyms`。

## 使用Chatito生成Rasa数据集
trainBot.chatito
```
%[request_search]('training': '420')
    ~[greet?] ~[please?] ~[find?] @[time?] @[item] ~[how_much?]
    ~[greet?] ~[please?] ~[find?] @[time?] @[phone_number] ~[how_much?]
    @[phone_number]

%[request_management]('training': '200')
    ~[greet?] ~[please?] ~[manage] @[price?] @[data_plan?] @[package] 

%[inform_time]('training': '100')
    ~[是?] @[time] ~[的?] ~[mood?]]

%[inform_item]('training': '30')
    ~[是?] @[time] ~[mood?]

%[inform_package]('training': '30')
    @[package] ~[number] ~[mood?]

%[greet]('training': '30')
    ~[greet] ~[greet?]

%[goodbye]('training': '10')
    ~[goodbye]

%[confirm]('training': '14')
    ~[confirm]


@[item]
    ~[流量]
    ~[余额]
    ~[消费]
    ~[话费]
    ~[钱]

@[time]
    ~[上个月]
    ~[这个月]
    ~[本月]
    ~[当前月]
    ~[上上个月]
    ~[一月]
    ~[二月]
    ~[三月]
    ~[四月]
    ~[五月]
    ~[六月]
    ~[七月]
    ~[八月]
    ~[九月]
    ~[十月]
    ~[十一月]
    ~[十二月]

@[phone_number]
    ~[18006208658]
    ~[18810397783]
    ~[18912936976]
    ~[18045905302]
    ~[17501667971]
    ~[18506107025]

@[data_plan]
    ~[流量]

@[package]
    ~[套餐]
    ~[业务]

@[price]
    ~[便宜]
    ~[合适]
    ~[昂贵]
    ~[适中]
    ~[超贵]
    ~[二十八]
    ~[三十]
    ~[六十]

~[greet]
    你好
    您好
    嗨
    喂
    在吗
    hi
    hello
    hey
    你好吗
    您好吗
    你是谁
    你是
    您是
    你是哪个
    您是哪个
    哪个    

~[please]
    请帮我
    请
    请你
    请您
    你能不能
    能不能
    能否
    给我
    我想
    帮我
    你能给我
    你给我

~[find]
    看看
    查查
    查一查
    看一看
    查下
    看一下
    问一下
    知道
    看下
    查一下
    问

~[how_much]
    有多少
    剩下多少
    多少流量
    多少话费
    多少
    多少钱 

~[manage]
    办理
    办一下
    办个
    开下
    开一个
    办
    开

~[number]
    一
    二
    三
    四

~[mood]
    就行
    可以
    吧
    就好
    就可以

~[goodbye]
    再见
    拜拜
    很高兴和你说话
    拜
    bye
    byebye
    see you
    回见
    下次见
    走你

~[confirm]
    嗯
    好的
    行
    可以的
    恩
    中
    中中
    办
    办吧
    办理
    办理吧
    行的
    没问题
    你看着办

```

运行npm生成器：
`npx chatito trainBot.chatito --format==rasa`

完整的参数如下：
Here is the full npm generator options:
```
npx chatito <pathToFileOrDirectory> --format=<format> --formatOptions=<formatOptions> --outputPath=<outputPath> --trainingFileName=<trainingFileName> --testingFileName=<testingFileName> --defaultDistribution=<defaultDistribution>
```

 - `<pathToFileOrDirectory>` path to a `.chatito` file or a directory that contains chatito files. If it is a directory, will search recursively for all `*.chatito` files inside and use them to generate the dataset. e.g.: `lightsChange.chatito` or `./chatitoFilesFolder`
 - `<format>` Optional. `default`, `rasa`, `luis`, `flair` or `snips`.
 - `<formatOptions>` Optional. Path to a .json file that each adapter optionally can use
 - `<outputPath>` Optional. The directory where to save the generated datasets. Uses the current directory as default.
- `<trainingFileName>` Optional. The name of the generated training dataset file. Do not forget to add a .json extension at the end. Uses `<format>`_dataset_training.json as default file name.
- `<testingFileName>` Optional. The name of the generated testing dataset file. Do not forget to add a .json extension at the end. Uses `<format>`_dataset_testing.json as default file name.
- `<defaultDistribution>` Optional. The default frequency distribution if not defined at the entity level. Defaults to `regular` and can be set to `even`.

- `<autoAliases>` Optional. The generaor behavior when finding an undefined alias. Valid opions are `allow`, `warn`, `restrict`. Defauls to 'allow'.

最后可以得到`rasa_dataset_traning.json`和`rasa_dataset_testing`两个文件。

rasa_dataset_traning.json里面的内容如下:
```
{"rasa_nlu_data":{"regex_features":[],"entity_synonyms":[],"common_examples":[{"text":"嗨 看一看 十月 18810397783 有多少","intent":"request_search","entities":[{"end":8,"entity":"time","start":6,"value":"十月"},{"end":20,"entity":"phone_number","start":9,"value":"18810397783"}]}
```