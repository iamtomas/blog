## 1) Associations

- has_many： 一对一，多对多
- has_one： 一对一
- belongs_to： 一对一
- has_and_belongs_to_many: 多对多

has_and_belongs_to_many 使用场景：博客和标签，存在多对多的关系，这时需要一张关联关系表 blogs_tag

has_many 使用场景：blogs_tags基础上添加自定义字段、回调及属性检查等

## 2) Model 自定义属性

``` ruby

class Blog < ActiceRecord
  
  def content= one_content
    write_attribute :content, one_content * 2
  end
end

```

注：在类初始化的时候自动执行

## 2) Rspec

``` ruby

expect().to receive().and_return() # 对实例变量的某个方法进行mock

expect(self).to receive_message_chain(:a,:b,:find).and_return() # 对方法链进行mock

```

参考自： https://rspec.info/documentation/

