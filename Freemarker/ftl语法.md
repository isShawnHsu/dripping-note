---
html:
    toc: true
---
# freemarker语法
### 一、字符
###### 1. 为空判断
```ftl
// 变量存在，输出该变量，否则不输出
${emp.name?if_exists}

${emp.name??}
```
###### 2.变量可空
```ftl
// 变量存在，输出该变量，否则不输出
${emp.name!}
```
###### 3.变量为空则取默认值
```ftl
// 变量不存在，取默认值xxx
${emp.name?default("xxx")}
${emp.name!"xxx"}
```
###### 4.输出html
```ftl
${"123<br>456"?html}
```

###### 5.字母大小写
```ftl
// 首字母大写
${"str"?cap_first}

// 将字符串转换成小写
${"Str"?lower_case}

// 将字符串转换成大写
${"Str"?upper_case}  
```

###### 6. 去除前后空格
```ftl
${"str"?trim}
```

###### 7.字符串拼接
```ftl
${"你好${emp.name！}"} 
${"hello"+ emp.name！}
```

###### 8.字符串截取
```ftl
${str?substring（0,4）}
```

###### 9.返回字符串所在的下标
```ftl
${str?index_of("n")}
```

### 二、日期
###### 1. 日期格式化
```ftl 
${emp.date?string('yyyy-MM-dd')}
```

### 三、数字（20为例）
###### 1.数字类型
```ftl
//输出 20 
${emp.name?string.number}
```

###### 2.货币类型
```ftl
//输出 ¥20.00 
${emp.name?string.currency}  
```

###### 3.百分比类型
```ftl
//输出 20%
${emp.name?string.percent}
```

###### 4.整数型
```ftl
//将小数转为int，输出1
${1.222?int} 
```

### 四、申明变量
```ftl
<#assign name = value> 
<#assign name1 = value1 name2 = value2 ... nameN = valueN> 
<#assign same as above... in namespacehash> 

<#assign name>
 capture this 
</#assign> 

<#assign name in namespacehash> 
capture this 
</#assign> 
```

### 五、运算符
- `=`
- `!=`
- `>` 或 `gt`
- `>=` 或 `gte`
- `<` 或 `lt`
- `<=` 或 `lte`
- `&&`
- `||`
- `!`
  
### 六、if语法
```ftl
<#if condition??> //判断是否为空
...
<#elseif condition2>
...
<#else>
...
</#if>
```

### 七、switch语法
```ftl
<#switch value> 
<#case refValue1> 
....
<#break> 
<#default> 
.... 
</#switch>
```

### 八、集合遍历
###### 1. 集合遍历
```ftl
// 遍历集合:
<#list empList! as emp> 
    ${emp.name!}
</#list>

// 可以这样遍历集合:
<#list 0..(empList!?size-1) as i>
    ${empList[i].name!}
</#list>
```

###### 2. 集合size
```ftl
empList?size
```

###### 3. 跳出循环
```ftl
<#list empList! as emp>
    <#if emp_index = 0><#break></#if>
</#list>

```

###### 4. 集合是否存在下一个对象
```ftl
<#list empList! as emp> 
    <#if emp_has_next>
    ...
    </#if>
</#list>

```

###### 5. 创建集合
```ftl
<#list ["星期一", "星期二", "星期三", "星期四", "星期五", "星期六", "星期天"] as x>

<#list 0..100 as i> 　　// 等效于java for(int i=0; i <= 100; i++)
　　${i}
</#list>

```

###### 6.判断集合是否存在某个对象
```ftl
<#assign x = ["red", 16, "blue", "cyan"]> 
${x?seq_contains("blue")?string("yes", "no")}    // yes
```

###### 7. 集合中元素所在的下标
```ftl
<#assign x = ["red", 16, "blue", "cyan", "blue"]> 
${x?seq_index_of("blue")}  // 2
```
###### 8.排序
```ftl
// sort_by：排序（升序）
<#list movies?sort_by("showtime") as movie></#list>

// sort_by：排序（降序）
<#list movies?sort_by("showtime")?reverse as movie></#list>

```

### 九、Map
###### 1.创建Map
```ftl
<#assign scores = {"语文":86,"数学":78}>
```
###### 2.Map取值
```ftl
emp.name       // 全部使用点语法
emp["name"]    // 使用方括号
```

### 十、include
```ftl
// include指令的作用类似于JSP的包含指令:
<#include "/test.ftl" encoding="UTF-8" parse=true>

// 在上面的语法格式中,两个参数的解释如下:
encoding="GBK"  // 编码格式
parse=true 　　 // 是否作为ftl语法解析,默认是true，false就是以文本方式引入
注意:在ftl文件里布尔值都是直接赋值的如parse=true,而不是parse="true"
```
