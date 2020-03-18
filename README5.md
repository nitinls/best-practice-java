@Bean("providers")
    public Map<String, KMSProvider> providers() {

        final Set<String> providerNames = new HashSet<>(getInstance().getAsList("wdpr.kms.providerNames", emptyList()));

        if (isEmpty(providerNames)) {
            log.error(() -> "Provider names are empty!");
            return emptyMap();
        }

        final Map<String, KMSProvider> providers = new LinkedHashMap<>();

        providerNames.forEach(providerName -> {

            if (isBlank(providerName)) {
                log.error(() -> format("Provider name is blank or null [%s]", providerName));
                return;
            }

            final String finalProviderName = providerName.trim().toLowerCase();

            switch (finalProviderName) {

            case "aws-kms":
            case "awskms": {

                final Set<String> regions = new LinkedHashSet<>(
                        getInstance().getAsList("wdpr.kms." + finalProviderName + ".regions", emptyList()));

                if (isEmpty(regions)) {
                    log.error(() -> format("Regions are empty for the provider %s", finalProviderName));
                    break;
                }

                regions.forEach(region -> {

                    if (isBlank(region)) {
                        log.error(() -> format("Region name is blank or null [%s], for the provider [%s]", region,
                                finalProviderName));
                        return;
                    }

                    final String finalRegion = region.trim().toLowerCase();

                    final List<String> kmsIdsList = getInstance()
                            .getAsList("wdpr.kms." + finalProviderName + "." + finalRegion + ".kmsIds", emptyList());

                    if (isEmpty(kmsIdsList)) {
                        log.error(() -> format("KSM IDS are empty for the provider %s, region %s", finalProviderName,
                                finalRegion));
                        return;
                    }

                    final Set<String> kmsIds = kmsIdsList.parallelStream().filter(Objects::nonNull)
                            .filter(StringUtils::isNotBlank).map(StringUtils::trim)
                            .collect(toCollection(LinkedHashSet::new));

                    if (isEmpty(kmsIds)) {
                        log.error(() -> format("KSM IDS are empty for the provider %s, region %s, after filtering",
                                finalProviderName, finalRegion));
                        return;
                    }

                    final String accessKey = getInstance().getAsString(
                            "wdpr.kms." + finalProviderName + "." + finalRegion + ".accessKey",
                            getInstance().getAsString("wdpr.kms." + finalProviderName + ".accessKey"));
                    final String secretKey = getInstance().getAsString(
                            "wdpr.kms." + finalProviderName + "." + finalRegion + ".secretKey",
                            getInstance().getAsString("wdpr.kms." + finalProviderName + ".secretKey"));
                    final int socketTimeout = getInstance().getAsInteger(
                            "wdpr.kms." + finalProviderName + "." + finalRegion + ".socketTimeout",
                            getInstance().getAsInteger("wdpr.kms." + finalProviderName + ".socketTimeout", 60000));

                    final AWSKMS awsKMS = awsKMS(finalRegion, accessKey, secretKey, socketTimeout);

                    for (final String kmsId : kmsIds) {

                        final KMSProvider provider = new AWSKMSProviderImpl(awsKMS, finalProviderName, finalRegion,
                                providerId(kmsId));

                        providers.put(hash(provider), provider);

                    }

                });

                break;

            }

            case "azure-kms":
            case "azurekms": {

                // Not implemented yet

                break;

            }

            case "aws-sm":
            case "awssm": {

                // Not implemented yet

                break;

            }

            case "vault": {

                // Not implemented yet

                break;

            }

            default: {
                // Do nothing
            }

            }

        });

        if (isEmpty(providers)) {
            log.error(() -> "Provider map is empty!");
        }

        return providers;
    }
