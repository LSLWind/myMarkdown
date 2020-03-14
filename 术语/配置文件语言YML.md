## yaml 

yaml 语言（或者说是一种规范吧）可以编写 .yml 文件，和 json 一样是配置文件。也许是有人认为 json 的写法不爽，于是乎发明了这玩意，通过下面的例子，可以看到 yaml 写的配置文件确实要比 json 方便很多。

## 编写规则

- **大小写敏感**

json 里也是大小写敏感的，这点二者一样。

- **使用缩进表示层级关系**

json 中使用 `{}` 的嵌套表示层级，而 yaml 使用缩进，后者更方便一些。

- **`#` 表示注释**

json 文件中不允许写注释，对于很长配置文件全靠字面意思猜挺痛快的，yaml 可以写注释​

## 数据结构

配置文件理应十分简洁，与 json 相比，不用频繁的写 `{}` 和 `[]`，毕竟换行和 `-` 符号更加简洁，字符串也不需要频繁的加引号（无论是单引号还是双引号）。

## 对象

```css
# conf.yml
animal: pets
hash: { name: Steve, foo: bar }
```

转换为 json 为：

```bash
{
    { "animal": "pets" },
    { "hash": { "name": "Steve", "foo": "bar" } }
}
```

## 数组

```bash
# conf.yml
Animal:
 - Cat
 - Dog
 - Goldfish
```

转换为 json 为：

```json
{ "Animal": [ "Cat", "Dog", "Goldfish" ] }
```

## 字符串

```bash
# conf.yml
# 正常情况下字符串不用写引号
str: 这是一行字符串
# 字符串内有空格或者特殊字符时需要加引号
str: '内容： 字符串'
```

## null

```php
# conf.yml
parent: ~
```

.yml 中 ~ 表示 null，转换为 json 为：

```json
{ "parent": null }
```