# Scheduler Pattern Reference

Reference for dept44 scheduled jobs. Look at existing schedulers in `service/scheduler/` for exact style.

## Scheduler Class

- Package: `service/scheduler/{jobname}/`
- `@Component`
- Uses `@Dept44Scheduled` (NOT `@Scheduled`) with config properties:
  - `cron`, `name`, `lockAtMostFor`, `maximumExecutionTime`
- Config values injected via `@Value("${scheduler.{jobname}.*}")`
- Health monitoring via `Dept44HealthUtility`
- Logger: `private static final Logger LOG = LoggerFactory.getLogger(...)`
- Delegates actual work to a separate `{Job}Worker` class
- Error handling: catch per-item so one failure doesn't stop the batch

## Worker Class

- `@Component` in same package as scheduler
- Contains actual business logic
- Constructor injection of repositories/integrations
- Per-item error handling in loops

## Scheduler Test — Unit Test

```java
@ExtendWith(MockitoExtension.class)
class {Job}SchedulerTest {

    private static final String MUNICIPALITY_ID = "testMunicipalityId";
    private static final String JOB_NAME = "testJobName";

    @Mock
    private ExecutionInformationRepository executionInformationRepository;
    @Mock
    private {Job}Worker worker;
    @Mock
    private Dept44HealthUtility dept44HealthUtility;

    private {Job}Scheduler scheduler;

    @BeforeEach
    void setUp() {
        // Construct manually — do NOT use @InjectMocks (scheduler uses @Value params)
        scheduler = new {Job}Scheduler(
            executionInformationRepository, worker, dept44HealthUtility,
            MUNICIPALITY_ID, JOB_NAME);
    }

    @Test
    void executeSuccess() {
        // Arrange
        final var entity = ExecutionInformationEntity.builder()
            .withMunicipalityId(MUNICIPALITY_ID)
            .withLastSuccessfulExecution(OffsetDateTime.now())
            .build();
        when(executionInformationRepository.findById(MUNICIPALITY_ID)).thenReturn(Optional.of(entity));
        when(worker.doWork(eq(entity), any())).thenReturn(true);

        // Act
        scheduler.execute();

        // Assert
        verify(worker).doWork(eq(entity), any());
        verify(executionInformationRepository).save(entity);
    }

    @Test
    void executeFailure() {
        // Arrange
        when(executionInformationRepository.findById(MUNICIPALITY_ID)).thenReturn(Optional.empty());

        // Act
        scheduler.execute();

        // Assert
        verify(executionInformationRepository).findById(MUNICIPALITY_ID);
        verifyNoMoreInteractions(executionInformationRepository);
    }
}
```

## Scheduler Test — Shedlock Integration

```java
@SpringBootTest(properties = {
    "scheduler.{jobname}.cron=* * * * * *",
    "scheduler.{jobname}.name={jobname}",
    "server.shutdown=immediate",
    "spring.lifecycle.timeout-per-shutdown-phase=0s"
})
@ActiveProfiles("junit")
class {Job}SchedulerShedlockTest {

    @Autowired
    private {Job}Worker workerMock;

    @Autowired
    private NamedParameterJdbcTemplate jdbcTemplate;

    private LocalDateTime mockCalledTime;

    @Test
    void verifyShedLock() {
        doAnswer(_ -> {
            mockCalledTime = LocalDateTime.now();
            await().forever().until(() -> false);
            return null;
        }).when(workerMock).doWork(any());

        await().until(() -> mockCalledTime != null
            && LocalDateTime.now().isAfter(mockCalledTime.plusSeconds(2)));

        await().atMost(5, TimeUnit.SECONDS)
            .untilAsserted(() -> assertThat(getLockedAt("{jobname}"))
                .isCloseTo(LocalDateTime.now(Clock.systemUTC()), within(10, ChronoUnit.SECONDS)));

        verify(workerMock, times(1)).doWork(any());
    }

    @TestConfiguration
    public static class {Job}WorkerConfiguration {
        @Bean
        @Primary
        {Job}Worker createMock() {
            return Mockito.mock({Job}Worker.class);
        }
    }
}
```

Key: Worker mock hangs forever to simulate long-running work — proves the lock prevents concurrent execution.
