- https://www.youtube.com/watch?v=Q9UiuHvNTp4  
- https://habr.com/ru/articles/312248/  
- 
#### 1
``` java
@Getter
@Setter
@ToString
public class NameBase {
    private String name;
}
```
``` java
@Getter
@Setter
@ToString(callSuper = true)
public class Book extends NameBase {
    private Author author;
    private int page;
}
```
``` java
@Getter
@Setter
@ToString(callSuper = true)
public class Author extends NameBase {
    private String surname;
    private int age;
}
```
``` java
public interface TestBuilder<T> {
    T build();
}
```
``` java
@AllArgsConstructor
@NoArgsConstructor(staticName = "aAuthor")
@With
public class AuthorTestBuilder implements TestBuilder<Author> {
    private String name = "name";
    private String surname = "surname";
    private int age = 30;

    @Override
    public Author build() {
        Author author = new Author();
        author.setName(name);
        author.setSurname(surname);
        author.setAge(age);
        return author;
    }
}
```
``` java
@AllArgsConstructor
@NoArgsConstructor(staticName = "aBook")
@With
public class BookTestBuilder implements TestBuilder<Book> {
    private String name = "book";
    private AuthorTestBuilder authorBuilder = AuthorTestBuilder.aAuthor();
    private int page = 0;

    @Override
    public Book build() {
        final Book book = new Book();
        book.setName(name);
        book.setAuthor(authorBuilder.build());
        book.setPage(page);
        return book;
    }
}
```
``` java
public class TestMain {
    public static void main(String[] args) {
        BookTestBuilder bookBuilder = BookTestBuilder.aBook();
        Book book = bookBuilder.build();
        System.out.println(book);               //Book(super=NameBase(name=book), author=Author(super=NameBase(name=name), surname=surname, age=30), page=0)
        bookBuilder = bookBuilder.withPage(5);
        Book bookCustomName = bookBuilder.withName("Name").build();
        System.out.println(bookCustomName);     //Book(super=NameBase(name=Name), author=Author(super=NameBase(name=name), surname=surname, age=30), page=5)
        Book bookCustomAuthor = bookBuilder.withAuthorBuilder(AuthorTestBuilder.aAuthor().withAge(22).withName("Jerry")).build();
        System.out.println(bookCustomAuthor);   //Book(super=NameBase(name=book), author=Author(super=NameBase(name=Jerry), surname=surname, age=22), page=5)
    }
}
```
#### 2
