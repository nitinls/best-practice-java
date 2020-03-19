<script>
function myFunction() {
  var num = 15;
  var a = num.toString();
  var b = num.toString(2);
  var c = num.toString(8);
  var d = num.toString(16);

  var n = a + "<br>" + b + "<br>" + c + "<br>" + d;

  document.getElementById("demo").innerHTML=n;
}
</script>

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
