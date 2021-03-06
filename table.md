## ** 数据库关键字 ** ##
关键字  
|  
|-ID  
|————问题  
|————答案  
|————答案所属类别  
||  
||——报税、CA、认证、开票、退税、个税、汇算清缴、其他问题  


## 预设计关键字 ##

|字段					|类型				|备注				|
|:------------|-----------|---------|
|ID						|INT				|	 				|
|问题					|VARCHAR		||
|答案					|VARCHAR		||
|类别关键字		|VARCHAR		|报税、CA、认证、开票、退税、个税、汇算清缴、其他问题(大类别，用于区分种类，提高识别率)__最好单独建表（数据字典表），增加扩展性__|
|tag(标签)		|VARCHAR		||  


## 详细分析 ##
=================================

1.首先对于问题设计，存在问题，问题答案是多对一还是一对多，还是多对多？
>     个人感觉是更多的情况是对于一个答案有多种问法，其实这个比较符合实际情况与语义，
>     所以get到关于问题存储还有答案存储的方式为：问题答案分开存储，以为相当于是对于  
>     和答案之间的关系是多对一的关系，所以可以直接用外键的方式就好

2.关于用户的交互方式？
初级设想，首先对于整体的管理系统存在的功能是CURD，那么对应四个scenario，分别讨论：
+ Scenario One:查询
	+ 查询内容输入：模糊查找，搜索某个词，可以直接显示相关信息，关键词高亮
	+ 结果显示：搜索结果显方式，直接将所有结果列举出来，然后分页显示
	+ Bonus: 多关键词搜索？待定，自然语言搜索不支持，一个管理系统要什么智能性
	+ 详细结果显示：关于详细结果，点击相应链接之后，将数据显示出来，详细的问答对以  
	及一些必要的标签，必要内容（待定，根据数据库最后表结构决定）  
	+ 界面跳转和后退：
	+ 还有一些细节：细节就是对于异常除了或者是一些特殊功能，异常处理包括：输入空的  
	情况应该默认清空或者不变。
	+ Bonus: 正则搜索，自动切割关键字搜索。
+ Scenario Two:增加
	+ 增加一条记录：正常输入，bonus：导入excel的方式进行批量增加插入。分别输入问  
	题答案并选择类型，以上操作步骤特指正常操作情况，不考虑重复情况
	+ bonus：智能推荐答案和
	+ 特殊情况，考虑到预料库数量过多，所以可以预留搜索功能，当准备插入一条数据的时  
	候，系统会分别对问题和答案进行查询（可选两种模式，是否验证），当查询到存在情况  
	的时候，会提醒用户已经存在相似问题或者答案，让用户决定是否继续插入。
+ Scenario Three:删除
+ Scenario Four:更改
	+ 上述两个功能应该和搜索功能连接起来
	+ 特殊功能，没有，就继续在那个接新的界面直接更改如何？
+ 上述四个场景主要是针对语料库的问答对的管理系统，但是对于其他类别管理格外再说
+ 额外功能
	+ 用户管理系统，管理员功能，添加管理员，删除管理员
	+ 问题种类的管理，更改增加删除种类
	+

## 第一阶段初级设计 ##
===================================

1.初级设计主要是面向最基础功能，后续功能以后再添加  
2.初级设计涉及到基础的建表，尽量以最终目标为准  
3.首先揭晓表结构设计
#### 数据库表结构 ####
------------

##### tags #####
| 字段 | 类型 | 备注 |
| :------------- | :------------- | :------------- |
| id | INT | AI,NN,PK |
| tag_content | VARCHAR | 长度可以稍微短一点 |
| parent_id | INT | 存贮父亲节点的tag_id，为了保证标签层级结构 |

##### questions #####
| 字段 | 类型 | 备注 |
| :------------- | :------------- | :------------- |
| id | INT | AI,NN,PK |
| question_content | VARCHAR | 长度要稍微长一点 |
>  数要是因为`question`和`answer`关系认定为多对一，哪怕比如某个问题答案有很多，亦可以将这些  
>  答案拼接起来，将所谓的多对多转换成多对一，这样在某种程度能够减轻问题的复杂度，同时提供  
>  给客户的使用方式是搜索，搜索问题和搜索答案

##### answers #####
| 字段 | 类型 | 备注 |
| :------------- | :------------- | :------------- |
| id | INT | AI,NN,PK |
| answer_content | VARCHAR | UN,长度要稍微长一点 |

##### rlat_answers_tags #####
| 字段 | 类型 | 备注 |
| :------------- | :------------- | :------------- |
| id | INT | AI,NN,PK |
| answer_id | INT | FK.NN |
| tag_id | INT | FK.NN |
>  关系表，主要是为了保存`answer`和`tag`的关系，一对多的关系，而且只存``叶子节点``的`tag`，  
>  这样可以大量的减少数据冗余已经增加访问速度，这个是个不错的选择  
>  可以随意增加更新删除，同时如果设定FK为`cascade`，能够比较快捷方便的保证数据的级联删除，维护
>  渐变，同时单独求改`tag`名称不会造成数据污染

##### rlat_questions_tags #####
| 字段 | 类型 | 备注 |
| :------------- | :------------- | :------------- |
| id | INT | AI,NN,PK |
| question_id | INT | FK.NN |
| tag_id | INT | FK.NN |
>  关系表，主要是为了保存`question`和`tag`的关系，一对多的关系，而且只存``叶子节点``的`tag`，  
>  这样可以大量的减少数据冗余已经增加访问速度，这个是个不错的选择  
>  可以随意增加更新删除，同时如果设定FK为`cascade`，能够比较快捷方便的保证数据的级联删除，维护
>  渐变，同时单独求改`tag`名称不会造成数据污染

##### rlat_answers_questions #####
| 字段 | 类型 | 备注 |
| :------------- | :------------- | :------------- |
| id | INT | AI,NN,PK |
| answer_id | INT | FK.NN |
| question_id | INT | FK.NN |
>  关系表，主要是为了保存`answer`和`question`的关系，一对多的关系，前提认定，对于答案来说，  
>  答案是唯一固定的，这样可以大量的减少数据冗余已经增加访问速度，这个是个不错的选择  
>  可以随意增加更新删除，同时如果设定FK为`cascade`，能够比较快捷方便的保证数据的级联删除，  
>  维护渐变，`CAUTION:`同时要求用户在使用增加的功能的时候，推荐首先查找答案是否存在，或者  
>  有相似答案存在，若存在，则会直接对该答案的问题列进行更新，否则，如果不存在，直接进行新表  
>  项的增加，程序默认使用保证`answer`的`unique`。如果用户没有查询就添加的话，默认首先查找  
>  answer是否存在，select之后如果不存在，就会新建，存在就更新，但是不保证相似答案的情况，  
>  应该注意  


## 初级设计 工作流程 ##
1.用户登录系统，（暂时不做，用户认证授权功能等待下次增加表user以及log的时候在进行处理）  
2.用户录入，并在标签栏选择标签（包括业务种类和问题类型），点击添加（系统自动处理重复情况）  
3.查找录入以及修改功能待定，逻辑已经确定了  
4.难点，整合原先系统，将基于文件的问答对匹配整合到基于数据库的数据村存贮以及展示
