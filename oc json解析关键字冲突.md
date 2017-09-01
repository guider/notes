`id`是[Objective-C](http://lib.csdn.net/base/objective-c "Objective-C知识库")里的关键字，我们一般用大写的`ID`替换，但是往往服务器给我们的数据是小写的id，这个时候就可以用MJExtension框架里的方法转换一下：

```text
+ (NSDictionary *)mj_replacedKeyFromPropertyName
{
    return @{@"ID": @"id"};
}
```