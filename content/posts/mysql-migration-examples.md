---
title: MySQL常用的DDL操作
description: MySQL常用DDL操作示例，包括创建表、添加字段、重命名字段等操作
date: 2020-02-15T13:36:04+08:00
tags:
- MySQL
draft: false
---

在web应用中，由于需求的变更或者当初设计的不合理，数据库设计难免需要同步变更。比如，在[SpringBoot](https://spring.io/projects/spring-boot)中，一般用[Flyway](https://flywaydb.org/)或者[Liquibase](https://www.liquibase.org/)做数据库的迁移和版本控制，在[Play](https://www.playframework.com/)中使用[Evolution](https://www.playframework.com/documentation/2.8.x/Evolutions)做数据库迁移。

这里将常用的MySQL的DDL操作总结录下，主要用于备查。

## 1. create table

```sql
USE `freeimmi`;

CREATE TABLE IF NOT EXISTS `user`
(
    `id`         BIGINT       NOT NULL AUTO_INCREMENT,
    `name`       VARCHAR(32)  NOT NULL,
    `email`      VARCHAR(32)  NOT NULL,
    `password`   VARCHAR(128) NOT NULL,
    `status`     VARCHAR(16)  NOT NULL,
    `created_at` TIMESTAMP    NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (`id`),
    UNIQUE KEY `idx_email` (`email`),
    KEY `idx_status` (`status`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4
  COLLATE = utf8mb4_0900_ai_ci;
```

## 2. add column

```sql
USE `freeimmi`;

ALTER TABLE `topic`
    ADD COLUMN `logo_url` VARCHAR(64) NULL AFTER `name`,
    ADD COLUMN `description` VARCHAR(64) DEFAULT '' AFTER `logo_url`;
```

## 3. rename column

```sql
USE `freeimmi`;

ALTER TABLE `post`
    CHANGE COLUMN `submit_at` `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CHANGE COLUMN `submit_by` `updated_by` bigint NOT NULL;
```

## 4. drop table

```sql
USE `freeimmi`;

DROP TABLE topic_group;
```

## 5. drop column

```sql
USE `freeimmi`;

ALTER TABLE `topic`
    DROP COLUMN `topic_group_id`;
```

## 6. truncate table

```sql
USE `freeimmi`;

TRUNCATE TABLE `topic`;
```

## 7. rename table

```sql
USE `freeimmi`;

RENAME TABLE `post_comment` TO `comment`;
```

## 8. modify table

```sql
ALTER TABLE `post`
    MODIFY COLUMN `subject` VARCHAR(128) DEFAULT '';
```