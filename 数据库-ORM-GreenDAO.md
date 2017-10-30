# GreenDAO

> 版本： GreenDAO 3.2.2

[GitHub](https://github.com/greenrobot/greenDAO "GreenDAO-GitHub")
[website](http://greenrobot.org/greendao/ "GreenDAO-WebSite")

## 为工程添加 GreenDAO

在工程目录的 `build.gradle` 添加如下内容：

```groovy
buildscript {
    repositories {
        mavenCentral()
    }

    dependencies {
        classpath 'org.greenrobot:greendao-gradle-plugin:3.2.2'
    }
}
```

在 app 模块的 `build.gradle` 添加如下内容：

```groovy
apply plugin: 'org.greenrobot.greendao'

dependencies {
    compile 'org.greenrobot:greendao:3.2.2'
}
```

这些配置将 GreenDAO Gradle 插件连接到编译进程中。在编译工程时，它可以创建 `DaoMaster` ， `DaoSession` 和 `DAO` 相关的一些类。

## 配置版本

在 app 模块的 `build.gradle` 对数据库版本等信息进行配置。

```groovy
android {
...
}

greendao {
    schemaVersion 1
}
```

可以添加的配置有：

- `schemaVersion` ：当前数据库的版本号。 `OpenHelpers` 类在迁移不同版本数据库时会用到。如果数据库的结构发生变化，就需要增加版本号。默认值是 `1` 。
- `daoPackage` ：生成的 `DAO` ， `DaoMaster` 和 `DaoSession` 的包名。默认值是源实体的包名。
- `targetGenDir` ：生成的源代码的存储位置。默认是 `build` 文件夹内的生成代码文件夹 `build/generated/source/greendao` 。
- `generateTests` ：设置为 `true` 会自动生成单元测试。
- `targetGenDirTests` ：生成的单元测试源代码的位置。默认为 `src/androidTest/java` 。

## 创建实体
