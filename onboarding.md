
SELF ID - Platform Onboarding Doc
===

1. Create an account on [SITE LINK] 
2. Input your Email and desired Password, submit.
3. On verification screen, input the verification code that you got in your email and submit.
4. Enter Details about you as an platform/attestor, Name, website and Logo/photo.
5. Select a category that your platform falls into, click next.
6. Select "Required for Login" molecules by selecting relevant categories and attestors ( might not be applicable for now as there aren't any attestors )
7. Next step will Ask if you want to generate a new mnemonic or use an existing mnemonic, you can use this mnemonic for now [funded mnemonic here]
8. it will now register you on the omni layer blockchain. After the transaction you will have to wait for about 15 minutes approximately until your transaction is confirmed on the chain and then finally you will have your DID address.
9. We can start integrating the frontend code as shown in the onboarding process.
10. Frontend Widget Code 
    ```
    <script type="text/javascript" src="https://davidshimjs.github.io/qrcodejs/qrcode.min.js"></script>
    <script type="text/javascript" src="https://some-publically-hosted-SELF-ID-address.com/btn.js"></script>
    <script>
        createBtn("wss://your-backend-endpoint-wss-enabled.com");
    </script>
    ```
11. Backend code  ( Node.js )
    
    Issue Attestation / Claim
    ---
    ```
    import IPFS from 'mdip/lib/client/utils/ipfs';

    const ctrl = new IPFS({ url: 'http://18.117.222.252:5001', wsPort: 4003 });
    const res = await ctrl.connect();
    console.log('[ipfs] res', res);
    const claimType = 'general';
    const claimData = { 'general': {name: 'Thomas Jefferson', age: 62 }};
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
        const enc = await ctrl.encryptContent([{content: issuedClaim, publicKey: requestorPublicKey}]);
        const CID = await ctrl.sendJSONToIPFS(enc);
        res.send(CID);
    } catch (e) {
        console.log('e :>> ', e);
    }
    ```
    Verify Claim ( Verify the Verifiable Presentation )
    ---
    ```
    const { challenge, token, requestorDID, userVP, clientId } = req.body;
    const getPlatformXUser = await Auth.findOne({ requestorDID });

    // if VP is sent, means user is registered and trying to login
    if (userVP) {
        // verify VP and login Authorization
        if (verifyVP(userVP)) {
            authorizeLogin(clientId);
            res.send('');
        }
    }
    ```
    authorizeLogin Method
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

13. Backend : WebSocket Handling 
    ```
    const wsc = require('../wsc.js')

    wss.on('connection', (ws, req) => {
    var userID = req.url.substr(1);
    // maintain an in-memory websocket for later use
    wsc[userID] = ws;
    let randChallenge = Math.random();
    // save challenge to db.
    let newAuth = new Auth({ 'challenge': randChallenge });
    newAuth.save();
    //send new session information for QR Code Generation
    ws.send(JSON.stringify({
    "endpoint": '/user/handle',
    "token": "token",
    "challenge": randChallenge,
    "domain": process.env.DOMAIN || 'http://localhost:3000',
    }));
    });

    ```
    What is wsc.js?
    ---
    wsc.js is a helpfer file for storing the Active Web Socket connections as it is in an in-memory object. This cannot persist, meaning; once the node.js server is restarted there are no active web sockets.
    ```
    module.exports = {};
    ```
