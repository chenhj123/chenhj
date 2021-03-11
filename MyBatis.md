# MyBatis

### 原理

1. 读取核心配置文件
2. 根据配置文件生成SqlSessionFactory工厂对象
3. 创建SqlSession
4. 调用Executor执行数据库操作&&生成具体SQL指令
5. 对查询结果二次封装
6. 提交与事务

