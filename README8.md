
@Log4j2
public final class AsyncExecutor {

    private final Collection<Supplier<KMSResponse>> suppliers = new LinkedHashSet<>();

    public AsyncExecutor() {
    }

    public AsyncExecutor add(final Supplier<KMSResponse> supplier) {
        suppliers.add(supplier);
        return this;
    }

    public AsyncExecutor add(final Collection<Supplier<KMSResponse>> suppliers) {
        suppliers.addAll(suppliers);
        return this;
    }

    public final KMSResponse executeWinner(final ThreadPoolTaskExecutor taskExecutor) {

        final CountDownLatch countDown = new CountDownLatch(1);
        final AtomicReference<KMSResponse> resultRef = new AtomicReference<>();

        suppliers.forEach(req -> {
            taskExecutor.execute(() -> {
                if (resultRef.compareAndSet(null, req.get())) {
                    countDown.countDown();
                }
            });
        });

        try {
            countDown.await();
        } catch (final InterruptedException exception) {
            log.error(format("InterruptedException occured in evaluteWinner(), %s", exception.getMessage()), exception);
        }

        return resultRef.get();

    }

    public final Collection<KMSResponse> executeAll(final ThreadPoolTaskExecutor taskExecutor) {

        final List<Future<KMSResponse>> results = new ArrayList<>();

        suppliers.forEach(req -> {
            results.add(taskExecutor.submit(() -> req.get()));
        });

        boolean isAnyRunning = false;

        final Collection<KMSResponse> response = new ArrayList<>();

        do {

            if (results.isEmpty()) {
                break;
            }

            final Iterator<Future<KMSResponse>> iterator = results.iterator();

            while (iterator.hasNext()) {

                final Future<KMSResponse> future = iterator.next();

                if (!future.isDone()) {
                    isAnyRunning = true;
                    continue;
                }

                try {

                    response.add(future.get());

                } catch (final InterruptedException exception) {
                    log.error(format("InterruptedException occured in executeAll(), %s", exception.getMessage()),
                            exception);
                } catch (final ExecutionException exception) {
                    log.error(format("ExecutionException occured in executeAll(), %s", exception.getMessage()),
                            exception);
                } finally {
                    iterator.remove();
                }

            }

        } while (isAnyRunning);

        return response;

    }

}
