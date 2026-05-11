# Scaffold New Scheduler Job

Create a scheduled job following dept44 patterns. Use existing schedulers in this repo as reference.

## Arguments
- `$ARGUMENTS` — the job name and description (e.g. "ExpiredNotificationCleaner - removes notifications past their TTL")

## What to create

Examine existing `service/scheduler/` packages to match the exact style.

### 1. Scheduler (`service/scheduler/{jobname}/`)
- `{JobName}Scheduler.java`
- `@Component`
- `@Dept44Scheduled` (NOT `@Scheduled`) with:
  - `cron = "${scheduler.{jobname}.cron}"`
  - `name = "${scheduler.{jobname}.name}"`
  - `lockAtMostFor = "${scheduler.{jobname}.lock-at-most-for}"`
  - `maximumExecutionTime = "${scheduler.{jobname}.maximum-execution-time}"`
- `@Value` fields for each config property
- Constructor injection of `{JobName}Worker` and `Dept44HealthUtility`
- Logger: `private static final Logger LOG = LoggerFactory.getLogger({JobName}Scheduler.class)`
- The `execute()` method orchestrates only — delegates actual work to Worker
- Sets health indicator unhealthy on failure via `Dept44HealthUtility`

### 2. Worker (`service/scheduler/{jobname}/`)
- `{JobName}Worker.java` — `@Component`
- Contains the actual business logic
- Constructor injection of required repositories/integrations
- Per-item error handling in loops (catch + log, don't stop batch)

### 3. Application config
- Add to `application.yml`:
  ```yaml
  scheduler.{jobname}:
    cron: "0 0 2 * * *"          # Adjust as needed
    name: "{jobname}"
    lock-at-most-for: PT30M
    maximum-execution-time: PT25M
  ```
- Add equivalent to `application-junit.yml` with `-` as cron (disabled in tests)

### 4. Tests
- `{JobName}SchedulerTest.java`:
  - `@ExtendWith(MockitoExtension.class)`
  - `@Mock` for Worker and `Dept44HealthUtility`
  - `@InjectMocks` for the Scheduler
  - `@BeforeEach` to set `@Value` fields via `ReflectionTestUtils.setField()`
  - Test happy path and exception cases
  - Comments: `// ARRANGE`, `// Act`, `// Verify`
  - `verifyNoMoreInteractions(...)` on all mocks

- `{JobName}WorkerTest.java`:
  - `@ExtendWith(MockitoExtension.class)`
  - `@Mock` for repositories/integrations
  - `@InjectMocks` for the Worker
  - Test processing logic, including error scenarios
  - `ArgumentCaptor` to verify exact items processed

### 5. Dependencies (`pom.xml`)
- Ensure `dept44-starter-scheduler` is present as a dependency

## Important
- Look at existing schedulers in this repo for exact patterns
- Use `@Dept44Scheduled`, never `@Scheduled`
- Use `final` on all variables and parameters
- No Lombok, no wildcard imports
- Error handling must be per-item — one failure should not stop the entire batch
