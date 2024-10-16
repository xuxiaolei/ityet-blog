---
layout: post
title: Mapstruct自定义转换规则
categories: Java
description: Mapstruct自定义转换规则
index_img: /img/post_def.png
date: 2021-07-10 09:09:09
tags: [Mapstruct,Java]
---

## Mapstruct自定义转换规则

# 一、定义转换规则

定义的类上边增加**@Named注解标注转换名称**

**定义转换规则**

```java
import cn.hutool.core.util.StrUtil;
import com.alibaba.fastjson.JSON;
import org.mapstruct.Named;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.Objects;

/**
 * Mapping通用转换
 */
@Component
public class TypeConversionWorker {
    /**
     * 对象转json字符串
     *
     * @param obj
     * @return
     */
    @Named("toJsonString")
    public String toJsonString(Object obj) {
        if (Objects.isNull(obj)) {
            return null;
        }
        return JSON.toJSONString(obj);
    }
    /**
     * json字符串转对象
     *
     * @param jsonStr
     * @return
     */
    @Named("jsonStringToNameObject")
    public List<Name> jsonStringToObject(String jsonStr) {
        if (StrUtil.isEmpty(jsonStr)) {
            return null;
        }
        List<Name> names = JSON.parseArray(jsonStr, Name.class);
        return names;
    }
    /**
     * json字符串转对象
     *
     * @param jsonStr
     * @return
     */
    @Named("jsonStringToNameValueObject")
    public List<NameValue> jsonStringToNameValueObject(String jsonStr) {
        if (StrUtil.isEmpty(jsonStr)) {
            return null;
        }
        List<NameValue> names = JSON.parseArray(jsonStr, NameValue.class);
        return names;
    }
    /**
     * json字符串转对象
     *
     * @param jsonStr
     * @return
     */
    @Named("jsonStringToValueObject")
    public List<Value> jsonStringToValueObject(String jsonStr) {
        if (StrUtil.isEmpty(jsonStr)) {
            return null;
        }
        List<Value> names = JSON.parseArray(jsonStr, Value.class);
        return names;
    }
}
```

# 二、使用转换规则

使用@Mapper注解**uses**引入转换规则，eg:**uses = TypeConversionWorker.class**

@Mapping使用**qualifiedByName**标识转换规则，eg:**qualifiedByName = "toJsonString"。**

```java
import org.mapstruct.Mapper;
import org.mapstruct.Mapping;
import org.mapstruct.factory.Mappers;

import java.util.List;

@Mapper(uses = TypeConversionWorker.class)
public interface ShopConvert {

    ShopConvert INSTANCE = Mappers.getMapper(ShopConvert.class);
    @Mapping(target = "serviceField", source = "serviceField", qualifiedByName = "jsonStringToNameObject")
    @Mapping(target = "costField", source = "costField", qualifiedByName = "jsonStringToNameValueObject")
    @Mapping(target = "examinationField", source = "examinationField", qualifiedByName = "jsonStringToValueObject")
    List<ShopRespVO> convertList(List<ShopDO> list);

    List<ShopSimpleRespVO> convertList02(List<ShopDO> list);

    @Mapping(target = "serviceField", source = "serviceField", qualifiedByName = "jsonStringToNameObject")
    @Mapping(target = "costField", source = "costField", qualifiedByName = "jsonStringToNameValueObject")
    @Mapping(target = "examinationField", source = "examinationField", qualifiedByName = "jsonStringToValueObject")
    ShopRespVO convert(ShopDO bean);

    @Mapping(target = "serviceField", source = "serviceField", qualifiedByName = "toJsonString")
    @Mapping(target = "costField", source = "costField", qualifiedByName = "toJsonString")
    @Mapping(target = "examinationField", source = "examinationField", qualifiedByName = "toJsonString")
    ShopDO convert(ShopCreateReqVO bean);

    @Mapping(target = "serviceField", source = "serviceField", qualifiedByName = "toJsonString")
    @Mapping(target = "costField", source = "costField", qualifiedByName = "toJsonString")
    @Mapping(target = "examinationField", source = "examinationField", qualifiedByName = "toJsonString")
    ShopDO convert(ShopUpdateReqVO bean);
}
```

** 涉及到实体(非重点)**

```java
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;
import lombok.EqualsAndHashCode;

import java.util.Date;
import java.util.List;

@ApiModel("管理后台 - 商品信息 Response VO")
@Data
@EqualsAndHashCode(callSuper = true)
public class ShopRespVO extends ShopBaseVO {

    @ApiModelProperty(value = "商品编号", required = true, example = "1024")
    private Long id;


    @ApiModelProperty(value = "创建时间", required = true, example = "时间戳格式")
    private Date createTime;

    /*分类名称*/
    private String systemClassificationName;

    /*前端回填ID*/
    private List<Long> ids;

}
```

