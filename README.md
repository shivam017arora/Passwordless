# APIs

## 0. Generete Custodial Wallet for an issuer (optional / FLOW only)
### API Example Input:
```
email: abc@gmail.com
password: ****
```
Generates a private, public key pair in MongoDB. (custodial wallet)
Output:
```
Successful, 200
```

## 1. API for Signing up an issuer
###  API Example Input:
```
public_key: <string>
```
Saves the Issuer with metadta in MongoDB. (public key, signature)
Output:
```
encryptedPassword: <string>
```

# 2. API for Integrating Sign Up Button on 3rd party app
### API Example Input:
```
public_key: <string>,
signature: <string>
```
Checks in MongoDB for if Issuer exists, after validating the signature
Output:
```
qr: <qr code image> (public key encoded, metadata to invoke callback ZK server to save the requested args received) 
```

# 3. ZK Server API, for Integrating Sign Up Button on 3rd party app

**On ZK Phone, **
### API Example Input:
```
qr: <image> (public key encoded, metadata has callback url which calls ZK server to save the requested args received and encryptedPassword of domain registered) 
```
 Send request to callback url i.e ZK Server which sends
```
EncryptedSecret: <string> (public key encoded **DID** (which is randomly generated)),
domainPublicKey: <string>
encryptedPassword: <string>
usersPublicKey <string> (this is mobile app guy)
```
Output:
```
Successful, 200 (from ZK Server) which will check domainPublicKey and encryptedPassword match record and save the message with mobile app's user's public key. 
```
**On Device, DID is saved as user's identity which is never transmitted over the network i.e never leaves the phone's memory.**


# 4. API for Integrating Sign In Button on 3rd party app

**On ZK Phone, **
### API Example Input:
```
qr: <image> (EncryptedSecret, encoded) 
```

ZK App will generate proof given:
Public Inputs:
  1. F (encryption function) (known)
  2. Hash (algortithm, most probably sha3 for eth) (known)
  3. EncryptedMessage, from QR code (received)
Private Input:
  1. Private Key (Secret Key)
  2. Salt
```
So the zksnark should have a system of constraints that implement the following:
𝐻−Hash(𝑆)==0encryptedSecret−f(𝑆,salt)==0
```
OffChain ZK App will do this:
```
const { proof, publicSignals } = await snarkjs.groth16.fullProve(
  privateVariables, //privateVariables
  "circuits/quadratic.wasm",
  "circuits/quadratic.zkey"
);
// Construct the raw calldata to be sent to the verifier contract
const rawcalldata = await snarkjs.groth16.exportSolidityCallData(
  proof,
  publicSignals
);
jsonCalldata = JSON.parse("[" + rawcalldata + "]");
console.log("JSON Calldata: ", jsonCalldata);
```

Then ZK App will send the proof to the smart contract:
```
await verifier.verifyProof(
  jsonCalldata[0],
  jsonCalldata[1],
  jsonCalldata[2],
  jsonCalldata[3]
)
```
ZK Server will listen to the event from this transaction and check if user is authenticated and return the response to 3rd party app. 

Output:
```
tx: <tx hash>
```
