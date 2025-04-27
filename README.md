# 业务校验工具包介绍

## 1. 概述

业务校验工具包是一个基于 Hibernate Validator 的轻量级校验框架，旨在简化和增强 Java 应用程序中的数据校验工作。该工具包提供了一系列常见的校验场景，帮助开发人员快速实现数据验证，提高代码的可维护性和可靠性。

---

## 2. 功能特性

### 2.1 集合数据量校验 `@CollectionSize`

支持对集合类型（如 List、Set 等）进行元素数量的校验，确保集合的大小符合业务需求。

#### 2.1.1 属性

| **属性名** | **描述**     | **类型**     | **默认值**      |
|------------|--------------|--------------|-----------------|
| min        | 最小值       | int          | 0               |
| max        | 最大值       | int          | 0               |
| message    | 错误信息     | string       | 集合校验失败    |
| groups     | 分组         | Class<?>[]   | {}              |

#### 2.1.2 示例

**业务场景**：校验集合的最小和最大元素数量。

```java
@CollectionSize(min = 1, max = 10, message = "集合大小必须在1到10之间")
private List<String> items;
```
### 2.2 多字段校验 `@ConditionFieldCheck`  
可以对多个字段进行组合校验，确保字段之间的逻辑关系符合要求。

#### 2.2.1 属性定义  
| 属性名             | 描述               | 类型              | 默认值           |
|--------------------|--------------------|-------------------|------------------|
| `conditionField`   | 判断条件字段       | `ConditionField`  | 必填             |
| `checkField`       | 待校验字段         | `ConditionField`  | 必填             |
| `message`          | 错误信息           | `string`          | `不符合校验规则` |
| `groups`           | 分组               | `Class<?>[]`      | `[]`             |

**`ConditionField` 属性定义**  
| 属性名         | 描述               | 类型              | 默认值               |
|----------------|--------------------|-------------------|----------------------|
| `fieldName`    | 字段名             | `String`          | `""`（空字符串）     |
| `fieldValue`   | 字段值             | `String[]`        | `[]`（空数组）       |
| `condition`    | 条件类型           | `ConditionEnum`   | `ConditionEnum.EQ`   |
| `message`      | 错误信息           | `String`          | `不符合校验规则`     |
| `groups`       | 分组               | `Class<?>[]`      | `[]`                 |

#### 2.2.2 条件枚举 `ConditionEnum`  
| 枚举值                   | 描述                                                                 |
|--------------------------|----------------------------------------------------------------------|
| `EQ`                     | 字段值必须等于指定值                                                 |
| `NOT_NULL`               | 字段值不能为 `null`（不检查空格）                                    |
| `NOT_BLANK`              | 字段值不能为空或仅包含空格                                           |
| `COLLECTION_NOT_EMPTY`   | 集合类型字段不能为空                                                 |
| `IN`                     | 字段值必须在指定集合中                                               |
| `NOT_IN`                 | 字段值不能在指定集合中                                               |
| `STRING_CONTAINS`        | 字符串必须包含指定子串                                               |
| `COLLECTION_MUST_EMPTY`  | 集合类型字段必须为空                                                 |
| `MUST_BLANK`             | 字段值必须为空或仅包含空格                                           |
| `STRING_NOT_CONTAINS`    | 字符串不能包含指定子串                                               |
| `MUST_NULL`              | 字段值必须为 `null` 或空字符串                                       |

#### 2.2.3 示例  
**业务场景**：用户注册时，选择身份证类型则身份证号必填，选择护照类型则护照号必填。

```java
@Data
@ConditionFieldCheck.List({
    @ConditionFieldCheck(
        conditionField = @ConditionField(fieldName = "busType", condition = ConditionEnum.EQ, fieldValue = "idCard"),
        checkField = @ConditionField(fieldName = "idCard", condition = ConditionEnum.NOT_BLANK),
        message = "注册类型是身份证，身份证号未填写"
    ),
    @ConditionFieldCheck(
        conditionField = @ConditionField(fieldName = "busType", condition = ConditionEnum.EQ, fieldValue = "passport"),
        checkField = @ConditionField(fieldName = "passportNo", condition = ConditionEnum.NOT_BLANK),
        message = "注册类型是护照，护照号未填写"
    )
})
public class RegisterReq {
    @NotBlank
    private String busType;     // 业务类型（idCard/passport）
    private String idCard;      // 身份证号
    private String passportNo;  // 护照号
}
```
