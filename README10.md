
@RunWith(SpringRunner.class)
@ContextConfiguration(classes = { EncryptionConfig.class, Application.class }, initializers = { MpropzApplicationContextInitializer.class })
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@TestPropertySource(properties = { "swagger.enabled=true", "cxf.openapi.enabled=true", "tes.svcs.client.asymmetric.key.provider=aws-sm",
		"tes.svcs.client.symmetric.key.provider=aws-sm" })
@ActiveProfiles("test")
public class EncryptionServiceImplAwsKMSTest {

	@LocalServerPort
	private int port;

	@Value("${security.user.name}")
	private String username;

	@Value("${security.user.password}")
	private String password;

	@Value("classpath:${tes.svcs.keyPair.file:config/dev/asymmetric-keys.json}")
	private Resource keyPairJsonFile;

	@Autowired
	private EncryptionService encryptionService;

	@Before
	public void setup() {
	}

	@Test(expected = EncryptionException.class)
	public void encryptTest() throws Exception {
		final PublicKeyResponse publicKeyResponse = encryptionService.fetchPublicKey();
		assertNotNull(publicKeyResponse);
		assertNotNull(publicKeyResponse.publicKey());
	}
	
}

