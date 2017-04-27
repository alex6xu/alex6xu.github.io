Python ORM 之间对比 对于在文章里提到的每一种 Python ORM ，我们来列一下他们的优缺点：  SQLObject  优点：
采用了易懂的ActiveRecord 模式  一个相对较小的代码库  缺点： 方法和类的命名遵循了Java 的小驼峰风格
不支持数据库session隔离工作单元  Storm  优点： 清爽轻量的API，短学习曲线和长期可维护性  不需要特殊的类构造函数，也没有必要的基类
缺点： 迫使程序员手工写表格创建的DDL语句，而不是从模型类自动派生  Storm的贡献者必须把他们的贡献的版权给Canonical公司  Django's
ORM  优点： 易用，学习曲线短  和Django紧密集合，用Django时使用约定俗成的方法去操作数据库  缺点：
不好处理复杂的查询，强制开发者回到原生SQL  紧密和Django集成，使得在Django环境外很难使用  peewee  优点：
Django式的API，使其易用  轻量实现，很容易和任意web框架集成  缺点： 不支持自动化 schema 迁移  多对多查询写起来不直观
SQLAlchemy  优点： 企业级 API，使得代码有健壮性和适应性  灵活的设计，使得能轻松写复杂查询  缺点： 工作单元概念不常见  重量级
API，导致长学习曲线  AndyLam 翻译于 2年前 3人顶 顶 翻译的不错哦!

