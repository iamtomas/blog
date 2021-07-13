## 1) Associations

- has_many： 一对一，多对多
- has_one： 一对一
- belongs_to： 一对一
- has_and_belongs_to_many: 多对多

has_and_belongs_to_many 使用场景：博客和标签，存在多对多的关系，这时需要一张关联关系表 blogs_tag

has_many 使用场景：blogs_tags基础上添加自定义字段、回调及属性检查等

