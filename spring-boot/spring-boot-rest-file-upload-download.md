# Integrate Angular in a Spring Boot Project

## Initialize the project
* Go to the start.spring.io [page](https://start.spring.io/ "spring boot project generator")
* Select
    * **Build Tool**: Gradle - Groovy
    * **Language**: Java
    * **Spring Boot Version**: 3.1.0
    * **Packaging**: Jar
    * **Java Version**: 17
* Fill the metadata part as your own
* Select Dependecies
    * **Spring WEB**: Contains Spring MVC and embedded Tomcat container
* Click the **GENERATE** Button
* Copy the compressed file to a desired location
* Unzip it

## Create an abstract storage service
```java
    public interface StorageService {
    void init();
    String store(MultipartFile file);
    Path getRootPath();
    List<FileInfo> loadAll();
    String load(String fileName);
    Resource loadFileAsResource(String fileName) throws IOException;
}
```
## Create its implementation
```java
@Service
public class StorageServiceImpl implements StorageService{

    @Value("${storage.upload.path}")
    private String path;

    @Bean
    @Override
    public void init() {
        Path uploadPath = Paths.get(path);

        if(Files.notExists(uploadPath)) {
            try {
                Files.createDirectories(uploadPath);
            } catch (IOException e) {
                System.err.println(e.getMessage());
            }
        }
    }

    @Override
    public String store(MultipartFile file) {
        init();
        String fileName = file.getOriginalFilename();
        try (
                InputStream inputStream = file.getInputStream()
        ) {
            Path filePath = getRootPath().resolve(fileName);

            Files.copy(inputStream, filePath, StandardCopyOption.REPLACE_EXISTING);

            return filePath.toString();

        } catch (IOException e) {
            System.err.println(e.getMessage());
        }

        return "/";
    }

    @Override
    public Path getRootPath() {
        return Paths.get(path);
    }

    @Override
    public List<FileInfo> loadAll() {
        try {
            return Files.list(getRootPath())
                    .map(file -> new FileInfo(file.toString()))
                    .collect(Collectors.toList());
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    @Override
    public String load(String fileName) {
        Path filePath = getRootPath().resolve(fileName);
        if(Files.exists(filePath)) {
            return filePath.toString();
        }
        return "/";
    }

    @Override
    public Resource loadFileAsResource(String fileName) throws IOException {
        Path filePath = getRootPath().resolve(fileName);
        if(Files.exists(filePath)) {
            return new UrlResource(filePath.toUri());
        } else {
            throw new IOException("File not found");
        }
    }

}
```
* Here we get the root path from the application.properties file.
    ```java
    @Value("${storage.upload.path}")
    private String path;
    ```
* In init() method, create the root directory if not exists.
* Annotate this class with @Service, so we can autowire it.
* The loadFileAsResource will be used for downloading the file.

## The application.properties file
```properties
spring.servlet.multipart.max-file-size=5MB
storage.upload.path=E:/vms/test
```
* We can't upload more than 5MB size file/files.
* Here, the path is a standard windows path.

## The controller class
```java
@RestController
public class FileUploadController {

    @Autowired
    private StorageService storageService;

    @PostMapping("/upload")
    public ResponseEntity<FileUploadResponse> uploadFiles(@RequestParam("file1") MultipartFile[] multipartFile) {

        List<String> files = Stream.of(multipartFile)
                .map(file -> storageService.store(file))
                .collect(Collectors.toList());

        FileUploadResponse fileUploadResponse = new FileUploadResponse(files);

        return new ResponseEntity<>(fileUploadResponse, HttpStatus.OK);
    }

    @GetMapping("/files")
    public ResponseEntity<List<FileInfo>> getAllFiles() {
        return new ResponseEntity<>(storageService.loadAll(), HttpStatus.OK);
    }

    @GetMapping("/file/{fileName}")
    public ResponseEntity<FileInfo> getFile(@PathVariable String fileName) {
        return new ResponseEntity<>(new FileInfo(storageService.load(fileName)), HttpStatus.OK);
    }

    @GetMapping("/download/{fileName}")
    public ResponseEntity<Resource> downloadFile(@PathVariable String fileName) throws IOException {
        Resource file = storageService.loadFileAsResource(fileName);

        return ResponseEntity.ok()
                .contentType(MediaType.APPLICATION_OCTET_STREAM)
                .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + file.getFilename() + "\"")
                .body(file);
    }
}
```    
* The corrosponding FileInfo and FileResponse classes
```java
public record FileInfo(String uri) {
}
```

```java
public record FileUploadResponse(List<String> fileUri) {
}
```
* Here the downloadFile method uses the Resource and ResponseEntity.

## Testing
* Go to browser and paste http://localhost:8080/download/<File-name>. It will download the file
* For listing all files: 
    ```bash
    curl http://localhost:8080/files
    ```
* For upload
    ```bash
    curl -v -F 'file1=@"E:/rush/frontend/package.json"' http://localhost:8080/upload
    ```
    Here the file1 is the multipart attribute(RequestBody)