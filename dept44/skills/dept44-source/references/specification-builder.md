# Specification / Filtering Pattern

## When to use

Use JPA Specifications when your repository queries need **dynamic filters** — where the WHERE clause changes based on which parameters the caller provides. If your query is always the same shape (e.g., `findByMunicipalityIdAndId`), a derived query method is simpler and preferred. Specifications shine when you have optional filters, search across multiple fields, or need complex joins.

Used in 67+ repos. Real examples:
- `<repos>/api-service-checklist/.../specification/SpecificationBuilder.java` (simple)
- `<repos>/api-service-support-management/.../util/SpecificationBuilder.java` (advanced)

## Simple SpecificationBuilder (from checklist)

Static utility class with methods returning `Specification<T>`:

```java
public class SpecificationBuilder<T> {

    public Specification<T> buildMunicipalityIdFilter(final String value) {
        return (entity, cq, cb) -> nonNull(value)
            ? cb.equal(entity.get(EMPLOYEE).get(COMPANY).get(MUNICIPALITY_ID), value)
            : cb.and(); // no-op predicate when value is null
    }

    public Specification<T> buildEmployeeNameFilter(final String value) {
        return (entity, cq, cb) -> nonNull(value)
            ? cb.like(cb.lower(cb.concat(cb.concat(
                entity.get(EMPLOYEE).get(FIRST_NAME), cb.literal(" ")),
                entity.get(EMPLOYEE).get(LAST_NAME))),
                "%" + value.toLowerCase() + "%")
            : cb.and();
    }

    public Specification<T> buildNotCompletedFilter() {
        return (entity, cq, cb) -> cb.equal(entity.get(COMPLETED), false);
    }

    public Specification<T> distinct() {
        return (entity, cq, cb) -> {
            cq.distinct(true);
            return null;
        };
    }
}
```

Key patterns:
- Return `cb.and()` (empty conjunction) when a filter value is null — this becomes a no-op predicate
- Use `entity.get("field").get("nestedField")` for traversing relationships
- `cb.literal(" ")` for literal strings in expressions
- `distinct()` sets `cq.distinct(true)` and returns `null`

## Repository Integration

Compose specifications in repository default methods:

```java
@Repository
@CircuitBreaker(name = "employeeChecklistRepository")
public interface EmployeeChecklistRepository extends
    JpaRepository<EmployeeChecklistEntity, String>,
    JpaSpecificationExecutor<EmployeeChecklistEntity> {

    default Page<EmployeeChecklistEntity> findAllByParameters(
            final FilterParameters params, final Pageable pageable) {
        return findAll(Specification
            .allOf(withMunicipalityId(params.getMunicipalityId())
                .and(withEmployeeName(params.getEmployeeName()))
                .and(withNonCompleted())
                .and(distinct())),
            pageable);
    }
}
```

## Advanced: Subqueries (from support-management)

For access control or many-to-many filtering where you need to compare counts:

```java
public static Specification<ErrandEntity> hasAllowedMetadataLabels(
        final Set<MetadataLabelEntity> allowedLabels) {
    return (entity, cq, cb) -> {
        if (isEmpty(allowedLabels)) return cb.and();

        final var totalLabelsSubquery = cq.subquery(Long.class);
        final var totalRoot = totalLabelsSubquery.from(ErrandEntity.class);
        final var totalJoin = totalRoot.join("labels", JoinType.LEFT);
        totalLabelsSubquery.select(cb.count(totalJoin))
            .where(cb.equal(totalRoot.get("id"), entity.get("id")));

        final var allowedSubquery = cq.subquery(Long.class);
        final var allowedRoot = allowedSubquery.from(ErrandEntity.class);
        final var allowedJoin = allowedRoot.join("labels", JoinType.LEFT);
        allowedSubquery.select(cb.count(allowedJoin))
            .where(cb.and(
                cb.equal(allowedRoot.get("id"), entity.get("id")),
                allowedJoin.in(allowedLabels)));

        return cb.equal(totalLabelsSubquery, allowedSubquery);
    };
}
```

## Service Usage

```java
public PagedErrandsResponse getErrands(final String namespace, final String municipalityId,
        final ErrandFilterParameters params) {
    final var spec = Specification.where(withNamespace(namespace))
        .and(withMunicipalityId(municipalityId))
        .and(withStatus(params.getStatus()))
        .and(withCategory(params.getCategory()));

    final var page = errandsRepository.findAll(spec, params.toPageable());
    return toPagedErrandsResponse(page);
}
```
