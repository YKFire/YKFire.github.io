---
title: Java DTO类与VO类的区别
date: 2022-11-27 10:53:00 +0800
categories: [技术科普]
tags: [学习]
pin: true
author: YKFire

toc: true
comments: true

math: false
mermaid: true

typora-root-url: ../../YKFire.github.io

---

# Java DTO类与VO类的区别 

## 一、联系

​	DTO类与VO类都是在java后端开发过程中用来**接收数据**和**返回数据**的类

## 二、区别

### 1、VO类

> VO代表展示层需要显示的数据

​	VO即**View Object表示层对象**，**主要体现在视图的对象，对于一个WEB页面将整个页面的属性封装成一个对象**。然后用一个VO对象将在控制层得到的数据传输到视图层。

### 2、DTO类

> DTO代表服务层需要接收的数据和返回的数据

​	DTO即**Data Transfer Object数据传输对象**，**用于后端接收前端传递过来的参数，并进行业务逻辑操作**，有些接收参数要用对象来接收，但是发现哪个Entity都不合适，就有了DTO；是经过处理后的PO，可能增加或减少了PO的属性。

### 3、其他相关的概念

- POJO：Plain Ordinary Java Object无规则简单Java对象，可以转化为VO、DTO、PO
- PO：Persistent Object持久化对象，它跟数据表形成一一对应的映射关系
- Entity：实体，和PO的功能类似，和数据表一一对应，一个实体一张表

## 三、应用场景

完整的订单PO类 与一张数据表形成对应关系

```java
/**
 * 订单表
 * @TableName orders
 */
@TableName(value ="orders")
@Data
public class Orders implements Serializable {
    /**
     * 订单id
     */
    @TableId(type = IdType.AUTO)
    private Long id;

    /**
     * 订单发起者id
     */
    private Long launchId;

    /**
     * 收货人电话
     */
    private String conPhone;

    /**
     * 收货人名称
     */
    private String consignee;

    /**
     * 订单接受者id
     */
    private Long acceptId;

    /**
     * 快递员电话
     */
    private String couPhone;

    /**
     * 快递员姓名
     */
    private String courier;

    /**
     * 产品id
     */
    private Long productId;

    /**
     * 订单价格
     */
    private Integer price;

    /**
     * 备注信息
     */
    private String remarks;

    /**
     * 地址信息
     */
    private String address;

    /**
     * 订单状态(0-已取消, 1-待接单,2-待确认,3-已接单, 4-订单完成)
     */
    private Integer orderStatus;

    /**
     * 创建时间
     */
    private Date createTime;

    /**
     * 更新时间
     */
    private Date updateTime;

    /**
     * 是否删除(0-未删, 1-已删)
     */
    @TableLogic
    private Integer isDeleted;

    @TableField(exist = false)
    private static final long serialVersionUID = 1L;
}
```

### 1、VO类的应用场景

​	假设我们在一个订单管理系统中，当前端发送**获取订单具体信息的请求**，我们可以将搜索到的数据记录装载到PO类中进行返回，但往往前端的数据只需要一部分的数据信息即可，而且如果直接将PO类进行返回还会泄露系统中对应表的数据库表字段，会造成安全性问题，因此我们可以创建VO类装载只需要的信息进行返回

```java
@Data
public class OrdersDetailsVO {

    /**
     * 收货人名称
     */
    private String consignee;

    /**
     * 收货人电话
     */
    private String conPhone;

    /**
     * 订单价格
     */
    private Integer price;

    /**
     * 地址信息
     */
    private String address;

    /**
     * 备注信息
     */
    private String remarks;

    /**
     * 订单状态(0-已取消, 1-待接单,2-待确认,3-已接单, 4-订单完成)
     */
    private Integer orderStatus;

    //------------------------------------------

    /**
     * 产品名称
     */
    private String name;

    /**
     * 产品图片
     */
    private String image;

}

```

控制层使用VO类向前端返回数据：

```java
	/**
     * 首页分页获取订单信息接口
     *
     * @param id
     * @param page
     * @param pageSize
     * @return R<PageVO>
     */
    @GetMapping("/homeList/{id}")
    public R<PageVO> homeList(@PathVariable Long id, int page, int pageSize) {
        PageVO pageVO = ordersService.getHomeList(id, page, pageSize);
        return R.success(pageVO);
    }
```

### 2、DTO类的应用场景

​	同样在一个订单管理系统中，当前端传递过**来需要添加的相应订单信息**，后端可以使用对应的对象来进行接受，但实际前端传递过来的信息仅仅只有一部分，如果直接使用PO类来接受，便会造成浪费，因而我们可以创建对应的DTO类进行接受

```java
@Data
public class AddOrderDto {
    /**
     * 序列化ID
     */
    private static final long serialVersionUID = 1L;

    /**
     * 收货人电话
     */
    private String conPhone;

    /**
     * 收货人名称
     */
    private String consignee;


    /**
     * 订单价格
     */
    private Integer price;

    /**
     * 备注信息
     */
    private String remarks;

    /**
     * 地址信息
     */
    private String address;

    /**
     * 产品名称
     */
    private String name;

    /**
     * 产品图片
     */
    private String image;
}
```

控制层使用DTO类进行接受参数：

```java
	/**
     * 发布订单接口
     *
     * @param addOrderDto
     * @return R<String>
     */
    @PostMapping("/add/{id}")
    private R<String> add(@PathVariable Long id, @RequestBody AddOrderDto addOrderDto) {
        return ordersService.addOrder(id, addOrderDto);
    }
```

> 被`@RequestBody` 修饰的请求参数必须为实体类，且实体类必须实现序列化接口，如果不序列化就会出错，一般的参数不需要添加`@RequestBody` 
