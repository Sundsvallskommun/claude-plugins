# File Upload / MultipartFile Pattern

## When to use

Use this pattern when your endpoint receives files from clients, or when you need to forward file data between services. The dept44 ecosystem has two common scenarios: (1) receiving uploads via REST and (2) adapting data from one format (Base64, database Blob) into `MultipartFile` for downstream Feign calls.

Real examples:
- `<repos>/api-service-support-management/.../ErrandAttachmentsResource.java`
- `<repos>/api-service-document/.../DocumentResource.java`
- `<repos>/api-service-case-data/.../Base64MultipartFile.java`
- `<repos>/api-service-support-management/.../BlobMultipartFile.java`

## 1. Single file upload endpoint

```java
@PostMapping(consumes = MULTIPART_FORM_DATA_VALUE, produces = ALL_VALUE)
@Operation(summary = "Create attachment")
ResponseEntity<Void> createAttachment(
    @Parameter(name = "municipalityId") @ValidMunicipalityId @PathVariable final String municipalityId,
    @Parameter(name = "errandId") @ValidUuid @PathVariable final String errandId,
    @NotNull @RequestPart("errandAttachment") final MultipartFile errandAttachment) {

    final var id = attachmentService.create(municipalityId, errandId, errandAttachment);

    return created(fromPath("/{municipalityId}/errands/{errandId}/attachments/{id}")
        .buildAndExpand(municipalityId, errandId, id).toUri())
        .header(CONTENT_TYPE, ALL_VALUE)
        .build();
}
```

Key points:
- `consumes = MULTIPART_FORM_DATA_VALUE`
- `@RequestPart("fieldName")` binds to the multipart field name
- `@NotNull` validates the file part is present

## 2. Multiple files + JSON metadata

When you need a JSON object alongside files, the JSON part must be a `String` and manually deserialized — Spring cannot auto-deserialize JSON within a multipart request:

```java
@PostMapping(consumes = MULTIPART_FORM_DATA_VALUE, produces = ALL_VALUE)
ResponseEntity<Void> create(
    @PathVariable @ValidMunicipalityId final String municipalityId,
    @RequestPart("document") final String documentString,
    @RequestPart("documentFiles") @ValidContentType final List<MultipartFile> files) {

    final var body = objectMapper.readValue(documentString, DocumentCreateRequest.class);
    validate(body);
    // ...
}
```

## 3. Custom MultipartFile — byte array adapter

Use when converting Base64-encoded content or in-memory data to `MultipartFile` for downstream Feign calls:

```java
public class Base64MultipartFile implements MultipartFile {

    private final String name;
    private final String originalFilename;
    private final String contentType;
    private final byte[] content;

    public Base64MultipartFile(final String name, final String originalFilename,
            final String contentType, final byte[] content) {
        this.name = name;
        this.originalFilename = originalFilename;
        this.contentType = contentType;
        this.content = content != null ? content : new byte[0];
    }

    @Override public String getName() { return name; }
    @Override public String getOriginalFilename() { return originalFilename; }
    @Override public String getContentType() { return contentType; }
    @Override public boolean isEmpty() { return content.length == 0; }
    @Override public long getSize() { return content.length; }
    @Override public byte[] getBytes() { return content; }
    @Override public InputStream getInputStream() { return new ByteArrayInputStream(content); }
    @Override public void transferTo(final File dest) throws IOException {
        throw new UnsupportedOperationException("transferTo is not supported");
    }
}
```

## 4. Custom MultipartFile — database Blob adapter

Use when streaming files from JPA Blob columns to downstream services:

```java
public class BlobMultipartFile implements MultipartFile {

    private final String name;
    private final String originalFilename;
    private final String contentType;
    private final long size;
    private final Blob blob;

    // constructor...

    @Override
    public byte[] getBytes() throws IOException {
        try (final var inputStream = blob.getBinaryStream()) {
            return inputStream.readAllBytes();
        } catch (final SQLException e) {
            throw new IOException("Failed to read blob data", e);
        }
    }

    @Override
    public InputStream getInputStream() throws IOException {
        try { return blob.getBinaryStream(); }
        catch (final SQLException e) { throw new IOException("Failed to get blob input stream", e); }
    }
}
```

## 5. File validators

### Content-type validator

```java
public class ValidContentTypeConstraintValidator
        implements ConstraintValidator<ValidContentType, List<MultipartFile>> {

    @Override
    public boolean isValid(final List<MultipartFile> files, final ConstraintValidatorContext ctx) {
        return files.stream()
            .map(MultipartFile::getContentType)
            .noneMatch("application/octet-stream"::equals);
    }
}
```

### Non-empty file list validator

```java
public class ValidMultipartFilesValidator
        implements ConstraintValidator<ValidMultipartFiles, List<MultipartFile>> {

    @Override
    public boolean isValid(final List<MultipartFile> files, final ConstraintValidatorContext ctx) {
        if (isNull(files) || files.isEmpty()) {
            return true; // let @NotEmpty handle empty lists
        }
        return files.stream().allMatch(file -> nonNull(file) && !file.isEmpty());
    }
}
```
