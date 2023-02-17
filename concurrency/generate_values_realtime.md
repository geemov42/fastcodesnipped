When you need to generate values fastly in a concurrent envirronment, you can use this pattern :

The generic generator :
```java
public class RealTimeGenerator<T> {

    private final Object mutex = new Object();
    private Boolean isPushInProgress = false;

    private final valueGenerator<T> valueGenerator;
    private final AfterPush afterPush;

    protected long valuesGeneratedByPush;

    private final ConcurrentLinkedQueue<T> generatedDataQueue = new ConcurrentLinkedQueue<>();

    public RealTimeGenerator(RealTimeGenerator.valueGenerator<T> valueGenerator, AfterPush afterPush, long valuesGeneratedByPush) {
        this.valueGenerator = valueGenerator;
        this.afterPush = afterPush;
        this.valuesGeneratedByPush = valuesGeneratedByPush;
    }

    public T pullNextValue() {

        T value;

        do {
            this.checkQueue();
            value = generatedDataQueue.poll();

        } while (Objects.isNull(value) || this.checkQueue());

        return value;
    }

    private boolean checkQueue() {

        if (this.isPushInProgress) {
            return false;
        }

        if (this.generatedDataQueue.isEmpty()) {
            pushNextValues();
            return false;
        }

        return false;
    }

    /**
     * The queue is empty, we need to filled it
     */
    private void pushNextValues() {

        synchronized (mutex) {

            if (!this.isPushInProgress) {
                try {
                    this.isPushInProgress = true;
                    this.filledQueue();
                    this.afterPush.executeAfterPush();
                } finally {
                    this.isPushInProgress = false;
                }
            }
        }
    }

    /**
     * Filled queue
     */
    private void filledQueue() {

        for (int i = 1; i <= valuesGeneratedByPush; i++) {

            T value = this.valueGenerator.generateValue(i);
            this.generatedDataQueue.add(value);
        }
    }

    /**
     * clear the cache
     */
    public void clearQueue() {

        this.generatedDataQueue.clear();
    }

    @FunctionalInterface
    public interface valueGenerator<T> {
        T generateValue(int currentPosition);
    }

    @FunctionalInterface
    public interface AfterPush {

        void executeAfterPush();
    }
}
```

The data generator (based on ssin) :
```java
public class SsinRealTimeGenerator {

    private static final Object mutex = new Object();
    private LocalDate currentLocalDate;

    private RealTimeGenerator<String> realTimeGenerator;

    private static volatile SsinRealTimeGenerator instance;

    public static SsinRealTimeGenerator getExistingInstance() {
        return instance;
    }

    /**
     * Use this method to create a static instance of ssin generator in concurrent env
     *
     * @param firstDateSsin : "1986-07-20"
     * @param ssinByDay
     * @return
     */
    public static SsinRealTimeGenerator initializeInstance(String firstDateSsin, int ssinByDay) {

        LocalDate firstDateSsinLocalDate = LocalDate.parse(
                firstDateSsin,
                DateTimeFormatter.ISO_DATE
        );

        SsinRealTimeGenerator result = instance;

        if (result == null) {
            synchronized (mutex) {
                result = instance;
                if (result == null) {

                    instance = result = new SsinRealTimeGenerator(firstDateSsinLocalDate, ssinByDay);
                }
            }
        }

        return result;
    }

    private SsinRealTimeGenerator(LocalDate firstDateForSSIN, int ssinByDay) {

        this.realTimeGenerator = new RealTimeGenerator<>(
                this::generateSSIN,
                this::executeAfterPush,
                ssinByDay
        );

        this.setCurrentLocalDate(firstDateForSSIN);
    }

    public void setCurrentLocalDate(LocalDate firstDateForSSIN) {

        if (firstDateForSSIN == null) {
            this.currentLocalDate = LocalDate.of(1900, Month.JANUARY, 1);
        } else {
            this.currentLocalDate = firstDateForSSIN;
        }
    }

    /**
     * generate ssin with input
     *
     * @param idx
     * @return
     */
    public String generateSSIN(int idx) {

        String serialFormatted = String.format("%03d", idx);

        String ssin = currentLocalDate.format(DateTimeFormatter.ofPattern("yyMMdd")) + serialFormatted;

        long ssinNumber = 0;
        if (currentLocalDate.get(YEAR) < 2000) {
            ssinNumber = Long.parseLong(ssin);
        } else {
            ssinNumber = Long.parseLong("2" + ssin);
        }

        int checksum = (int) (97 - (ssinNumber % 97));

        return ssin + String.format("%02d", checksum);
    }

    public void executeAfterPush() {
        currentLocalDate = currentLocalDate.plusDays(1);
    }

    public String pullNextValue() {
        return this.realTimeGenerator.pullNextValue();
    }

    public void clearQueue() {
        this.realTimeGenerator.clearQueue();
    }
}
```

And finally a usage : 
```java
@BeforeAll
public static void init() {
    SsinRealTimeGenerator.initializeInstance("1985-04-24", 170);
}

@Test
public void testSharedInstance() {

    SsinRealTimeGenerator ssinGeneratorRealTime = SsinRealTimeGenerator.getExistingInstance();

    for (int i = 0; i < 1000000; i++) {
        System.out.println(ssinGeneratorRealTime.pullNextValue());
    }
}
```