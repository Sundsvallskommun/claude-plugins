# Pagination Pattern

## When to use

Add pagination whenever a list endpoint could return a large number of results. Spring Data's `Pageable` and `Page<>` handle the mechanics — the dept44-specific part is how to wire the response DTO and convert between 0-based (JPA) and 1-based (API) page numbers.

dept44 provides `PagingMetaData` and `PagingAndSortingMetaData` in `dept44-models`:
- `<repos>/dept44/dept44-models/src/main/java/se/sundsvall/dept44/models/api/paging/PagingMetaData.java`
- `<repos>/dept44/dept44-models/src/main/java/se/sundsvall/dept44/models/api/paging/PagingAndSortingMetaData.java`

## 1. Response DTO — extend PagingMetaData

```java
public class PagedEntitiesResponse extends PagingMetaData {

    private List<Entity> entities;

    public static PagedEntitiesResponse create() {
        return new PagedEntitiesResponse();
    }

    public PagedEntitiesResponse withEntities(final List<Entity> entities) {
        this.entities = entities;
        return this;
    }

    // getter, setter, equals, hashCode, toString
}
```

Extend `PagingAndSortingMetaData` instead when sort info should be included in the response.

## 2. Resource — accept Pageable

```java
@GetMapping(produces = APPLICATION_JSON_VALUE)
ResponseEntity<PagedEntitiesResponse> getEntities(
    @Parameter(name = "municipalityId") @PathVariable @ValidMunicipalityId final String municipalityId,
    @ParameterObject final Pageable pageable) {

    return ok(entityService.getEntities(municipalityId, pageable));
}
```

`@ParameterObject` tells springdoc to expand `Pageable` into query params (page, size, sort) in the OpenAPI spec. No `@RequestParam` needed — Spring binds `Pageable` automatically.

## 3. Service — pass Pageable to repository

```java
public PagedEntitiesResponse getEntities(final String municipalityId, final Pageable pageable) {
    final var page = entityRepository.findAllByMunicipalityId(municipalityId, pageable);
    return toPagedEntitiesResponse(page);
}
```

## 4. Repository — return Page

```java
@CircuitBreaker(name = "entityRepository")
public interface EntityRepository extends JpaRepository<EntityRecord, String>,
    JpaSpecificationExecutor<EntityRecord> {

    Page<EntityRecord> findAllByMunicipalityId(String municipalityId, Pageable pageable);
}
```

## 5. Mapper — Page to response DTO

The critical detail: `page.getNumber()` is **0-based** but the API uses **1-based** page numbers.

```java
public static PagedEntitiesResponse toPagedEntitiesResponse(final Page<EntityRecord> page) {
    final var response = PagedEntitiesResponse.create()
        .withEntities(page.getContent().stream()
            .map(EntityMapper::toEntity)
            .toList());

    response.setPage(page.getNumber() + 1);  // 0-based -> 1-based
    response.setLimit(page.getSize());
    response.setCount(page.getNumberOfElements());
    response.setTotalRecords(page.getTotalElements());
    response.setTotalPages(page.getTotalPages());

    return response;
}
```

If using `PagingAndSortingMetaData`, use the convenience method instead:
```java
new PagingAndSortingMetaData().withPageData(page);
```

## 6. Sort field mapping (when API names differ from JPA names)

```java
public static PageRequest toPageRequest(final FilterParameters params) {
    final var sortProperties = mapSortProperty(params.getSortBy());
    final var sort = Sort.by(params.getSortDirection(), sortProperties);
    return PageRequest.of(params.getPage() - 1, params.getLimit(), sort);
}

private static String[] mapSortProperty(final String apiField) {
    return switch (apiField) {
        case "employeeName" -> new String[]{"employee.firstName", "employee.lastName"};
        case "managerName" -> new String[]{"employee.manager.firstName", "employee.manager.lastName"};
        default -> new String[]{apiField};
    };
}
```
