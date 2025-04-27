# validate-utils介绍

## 1. 概述
validate-utils是一个基于Hibernate Validator的轻量级校验框架，旨在简化和增强Java应用程序中的数据校验工作。该工具包提供了一系列常见的校验场景，帮助开发人员快速实现数据验证，提高代码的可维护性和可靠性。

## 2. 功能特性

### 2.1 集合数据量校验 `@CollectionSize`
支持对集合类型（如`List`、`Set`等）进行元素数量的校验，确保集合的大小符合业务需求。

#### 属性
| 属性名   | 描述         | 类型   | 默认值   |
|----------|--------------|--------|----------|
| min      | 最小值       | int    | 0        |
| max      | 最大值       | int    | 0        |
| message  | 错误信息     | string | 集合校验失败 |
| groups   | 分组         | Class<?>[] | {}      |

#### 示例
业务场景：校验集合的最小和最大元素数量。
```
@CollectionSize(min = 1, max = 10, message = "集合大小必须在1到10之间")
private List<String> items;
```

### 2.2 多字段校验 `@ConditionFieldCheck`
可以对多个字段进行组合校验，确保字段之间的逻辑关系符合要求。

#### 属性
| 属性名          | 描述                   | 类型              | 默认值           |
|-----------------|------------------------|-------------------|------------------|
| conditionField  | 判断条件               | ConditionField    |                  |
| checkField      | 待校验的数据           | ConditionField    |                  |
| message         | 错误信息               | string            | 不符合校验规则   |
| groups          | 分组                   | Class<?>[]        | {}               |

#### 字段校验 `@ConditionField`
| 属性名       | 描述         | 类型       | 默认值       |
|--------------|--------------|------------|--------------|
| fieldName    | 字段名       | String     | ""           |
| fieldValue   | 字段值       | String[]   | {}           |
| condition    | 条件         | ConditionEnum | ConditionEnum.EQ |
| message      | 错误信息     | String     | 不符合校验规则 |
| groups       | 分组         | Class<?>[] | {}           |

#### 条件 `ConditionEnum`
| 条件名称                | 描述                   |
|-------------------------|------------------------|
| EQ                      | 相等判断               |
| NOT_NULL                | 非空判断，不包含空格   |
| NOT_BLANK               | 非空判断，包含空格     |
| COLLECTION_NOT_EMPTY    | 集合不能为空           |
| IN                      | 集合中存在             |
| NOT_IN                  | 集合中不存在           |
| STRING_CONTAINS         | 字符串包含             |
| COLLECTION_MUST_EMPTY   | 集合必须为空           |
| MUST_BLANK              | 必须为空，不可包含空格 |
| STRING_NOT_CONTAINS     | 字符串不包含           |
| MUST_NULL               | 必须为空，可以不传或空字符串 |

#### 示例
业务场景：校验两个字段是否满足某种关系，例如在用户注册时，选择身份证类型时身份证号必填，选择护照类型时护照号必填。
```
@Data
@ConditionFieldCheck.List({
        @ConditionFieldCheck(conditionField = @ConditionField(fieldName = "busType", condition = ConditionEnum.EQ, fieldValue = "idCard"), checkField = @ConditionField(fieldName = "idCard", condition = ConditionEnum.NOT_BLANK), message = "注册类型是身份证,身份证号未填写"),
        @ConditionFieldCheck(conditionField = @ConditionField(fieldName = "busType", condition = ConditionEnum.EQ, fieldValue = "passport"), checkField = @ConditionField(fieldName = "passportNo", condition = ConditionEnum.NOT_BLANK), message = "注册类型是护照,护照号未填写")
})
public class RegisterReq {
    //业务类型
    @NotBlank
    private String busType;
    //身份证号
    private String idCard;
    //护照号
    private String passportNo;
}
```

### 2.3 多字段有一个不为空/为空校验 `@MultiFieldCheck`
支持对多个字段进行校验，确保至少有一个字段不为空，或者至少有一个字段为空。

