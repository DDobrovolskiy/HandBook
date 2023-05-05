https://www.youtube.com/watch?v=Q9UiuHvNTp4
#### 1
``` java
@Getter
@Setter
@ToString
public class Book {
    private String name;
    private String author;
    private int page;
}
```
``` java
public interface TestBuilder<T> {
    T build();
}
```
``` java
@AllArgsConstructor
@NoArgsConstructor(staticName = "aBook")
@With
public class BookTestBuilder implements TestBuilder<Book> {
    private String name = "book";
    private String author = "author";
    private int page = 0;

    @Override
    public Book build() {
        final Book book = new Book();
        book.setName(name);
        book.setAuthor(author);
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
        System.out.println(book);               //Book(name=book, author=author, page=0)
        bookBuilder = bookBuilder.withPage(5);
        Book bookCustomName = bookBuilder.withName("Name").build();
        System.out.println(bookCustomName);     //Book(name=Name, author=author, page=5)
        Book bookCustomAuthor = bookBuilder.withAuthor("Jerry").build();
        System.out.println(bookCustomAuthor);   //Book(name=book, author=Jerry, page=5)
    }
}
```
#### 2
