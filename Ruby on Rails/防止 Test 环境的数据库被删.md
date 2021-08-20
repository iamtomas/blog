问题：平常用于跑 UT 的数据库，每次执行 rspec 都会先创建数据最后再清空包括库，但如果设置了 schema ，那么数据库每次被删再被创建的话就会报错

```yml

# test:
#   <<: *default
#   host: 
#   database: 
#   username: 
#   password: 
#   schema_search_path:

```

解决措施： 在 `spec/rails_helper.rb` 下加上

```ruby
ActiveRecord::Migration.maintain_test_schema!
```
