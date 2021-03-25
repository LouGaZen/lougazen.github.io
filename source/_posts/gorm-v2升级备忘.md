---
title: gorm.v2升级备忘
date: 2021-03-25 11:01:33
tags:
---

## 序

[GORM v2](https://gorm.io/zh_CN/)发布半年有余，因为有太多破坏性变更，目前现有的项目都不敢贸贸然进行升级。恰逢最近新启动一个小项目，遂打算是用新版的gorm，为以后的升级提供一些参考。

## 变更

### 外键

v1版在处理关联对象时默认不会创建外键，创建外键需要显式在tag里声明。

升级后的v2默认会给关联的对象创建外键。如果需要跟v1保持一致，需要手动配置

```golang
db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{
	DisableForeignKeyConstraintWhenMigrating: true,
})
```

### datetime精度

v1版datetime精度默认是0（秒），v2版默认是3（毫秒），配置方式：

```golnag
db, err = gorm.Open(mysql.New(mysql.Config{
	DSN: dsn,
	DisableDatetimePrecision: true,
}), nil)
```

因为我使用的时间戳是是13位，所以这里没跟v1保持一致

### sql连接池

v1版的sql连接池可以直接通过`db.DB()`方法直接获取，v2则同时会返回一个err，需要单独处理

### log输出

v1版可以通过`db.LogMode(true)`在日志里输出每一条sql语句，v2里没有了这个方法，取而代之的是根据log level进行输出

```golang
db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{
	Logger: logger.Default.LogMode(logger.Info),
})
```

或

```golang
db, err := gorm.Open(mysql.Open(dsn), nil)
db = db.Debug()
```

使用下来发现日志详细程度会比v1更高

### RecordNotFound

v1版自带`gorm.IsErrRecordNotFound(err)`方法判断，升级v2后不再提供这个方法，需要改成`errors.Is(err, gorm.ErrRecordNotFound)`

## 鸣谢

> [GORM 2.0 发布说明](https://gorm.io/zh_CN/docs/v2_release_note.html)
