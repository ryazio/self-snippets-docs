
 Backend code  ( Node.js )
=    
    1. Issue Attestation / Claim
    ---
    ```
    const { issueNewClaim, verifySign } = require("self");

    const claimType = 'general';
    const claimData = { 'general': {name: 'TOM', age: 62 }};
    try {
                        // method imported from SDK
        const issuedClaim = await ctrl.issueNewClaim({
            blockchain,
            requestorDID,
            claimType,
            claimData,
            attestorName,
            attestorPublicKey: PDB_publicKey,
            attestorPrivateKey: PDB_privateKey,
            attestorDID: PDB_attestorDID
        });
  
    } catch (e) {
        console.log('e :>> ', e);
    }
    ```
    2. Verify Claim ( Verify the Verifiable Presentation )
    ---
    ```
    const { challenge, token, requestorDID, userVP, clientId } = req.body;
    const { issueNewClaim, verifySign } = require("self");

    const verifyClaim = verifySign(
        JSON.stringify(vc),
        publicKey,
        proof.jws,
        blockchain
    );

    if (verifyClaim) authorizeLogin(clientID);
    ```
    3. authorizeLogin Method
    --
    ```
    const authorizeLogin = (clientId) => {
    // if client is available, generate a new AccessToken for the
    // client and send needed information to frontend via WebSocket
        wsc[clientId] ?
        wsc[clientId]
            .send(JSON.stringify(
                { 
                    'success': true, 
                    'token': 'xyz' 
                }
            )) :
        console.log("ClientId not found", clientId);
    }

    ```

4. Backend : WebSocket Handling 
    ```
    const wsc = require('../wsc.js')
wss.on('connection', (ws, req) => {

  var userID = req.url.substr(1);
  console.log('userID :>> ', userID);
  wsc[userID] = ws;
  let randChallenge = Math.random();
  let newAuth = new Auth({ 'challenge': randChallenge });
  newAuth.save();
  
  //send immediatly a feedback to the incoming connection
  const json = { 
    "endpoint": '/user/handle',
    "challenge": randChallenge,
    "domain": process.env.DOMAIN || 'http://localhost:3000',
    "attestorDID": process.env.ATTESTOR_DID,
    "attestorPublicKeyHex": process.env.PUBLIC_KEY_HEX,
  };
  ws.send(JSON.stringify(json));
});

    ```
    What is wsc.js?
    ---
    wsc.js is a helpfer file for storing the Active Web Socket connections as it is in an in-memory object. This cannot persist, meaning; once the node.js server is restarted there are no active web sockets.
    ```
    module.exports = {};
    ```