#### 属性
| 属性名             | 描述                     | 类型       | 默认值   |
|--------------------|--------------------------|------------|----------|
| onlyExistOneField  | 只要有一个字段存在即可   | String[]   | {}       |
| onlyEmptyOneField  | 只要有一个字段不存在即可 | String[]   | {}       |
| message            | 错误信息                 | string     | 集合校验失败 |
| groups             | 分组                     | Class<?>[] | {}       |

#### 示例
业务场景：在用户注册时，邮箱和手机号至少填写一个。
```
@Data
@MultiFieldCheck.List({ 
@MultiFieldCheck(onlyExistOneField = { "email","phone" }, message = "请输入邮箱或手机号")
})
public class RegisterReq {
    //邮箱
    private String email;
    //手机号
    private String phone;
}
```

### 2.4 数字最大最小值校验 `@NumberLimit`
提供对数字类型字段的最大值和最小值的校验，确保数值在合理范围内。

#### 属性
| 属性名   | 描述         | 类型   | 默认值   |
|----------|--------------|--------|----------|
| min      | 最小值       | double | 0.0      |
| max      | 最大值       | double | 0.0      |
| message  | 错误信息     | string | 集合校验失败 |
| groups   | 分组         | Class<?>[] | {}      |

#### 示例
业务场景：校验年龄字段的值必须在10到30之间。
```
@NumberLimit(min=15,max=30,message = "年龄不能小于10岁且不能超过30岁")
private Integer age;
```

### 2.5 数据格式校验 `@Regex`
支持对字符串等数据类型进行格式校验，如邮箱、手机号、身份证号等。

#### 属性
| 属性名       | 描述         | 类型       | 默认值               |
|--------------|--------------|------------|----------------------|
| value        | 校验的格式   | RegexEnum  | RegexEnum.DEFAULTS  |
| rightValue   | 正确字段值   | string[]   | {}                   |
| regexp       | 正则表达式   | string     | ""                   |
| message      | 错误信息     | string     | 集合校验失败         |
| groups       | 分组         | Class<?>[] | {}                   |

#### RegexEnum
| 枚举名称                | 描述                   |
|-------------------------|------------------------|
| DEFAULTS                | 默认格式配置，返回校验通过 |
| NUMBER                  | 只能是数字             |
| ALPHABET                | 只能是字母             |
| ALPHABET_BIG            | 只能是大写字母         |
| ALPHABET_SMALL          | 只能是小写字母         |
| NUMBER_ALPHABET         | 只能是数字和字母       |
| NUMBER_ALPHABET_BIG     | 只能是数字和大写字母   |
| NUMBER_ALPHABET_SMALL   | 只能是数字和小写字母   |
| CHINESE                 | 只能是中文             |
| IDENTITY_CARD           | 只能是身份证           |
| EMAIL                   | 只能是邮箱             |
| CN_PHONE                | 只能是中国手机号       |
| EN_PHONE                | 只能是国外手机号       |
| RIGHT_VALUE             | 只能是选定的值         |

#### 示例
业务场景：校验输入的用户名只能是字母和数字。

```
@Regex(value = RegexEnum.NUMBER_ALPHABET, message = "用户名称只允许输入字母和数字")
private String userName;

 //业务类型
@NotBlank
@Regex(value = RegexEnum.RIGHT_VALUE,rightValue={"idCard","passport"}, message = "请选择身份证或护照")
private String busType;
```
## 3. 引入工具包
```
<dependency>
    <groupId>io.github.fengyantao</groupId>
    <artifactId>validate-utils</artifactId>
    <version>1.0.0</version>
</dependency>
```
---

## 结论
validate-utils通过提供灵活和强大的校验功能，帮助开发人员轻松实现数据验证，减少了重复代码，提高了应用程序的健壮性。欢迎大家在项目中使用并提出建议！
