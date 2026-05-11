# dept44 Module Index

Framework root: `<repos>/dept44/` — `<repos>/` is wherever the Sundsvall repos are cloned (see the dept44-source SKILL.md for how to resolve it on this machine).

This is the starting point when you need to find any dept44 class. If what you're looking for isn't in the quick reference table below, use `Glob` or `Grep` to search the source tree.

## Module -> Source Path

| Starter | Source root |
|---|---|
| `dept44-starter` (core) | `<repos>/dept44/dept44-starter/src/main/java/se/sundsvall/dept44/` |
| `dept44-starter-feign` | `<repos>/dept44/dept44-starter-feign/src/main/java/se/sundsvall/dept44/configuration/feign/` |
| `dept44-starter-webclient` | `<repos>/dept44/dept44-starter-webclient/src/main/java/se/sundsvall/dept44/configuration/webclient/` |
| `dept44-starter-webservicetemplate` | `<repos>/dept44/dept44-starter-webservicetemplate/src/main/java/se/sundsvall/dept44/configuration/webservicetemplate/` |
| `dept44-starter-scheduler` | `<repos>/dept44/dept44-starter-scheduler/src/main/java/se/sundsvall/dept44/scheduling/` |
| `dept44-starter-authorization` | `<repos>/dept44/dept44-starter-authorization/src/main/java/se/sundsvall/dept44/authorization/` |
| `dept44-starter-test` | `<repos>/dept44/dept44-starter-test/src/main/java/se/sundsvall/dept44/test/` |
| `dept44-starter-jpa` | `<repos>/dept44/dept44-starter-jpa/src/main/java/` |
| `dept44-common-validators` | `<repos>/dept44/dept44-common-validators/src/main/java/se/sundsvall/dept44/common/validators/` |
| `dept44-models` | `<repos>/dept44/dept44-models/src/main/java/se/sundsvall/dept44/models/` |

## Key Classes Quick Reference

| Class / Annotation | File path |
|---|---|
| `Problem` (error throwing) | `<repos>/dept44/dept44-starter/src/main/java/se/sundsvall/dept44/problem/Problem.java` |
| `RequestId` (x-request-id propagation) | `<repos>/dept44/dept44-starter/src/main/java/se/sundsvall/dept44/requestid/RequestId.java` |
| `Constants` (default timeouts) | `<repos>/dept44/dept44-starter/src/main/java/se/sundsvall/dept44/configuration/Constants.java` |
| `FeignMultiCustomizer` | `<repos>/dept44/dept44-starter-feign/src/main/java/se/sundsvall/dept44/configuration/feign/FeignMultiCustomizer.java` |
| `OAuth2RequestInterceptor` | `<repos>/dept44/dept44-starter-feign/src/main/java/se/sundsvall/dept44/configuration/feign/interceptor/OAuth2RequestInterceptor.java` |
| `WebClientBuilder` | `<repos>/dept44/dept44-starter-webclient/src/main/java/se/sundsvall/dept44/configuration/webclient/WebClientBuilder.java` |
| `WebServiceTemplateBuilder` | `<repos>/dept44/dept44-starter-webservicetemplate/src/main/java/se/sundsvall/dept44/configuration/webservicetemplate/WebServiceTemplateBuilder.java` |
| `@Dept44Scheduled` | `<repos>/dept44/dept44-starter-scheduler/src/main/java/se/sundsvall/dept44/scheduling/Dept44Scheduled.java` |
| `Dept44HealthUtility` | `<repos>/dept44/dept44-starter-scheduler/src/main/java/se/sundsvall/dept44/scheduling/health/Dept44HealthUtility.java` |
| `AbstractAppTest` | `<repos>/dept44/dept44-starter-test/src/main/java/se/sundsvall/dept44/test/AbstractAppTest.java` |
| `PagingMetaData` | `<repos>/dept44/dept44-models/src/main/java/se/sundsvall/dept44/models/api/paging/PagingMetaData.java` |
| `PagingAndSortingMetaData` | `<repos>/dept44/dept44-models/src/main/java/se/sundsvall/dept44/models/api/paging/PagingAndSortingMetaData.java` |
| Validator annotations | `<repos>/dept44/dept44-common-validators/src/main/java/se/sundsvall/dept44/common/validators/annotation/` |
