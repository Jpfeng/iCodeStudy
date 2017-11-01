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

可以使用的配置有：

- `schemaVersion`

    当前数据库模式版本。 `OpenHelpers` 类在迁移不同版本数据库时会用到。如果数据库的模式发生变化，就需要增加版本号。默认值是 `1` 。

- `daoPackage`

    生成的 `DAO` ， `DaoMaster` 和 `DaoSession` 的包名。默认值是源实体的包名。

- `targetGenDir`

    生成的源代码的存储位置。默认是 `build` 文件夹内的生成代码文件夹 `build/generated/source/greendao` 。

- `generateTests`

    设置为 `true` 会自动生成单元测试。

- `targetGenDirTests`

    生成的单元测试源代码的位置。默认为 `src/androidTest/java` 。

## 创建实体

使用注解 `@Entity` 来定义实体。该注解将 Java 类定义为数据库实体。 GreenDAO 会根据注解生成必须的代码。**注意，只支持 Java 类。如果工程使用 Kotlin 语言，实体类也必须用 Java 编写。**

```java
@Entity
public class Note {
    ...
}
```

`@Entity` 注解还可以用来配置一些详情。

```java
@Entity(
        schema = "myschema",
        active = true,
        nameInDb = "AWESOME_USERS",
        indexes = {
                @Index(value = "name DESC", unique = true)
        },
        createInDb = false,
        generateConstructors = true,
        generateGettersSetters = true
)
public class Note {
    ...
}
```

- `schema`

    如果数据库有多种模式，该属性指定实体的所属模式。使用任意字符串定义模式名。**注意，目前 Gradle 插件不支持多种模式。如果只有一种模式，则不要定义 `schema` 。**

- `active`

    将实体标记为 `active` 。标记为 `active` 的实体会生成 `update()` ，`delete()` 和 `refresh()` 方法。默认值为 `false` 。

- `nameInDb`

    指定数据库中表的名称。默认值基于实体类名。

- `indexes`

    定义索引，可以跨越多个列。

- `createInDb`

    标记 DAO 是否在数据库创建表。默认值为 `true` 。如果有多个实体映射到一个表，或者在 GreenDAO 之外完成了表的创建，则需要设置为 `false` 。

- `generateConstructors`

    标记是否生成具有所有的属性构造函数。注意，总是需要一个无参数的构造函数。默认值为 `true` 。

- `generateGettersSetters`

    标记是否为每个属性自动生成 `get()` 和 `set()` 方法。默认值为 `true` 。

### 基本属性

```java
@Entity
public class Note {
    @Id(autoincrement = true)
    private Long id;

    @Property(nameInDb = "USERNAME")
    private String name;

    @NotNull
    private int repos;

    @Transient
    private int tempUsageCount;

    ...
}
```

- `@Id`

    使用 `long` 类型的属性作为实体 ID 。在数据库层面，这个属性就是主键。参数 `autoincrement` 指定属性是自增长的。

- `@Property`

    定义属性映射到非默认的列名。若不定义， GreenDAO 将用 SQL 命名方法，大写字母 + 下划线，作为列名。**注意：当前只能使用内联常量指定列名。**

- `@NotNull`

    将属性映射的列在数据库中指定为 `NOT NULL` 。通常使用 `@NotNull` 标记原始类型 (`short` , `byte` , `int` , `long`) 也有意义，因为其包装类型 (`Short` , `Byte` , `Integer` , `Long`) 有可空值。

- `@Transient`

    标记需要排除在持久化以外的属性。将该注解用于临时状态等。另外，也可以使用 Java 的 `transient` 关键字。

### 主键限制

目前，实体必须有一个 `long` 或 `Long` 类型的属性作为主键。这是 Android 和 SQLite 建议的标准做法。你可以将自定义的主键属性定义为额外的属性，并为其添加独特的索引。

```java
@Id
private Long id;

@Index(unique = true)
private String key;
```

### 属性索引

在属性上使用 `@Index` 注解来创建数据库对应列的索引。可以使用以下参数：

- `name`

    可以指定索引的名字，以代替默认命名。

- `unique`

    为索引添加 `UNIQUE` 约束，强制所有值都是独特的。

在属性上使用 `@Unique` 注解同样可以为对应列的索引添加 `UNIQUE` 约束。同时， SQLite 也会为其隐式创建索引。

```java
@Entity
public class Note {
    @Id
    private Long id;

    @Unique
    private String name;
}
```

若添加跨多个属性的索引，请使用 `@Entity` 注解的 `indexes` 配置。

