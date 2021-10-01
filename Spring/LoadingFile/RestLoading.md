Пример раздачи файла
Под REST архитектуру пишутся контроллеры с аннотацией @RestController, но часто возникает задача раздачи файлов. В случае если нужно раздавать особые форматы, типа pdf, очень удобно использовать класс ResponseEntity.

Пример выгрузки pdf:
```java
@GetMapping("/example5")
public ResponseEntity<byte[]> example5() throws IOException {
        var content = Files.readAllBytes(Path.of("./book.pdf"));
        return ResponseEntity.ok()
                .contentType(MediaType.APPLICATION_PDF)
                .contentLength(content.length)
                .body(content);
}
```

Если надо раздавать файл с заголовком ответа "Content-type: application/octet-stream", то достаточно написать так:
```java
@GetMapping("/example6")
public byte[] example6() throws IOException {
        return Files.readAllBytes(Path.of("./pom.xml"));
}
```
Если выполнить запрос в PostMan, то он бинарные данные преобразует в текст:



Если выполнить запрос в браузере, то браузер предложит загрузить файл:

