
import static java.nio.ByteBuffer.wrap;
import static java.nio.charset.StandardCharsets.ISO_8859_1;
import static org.apache.commons.codec.binary.Base64.decodeBase64;
import static org.apache.commons.codec.binary.Base64.encodeBase64;
import static org.apache.commons.collections.MapUtils.isEmpty;

import java.util.LinkedHashMap;
import java.util.Map;
import java.util.function.Supplier;

import com.amazonaws.AmazonWebServiceRequest;
import com.amazonaws.services.kms.AWSKMS;
import com.amazonaws.services.kms.model.DecryptRequest;
import com.amazonaws.services.kms.model.DecryptResult;
import com.amazonaws.services.kms.model.EncryptRequest;
import com.amazonaws.services.kms.model.EncryptResult;
import com.amazonaws.services.kms.model.GenerateDataKeyRequest;
import com.amazonaws.services.kms.model.GenerateDataKeyResult;


import lombok.EqualsAndHashCode;

@EqualsAndHashCode(of = { "providerName", "providerKey", "region" })
public class AWSKMSProviderImpl implements KMSProvider {

    private final AWSKMS awsKMS;
    private final String providerName;
    private final String providerKey;
    private final String region;

    // incremented for design changes that break backward compatibility.
    private static final String VERSION_NUM = "1";
    // incremented for major changes to the implementation
    private static final String MAJOR_REVISION_NUM = "1";
    // incremented for minor changes to the implementation
    private static final String MINOR_REVISION_NUM = "0";
    // incremented for releases containing an immediate bug fix.
    private static final String BUGFIX_REVISION_NUM = "0";
    private static final String RELEASE_VERSION = VERSION_NUM + "." + MAJOR_REVISION_NUM + "." + MINOR_REVISION_NUM
            + "." + BUGFIX_REVISION_NUM;
    private static final String USER_AGENT = "AwsCrypto/" + RELEASE_VERSION;

    public AWSKMSProviderImpl(final AWSKMS awsKMS, final String provider, final String region, final String kmsId) {
        this.awsKMS = awsKMS;
        providerName = provider;
        this.region = region;
        providerKey = kmsId;
    }

    @Override
    public String providerName() {
        return providerName;
    }

    @Override
    public String providerKey() {
        return providerKey;
    }
    
    Supplier<PublicKeyResponse> fetchPublicKey(){
    	
    }

    @Override
    public Supplier<KMSResponse> generateDataKey(final KMSRequest request) {

        return () -> {

            final GenerateDataKeyResult result = awsKMS.generateDataKey(updateUserAgent(
                    new GenerateDataKeyRequest().withKeyId(providerKey).withNumberOfBytes(request.keyLength())));

            final byte[] rawKey = new byte[request.keyLength()];
            final byte[] encryptedKey = new byte[result.getCiphertextBlob().remaining()];

            result.getPlaintext().get(rawKey);
            result.getCiphertextBlob().get(encryptedKey);

            return new KMSResponse().dataKey(new String(encodeBase64(rawKey), ISO_8859_1))
                    .encryptedDataKey(new String(encodeBase64(encryptedKey), ISO_8859_1)).metadata(metadata());

        };

    }

    @Override
    public Supplier<KMSResponse> encryptDataKey(final KMSRequest request) {

        return () -> {

            final EncryptResult result = awsKMS.encrypt(updateUserAgent(
                    new EncryptRequest().withKeyId(providerKey).withPlaintext(wrap(decodeBase64(request.dataKey())))));

            final byte[] encryptedKey = new byte[result.getCiphertextBlob().remaining()];

            result.getCiphertextBlob().get(encryptedKey);

            return new KMSResponse().dataKey(request.dataKey())
                    .encryptedDataKey(new String(encodeBase64(encryptedKey), ISO_8859_1)).metadata(metadata());

        };
    }

    @Override
    public Supplier<KMSResponse> decryptDataKey(final KMSRequest request) {

        return () -> {

            final DecryptResult result = awsKMS.decrypt(updateUserAgent(new DecryptRequest().withKeyId(providerKey)
                    .withCiphertextBlob(wrap(decodeBase64(request.encryptedDataKey())))));

            final byte[] rawKey = new byte[result.getPlaintext().remaining()];

            result.getPlaintext().get(rawKey);

            return new KMSResponse().dataKey(new String(encodeBase64(rawKey), ISO_8859_1))
                    .encryptedDataKey(request.encryptedDataKey()).metadata(metadata());

        };

    }

    @Override
    public String match(final Map<String, String> request) {

        if (isEmpty(request)) {
            return null;
        }

        return request.get(hash(metadata()));

    }

    @Override
    public Map<String, String> metadata() {

        final Map<String, String> metadata = new LinkedHashMap<>();

        metadata.put("PROVIDER-NAME", providerName);
        metadata.put("PROVIDER-REGION", region);
        metadata.put("PROVIDER-KEY", providerId(providerKey));

        return metadata;

    }

    private <T extends AmazonWebServiceRequest> T updateUserAgent(final T request) {
        request.getRequestClientOptions().appendUserAgent(USER_AGENT);
        return request;
    }

	@Override
	public Supplier<PublicKeyResponse> fetchPublicKey() {
		// TODO Auto-generated method stub
		return null;
	}

}
