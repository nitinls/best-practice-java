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
        
        