### 关系

- `@ToOne`
- `@ToMany`

*TODO*

## 触发代码生成

在进行完配置并至少创建一个实体类后，可以进行编译。执行 `Build` → `Make Project` 进行编译。编译好就会生成 DAO 相关的代码。在实体类中也会生成相关的方法。

### 处理生成代码

在 GreenDAO 3 中，实体类是开发者创建并编辑的。然而，在生成代码的过程中可能会和实体代码产生冲突。 GreedDAO 会在生成的方法和变量上添加 `@Generated` 注解，以提示开发者并防止代码丢失。**在大多数情况下，不应该编辑带有 `@Generated` 注解的代码。**

遇到生成代码被修改的情况， GreenDAO 不会覆盖现有代码并抛出编译失败：

```shell
Error:Execution failed for task ':app:greendao'.
> Constructor (see ExampleEntity:21) has been changed after generation.
Please either mark it with @Keep annotation instead of @Generated to keep it untouched,
or use @Generated (without hash) to allow to replace it.
```

如错误信息所述，有两种方法解决：

- 将带有 `@Generated` 注解的代码恢复。或者，也可以将更改的构造方法或方法彻底删除。在下次编译时这些代码会再次生成。

- 将 `@Generated` 注解替换为 `@Keep` 注解。 GreenDAO 不会处理该注解下的代码。谨记你所做的更改可能会破坏实体与 GreenDAO 其他部分间的联系。另外， GreenDAO 后续版本可能会更改生成方法中的代码。故请万分小心！强烈建议在修改处进行单元测试以防错误发生。

## 操作数据库

在操作数据库的示例中，我们使用 `Note` 实体作为例子。经过编译， GreenDAO 会自动生成 `NoteDao` 。

### 初始化

通过 `DaoMaster` 的内部类 `DevOpenHelper` 可以得到一个 `SQLiteOpenHelper` 对象。通过该对象可以获得可读写的数据库对象 `SQLiteDatabase` 。调用 `DaoMaster(SQLiteDatabase)` 构造方法创建 `DaoMaster` 对象。调用 `DaoMaster` 的 `newSession()` 方法获得 `DaoSession` 对象。这些操作在程序中只需要进行一次，一般在 `Application` 中进行。

```java
DaoMaster.DevOpenHelper openHelper = new DaoMaster.DevOpenHelper(context, "noteDb", null);
SQLiteDatabase database = openHelper.getWritableDatabase();
DaoMaster daoMaster = new DaoMaster(database);
DaoSession daoSession = daoMaster.newSession();
```

在 `Activity` 或 `Fragment` 中，通过 `DaoSession` 对象获取 DAO ，并通过 DAO 进行数据库操作。

```java
NoteDao noteDao = daoSession.getNoteDao();
```

### 增

新建实体对象。实体对象的 `Id` 传入 `null` ，在插入过程中会自动赋值并实现自增长。然后调用 `insert()` 方法新增一条数据。该方法返回插入数据的行 id 。

```java
Note note = new Note(...);
noteDao.insert(note);
```

新增多条数据可以使用 `insertInTx(Iterable)` 方法。该方法会开启一个事务以添加所有数据。

```java
ArrayList<Note> entities = new ArrayList<>();
entities.add(note);
...
noteDao.insertInTx(entities);
```

### 删

删除数据首先得到要删除对象的主键 `id` ，调用 `deleteByKey()` 通过 `id` 进行删除。

```java
noteDao.deleteByKey(primaryKey);
```

也可以直接通过实体对象，调用 `delete()` 方法进行删除。

```java
noteDao.delete(note);
```

删除多条数据可以使用 `deleteByKeyInTx(Iterable)` 或 `deleteInTx(Iterable)` 。这些方法会在一个事务中删除多条数据。

```java
ArrayList<Long> ids = new ArrayList<>();
ids.add(id);
...
noteDao.deleteByKeyInTx(ids);

ArrayList<Note> notes = new ArrayList<>();
notes.add(note);
...
noteDao.deleteInTx(notes);
```

如果要删除所有数据，可以使用 `deleteAll()` 方法。

```java
noteDao.deleteAll();
```

### 改

修改数据可以调用 `update()` 方法。

```java
noteDao.update(note);
```

修改多条数据可以使用 `updateInTx(Iterable)` 方法。该方法会在一个事务中修改多条数据。

```java
ArrayList<Note> notes = new ArrayList<>();
notes.add(note);
...
noteDao.updateInTx();
```

### 查

## 数据库升级
