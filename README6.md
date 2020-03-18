
import static java.nio.charset.StandardCharsets.ISO_8859_1;
import static org.apache.commons.codec.binary.Base64.decodeBase64;
import static org.apache.commons.codec.binary.Base64.encodeBase64;

import java.math.BigInteger;
import java.nio.ByteBuffer;
import java.security.MessageDigest;
import java.util.Collection;
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Map;
import java.util.Set;
import java.util.TreeSet;
import java.util.function.Supplier;

public interface KMSProvider {

    String providerName();

    String providerKey();

    Map<String, String> metadata();

    Supplier<KMSResponse> generateDataKey(KMSRequest request);

    Supplier<KMSResponse> encryptDataKey(KMSRequest request);

    Supplier<KMSResponse> decryptDataKey(KMSRequest request);

    String match(Map<String, String> request);

    /**
     * Calculates estimated size in bytes
     *
     * @param kmsResponse
     * @return
     */
    static int capacity(final KMSResponse kmsResponse) {
        final String providerName = kmsResponse.metadata().get("PROVIDER-NAME");
        final String region = kmsResponse.metadata().get("PROVIDER-REGION");
        final String providerId = providerId(kmsResponse.metadata().get("PROVIDER-KEY"));
        return (providerName + region + providerId + kmsResponse.encryptedDataKey()).getBytes(ISO_8859_1).length + 16;
    }

    /**
     * Packs all encrypted data keys into byte buffer
     *
     * @param request
     * @return
     */
    static String pack(final Collection<KMSResponse> request) {

        final Set<String> hashKeys = new TreeSet<>();
        final Map<String, KMSResponse> map = new HashMap<>();

        int capacity = 4;

        // Put all response objects into map and add the hash into sorted set.
        for (final KMSResponse kmsResponse : request) {
            final String hash = hash(kmsResponse);
            hashKeys.add(hash);
            map.put(hash, kmsResponse);
            capacity += capacity(kmsResponse);
        }

        final ByteBuffer buffer = ByteBuffer.allocate(capacity);

        // Write number of elements
        buffer.putInt(hashKeys.size());

        // Now collection is in sorted order
        for (final String hash : hashKeys) {

            final KMSResponse res = map.get(hash);
            final String providerName = res.metadata().get("PROVIDER-NAME");
            final String region = res.metadata().get("PROVIDER-REGION");
            final String providerId = providerId(res.metadata().get("PROVIDER-KEY"));

            // Size of the provider name
            buffer.putInt(providerName.getBytes(ISO_8859_1).length);

            // Provider name
            buffer.put(providerName.getBytes(ISO_8859_1));

            // Size of the region
            buffer.putInt(region.getBytes(ISO_8859_1).length);

            // Region
            buffer.put(region.getBytes(ISO_8859_1));

            // Size of the key Id
            buffer.putInt(providerId.getBytes(ISO_8859_1).length);

            // Key Id
            buffer.put(providerId.getBytes(ISO_8859_1));

            // Size of the provider encrypted data key
            buffer.putInt(res.encryptedDataKey().getBytes(ISO_8859_1).length);

            // provider encrypted data key
            buffer.put(res.encryptedDataKey().getBytes(ISO_8859_1));

        }

        return new String(encodeBase64(buffer.array()), ISO_8859_1);
    }

    /**
     * Unpacks string into encrypted data keys and corresponding provider's hash function.
     *
     * @param request
     * @return
     */
    static Map<String, String> unpack(final String request) {

        final Map<String, String> unpacked = new LinkedHashMap<>();
        final ByteBuffer buffer = ByteBuffer.wrap(decodeBase64(request));

        // Read number of elements
        final int numberOfElements = buffer.getInt();

        // Loop for each element
        for (int i = 0; i < numberOfElements; i++) {

            // Size of the provider name
            final int providerNameSize = buffer.getInt();

            final byte[] providerNameBtr = new byte[providerNameSize];

            buffer.get(providerNameBtr);

            // Provider name
            final String providerName = new String(providerNameBtr, ISO_8859_1);

            // Size of the region
            final int regionSize = buffer.getInt();

            final byte[] regionBtr = new byte[regionSize];

            buffer.get(regionBtr);

            // Region
            final String region = new String(regionBtr, ISO_8859_1);

            // Size of the provider id
            final int providerIdSize = buffer.getInt();

            final byte[] providerIdBtr = new byte[providerIdSize];

            buffer.get(providerIdBtr);

            // Provider id
            final String providerId = new String(providerIdBtr, ISO_8859_1);

            // Size of the provider encrypted data key
            final int encryptedDataKeySize = buffer.getInt();

            final byte[] encryptedDataKeyBtr = new byte[encryptedDataKeySize];

            buffer.get(encryptedDataKeyBtr);

            // Provider encrypted data key
            final String encryptedDataKey = new String(encryptedDataKeyBtr, ISO_8859_1);

            unpacked.put(hash(providerName, region, providerId), encryptedDataKey);

        }

        return unpacked;
    }

    /**
     * Java program to calculate SHA hash value
     *
     * @param metadata
     * @return
     */
    static String hash(final KMSResponse response) {
        return hash(response.metadata());
    }

    /**
     * Java program to calculate SHA hash value
     *
     * @param metadata
     * @return
     */
    static String hash(final KMSProvider provider) {
        return hash(provider.metadata());
    }

    /**
     * Java program to calculate SHA hash value
     *
     * @param metadata
     * @return
     */
    static String hash(final Map<String, String> metadata) {
        return hash(metadata.get("PROVIDER-NAME"), metadata.get("PROVIDER-REGION"),
                providerId(metadata.get("PROVIDER-KEY")));
    }

    /**
     * Hash function for creating unique string out of each KMS provider.
     *
     * @param providerName
     * @param region
     * @param providerKey
     * @return
     */
    static String hash(final String providerName, final String region, final String providerKey) {

        try {

            // Static getInstance method is called with hashing SHA
            final MessageDigest md = MessageDigest.getInstance("SHA-256");

            // Only take the last unique part of the key/alias
            final String key = providerId(providerKey);

            // digest() method called to calculate message digest of an input and return array of byte
            final byte[] hash = md.digest((providerName + region + key).getBytes(ISO_8859_1));

            // Convert byte array into signum representation
            final BigInteger number = new BigInteger(1, hash);

            // Convert message digest into hex value
            final StringBuilder hexString = new StringBuilder(number.toString(16));

            // Pad with leading zeros
            while (hexString.length() < 32) {
                hexString.insert(0, '0');
            }

            // Return the hex string
            return hexString.toString();

        } catch (final Exception e) {
            throw new IllegalStateException(e);
        }

    }

    /**
     * consider only the unique part of the provider key
     *
     * @param providerKey
     * @return
     */
    static String providerId(final String providerKey) {

        // Using alias ?? using alias is not recommended
        if (providerKey.toLowerCase().contains("alias")) {
            return providerKey;
        }

        // In case using AWS ARN with key id, only take the key id part to make this unique.
        return providerKey.contains("/") ? providerKey.substring(providerKey.indexOf("/") + 1) : providerKey;
    }

}
