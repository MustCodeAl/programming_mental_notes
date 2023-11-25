TLS and SSL Handshakes
In order to establish an encrypted connection, the client and server must undergo the handshake process. Fortunately for us, TLS and SSL handshakes are mostly similar in their steps.

To break it down further, we might observe the following occur during our traffic analysis efforts.
	1	Client Hello - The initial step is for the client to send its hello message to the server. This message contains information like what TLS/SSL versions are supported by the client, a list of cipher suites (aka encryption algorithms), and random data (nonces) to be used in the following steps.
	2	Server Hello - Responding to the client Hello, the server will send a Server Hello message. This message includes the server's chosen TLS/SSL version, its selected cipher suite from the client's choices, and an additional nonce.
	3	Certificate Exchange - The server then sends its digital certificate to the client, proving its identity. This certificate includes the server's public key, which the client will use to conduct the key exchange process.
	4	Key Exchange - The client then generates what is referred to as the premaster secret. It then encrypts this secret using the server's public key from the certificate and sends it on to the server.
	5	Session Key Derivation - Then both the client and the server use the nonces exchanged in the first two steps, along with the premaster secret to compute the session keys. These session keys are used for symmetric encryption and decryption of data during the secure connection.
	6	Finished Messages - In order to verify the handshake is completed and successful, and also that both parties have derived the same session keys, the client and server exhange finished messages. This message contains the hash of all previous handshake messages and is encrypted using the session keys.
	7	Secure Data Exchange - Now that the handshake is complete, the client and the server can now exchange data over the encrypted channel.
We can also look at this from a general algorithmic perspective.
Handshake Step
Relevant Calculations
Client Hello
ClientHello = { ClientVersion, ClientRandom, Ciphersuites, CompressionMethods }
Server Hello
ServerHello = { ServerVersion, ServerRandom, Ciphersuite, CompressionMethod }
Certificate Exchange
ServerCertificate = { ServerPublicCertificate }
Key Exchange
	•	ClientDHPrivateKey
	•	ClientDHPublicKey = DH_KeyGeneration(ClientDHPrivateKey)
	•	ClientKeyExchange = { ClientDHPublicKey }
	•	ServerDHPrivateKey
	•	ServerDHPublicKey = DH_KeyGeneration(ServerDHPrivateKey)
	•	ServerKeyExchange = { ServerDHPublicKey }
Premaster Secret
	•	PremasterSecret = DH_KeyAgreement(ServerDHPublicKey, ClientDHPrivateKey)
	•	PremasterSecret = DH_KeyAgreement(ClientDHPublicKey, ServerDHPrivateKey)
Session Key Derivation
MasterSecret = PRF(PremasterSecret, "master secret", ClientNonce + ServerNonce)

KeyBlock = PRF(MasterSecret, "key expansion", ServerNonce + ClientNonce)
Extraction of Session Keys
	•	ClientWriteMACKey = First N bytes of KeyBlock
	•	ServerWriteMACKey = Next N bytes of KeyBlock
	•	ClientWriteKey = Next N bytes of KeyBlock
	•	ServerWriteKey = Next N bytes of KeyBlock
	•	ClientWriteIV = Next N bytes of KeyBlock
	•	ServerWriteIV = Next N bytes of KeyBlock
Finished Messages
FinishedMessage = PRF(MasterSecret, "finished", Hash(ClientHello + ServerHello))