```java
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;
import org.hibernate.validator.constraints.Length;

import java.math.BigDecimal;
import java.util.List;

/**
 * 商品 Base VO，提供给添加、修改、详细的子 VO 使用
 * 如果子 VO 存在差异的字段，请不要添加到这里，影响 Swagger 文档生成
 */
@Data
public class ShopBaseVO {

    /**
     * 商品名称
     */
    @ApiModelProperty("商品名称")
    @Length(max= 255,message="编码长度不能超过255")
    private String name;
    /**
     * 商品分类
     */
    @ApiModelProperty("商品分类")
    private Long systemClassificationId;
    /**
     * 标准价格
     */
    @ApiModelProperty("标准价格")
    private BigDecimal price;
    /**
     * 最低价格
     */
    @ApiModelProperty("最低价格")
    private BigDecimal minPrice;
    /**
     * 商品状态（0正常 1停用）
     */
    @ApiModelProperty("商品状态（0正常 1停用）")
    private Byte status;
    /**
     * 服务字段
     */
    @ApiModelProperty("服务字段")
    private List<Name> serviceField;
    /**
     * 成本费用标准
     */
    @ApiModelProperty("成本费用标准")
    private List<NameValue> costField;
    /**
     * 补充报考字段
     */
    @ApiModelProperty("补充报考字段")
    private List<Value> examinationField;
    /**
     * 备注
     */
    @ApiModelProperty("备注")
    @Length(max= 255,message="备注长度不能超过255")
    private String remark;
}
```

```java
import com.baomidou.mybatisplus.annotation.TableName;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;
import lombok.EqualsAndHashCode;
import org.hibernate.validator.constraints.Length;

import java.math.BigDecimal;
import java.util.Date;

/**
 * 商品
 * @TableName system_shop
 */
@TableName("system_shop")
@Data
@EqualsAndHashCode(callSuper = true)
public class ShopDO extends TenantBaseDO {

    /**
     * 商品ID
     */
    @ApiModelProperty("商品ID")
    private Long id;
    /**
     * 商品名称
     */
    @ApiModelProperty("商品名称")
    @Length(max= 255,message="编码长度不能超过255")
    private String name;
    /**
     * 商品分类
     */
    @ApiModelProperty("商品分类")
    private Long systemClassificationId;
    /**
     * 标准价格
     */
    @ApiModelProperty("标准价格")
    private BigDecimal price;
    /**
     * 最低价格
     */
    @ApiModelProperty("最低价格")
    private BigDecimal minPrice;
    /**
     * 商品状态（0正常 1停用）
     */
    @ApiModelProperty("商品状态（0正常 1停用）")
    private Byte status;
    /**
     * 服务字段
     */
    @ApiModelProperty("服务字段")
    private String serviceField;
    /**
     * 成本费用标准
     */
    @ApiModelProperty("成本费用标准")
    private String costField;
    /**
     * 补充报考字段
     */
    @ApiModelProperty("补充报考字段")
    private String examinationField;
    /**
     * 创建者
     */
    @ApiModelProperty("创建者")
    @Length(max= 64,message="编码长度不能超过64")
    private String creator;
    /**
     * 创建时间
     */
    @ApiModelProperty("创建时间")
    private Date createTime;
    /**
     * 更新者
     */
    @ApiModelProperty("更新者")
    @Length(max= 64,message="编码长度不能超过64")
    private String updater;
    /**
     * 更新时间
     */
    @ApiModelProperty("更新时间")
    private Date updateTime;
    /**
     * 是否删除
     */
    @ApiModelProperty("是否删除")
    private Boolean deleted;
    /**
     * 租户编号
     */
    @ApiModelProperty("租户编号")
    private Long tenantId;
    /**
     * 备注
     */
    @ApiModelProperty("备注")
    @Length(max= 255,message="编码长度不能超过255")
    private String remark;


}
```

```java
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;
import lombok.EqualsAndHashCode;

import javax.validation.constraints.NotNull;

@ApiModel("管理后台 - 商品更新 Request VO")
@Data
@EqualsAndHashCode(callSuper = true)
public class ShopUpdateReqVO extends ShopBaseVO {

    @ApiModelProperty(value = "商品编号", required = true, example = "1024")
    @NotNull(message = "商品编号不能为空")
    private Long id;


}
```

```java
import io.swagger.annotations.ApiModel;
import lombok.Data;
import lombok.EqualsAndHashCode;
import lombok.ToString;

@ApiModel("管理后台 - 商品创建 Request VO")
@Data
@EqualsAndHashCode(callSuper = true)
@ToString(callSuper = true)
public class ShopCreateReqVO extends ShopBaseVO {
}
```

# 三、简单快速使用方式
```java
import org.apache.commons.lang3.time.DateUtils;
import org.mapstruct.Mapper;
import org.mapstruct.Mapping;
import org.mapstruct.Named;
import org.mapstruct.factory.Mappers;

import java.text.ParseException;
import java.util.Date;

/**
* ParamConverter
* 导出入参mapstruct转换
*
*/
  @Mapper(componentModel = "spring")
  public interface ParamConverter {

  ParamConverter INSTANCE = Mappers.getMapper(ParamConverter.class);

  /**
    * 入参转换
    *
    * @param param param
    * @return VipBuyListRequest
      */
      @Mapping(source = "startTime", target = "startTime", qualifiedByName = "customStringToDate")
      @Mapping(source = "endTime", target = "endTime", qualifiedByName = "customStringToDate")
      VipBuyListRequest convert(VipBuyListParam param);


    /**
     * 字符串转日期（兼容多个格式）
     *
     * @param dateStr 日期字符串
     * @return 日期
     */
    @Named("customStringToDate")
    default Date jsonStringToObject(String dateStr) {
        try {
            if (dateStr.length() > 10) {
                return DateUtils.parseDate(dateStr, "yyyy/MM/dd HH:mm:ss");
            } else {
                return DateUtils.parseDate(dateStr, "yyyy/MM/dd");
            }
        } catch (ParseException e) {
            throw new RuntimeException(e);
        }
    }
}
```