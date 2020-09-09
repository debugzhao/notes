```java
public SystemStorageEntity store(InputStream inputStream, long contentLength, String contentType, String fileName) 
```

```java
 private String generateKey(String originalFilename)
```

```java
    @Override
    public void store(InputStream inputStream, long contentLength, String contentType, String keyName) {
        try {
            Files.copy(inputStream, rootLocation.resolve(keyName), StandardCopyOption.REPLACE_EXISTING);
        } catch (IOException e) {
            throw new RuntimeException("Failed to store file " + keyName, e);
        }
    }
```

```java
generateUrl(key)
```

