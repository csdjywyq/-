## @Query注解的用法(Spring Data JPA)

### 1. 一个使用@Query注解的简单例子

```java
@Query(value = "select name,author,price from Book b where b.price>?1 and b.price<?2")
List<Book> findByPriceRange(long price1, long price2);
```

### 2. Like表达式

```java
@Query(value = "select name,author,price from Book b where b.name like %:name%")
List<Book> findByNameMatch(@Param("name") String name);
```

### 3. 使用Native SQL Query

所谓本地查询，就是使用原生的sql语句（根据数据库的不同，在sql的语法或结构方面可能有所区别）进行查询数据库的操作。

```java
@Query(value = "select * from book b where b.name=?1", nativeQuery = true)
List<Book> findByName(String name);
```

### 4. 使用@Param注解注入参数

```java
@Query(value = "select name,author,price from Book b where b.name = :name AND b.author=:author AND b.price=:price")
List<Book> findByNamedParam(@Param("name") String name, @Param("author") String author,
        @Param("price") long price);
```

### 5. SPEL表达式(使用时请参考最后的补充说明)

   '#{#entityName}'值为'Book'对象对应的数据表名称(book)。

```java
public interface BookQueryRepositoryExample extends Repository<Book, Long>{
       @Query(value = "select * from #{#entityName} b where b.name=?1", nativeQuery = true)
       List<Book> findByName(String name);

}
```

### 6. 一个较完整的例子

```java
public interface BookQueryRepositoryExample extends Repository<Book, Long> {
    @Query(value = "select * from Book b where b.name=?1", nativeQuery = true) 
    List<Book> findByName(String name);// 此方法sql将会报错(java.lang.IllegalArgumentException)，看出原因了吗,若没看出来，请看下一个例子

    @Query(value = "select name,author,price from Book b where b.price>?1 and b.price<?2")
    List<Book> findByPriceRange(long price1, long price2);

    @Query(value = "select name,author,price from Book b where b.name like %:name%")
    List<Book> findByNameMatch(@Param("name") String name);

    @Query(value = "select name,author,price from Book b where b.name = :name AND b.author=:author AND b.price=:price")
    List<Book> findByNamedParam(@Param("name") String name, @Param("author") String author,
            @Param("price") long price);

}
```

### 7. 解释例6中错误的原因：

​     因为指定了nativeQuery = true，即使用原生的sql语句查询。使用java对象'Book'作为表名来查自然是不对的。只需将Book替换为表名book。

```java
@Query(value = "select * from book b where b.name=?1", nativeQuery = true)
List<Book> findByName(String name);
```