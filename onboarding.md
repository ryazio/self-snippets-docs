
SELF ID - Platform Onboarding Doc
===

1. Create an account on https://onboarding.yourself.id/
2. Input your Email and desired Password, submit.
3. On verification screen, input the verification code that you got in your email and submit.
4. Enter Details about you as an platform/attestor, Name, website. Logo/Photo can be skipped for now. 
5. Select a category that your platform falls into, click next.
6. Select "Required for Login" molecules by selecting relevant categories and attestors ( might not be applicable for now as there aren't any attestors )
7. Next step will Ask if you want to generate a new mnemonic or use an existing mnemonic, you can use a funded mnemonic that has already been given to you.
8. It will now register you on the Bitcoin Blockchain using the omni layer protocol. Wait until you get your Transaction ID on screen. Copy all the keys on the screen ( Public Key, Private Key, Public Key Hex, Private Key Hex and Transaction ID ), Click next.
9. After the transaction you will have to wait for about 15 minutes approximately until your transaction is confirmed on the chain and then finally you will have your DID address, which you should be able to see in settings.
10. We can start integrating the frontend code as shown in the onboarding process.
11. Frontend Widget Code 
    ```html
    <!-- GOES IN <HEAD> -->
    <script type="text/javascript" src="https://davidshimjs.github.io/qrcodejs/qrcode.min.js"></script>
    <script type="text/javascript" src="https://some-publically-hosted-SELF-ID-address.com/btn.js"></script>
    <link rel="stylesheet" href="https://onboarding.yourself.id/btn.css"
    ```
    ```html
    <script>
        // goes in <body>
        createBtn("wss://your-backend-endpoint-wss-enabled.com");
    </script>
    ```
12. install SDK in your node.js backend
    ```bash
    npm i git+https://git@gitlab.com:self-id/self-platform-sdk.git#attestor-sdk
    ```
13. needed imports
    ```js
    const { verifyVPGetData } = require('self/lib/attestations/flow');
    const { offloadGranularClaims } = require('self/lib/attestations/offload');
    const {
    fetchAndCheckRequiredForLoginVPData,
    } = require('self/lib/attestations/cid');
    const { IPNSHelper } = require('self/lib/ipns/helper');
    ```
14. Initialize IPFS Helper
    ```js
    const helper = new IPNSHelper();
    helper.initializeIPNS(CONFIG.ipfsNode, CONFIG.ipfsPort);
    ```
11. Backend code  ( Node.js )
    
    Issue Attestation / Claim
    ---
    ```js
    const claimData = {
        'name': 'Thomas Edison',
        'drivers_license': 'RC45SDADY'
    };
    const { clientId, publicKey, requestorDID } = req.body;
    const keys = Object.keys(claimData);
    if (keys.length) {
        
        /*  this method takes all the that attestor wants to send to a SELF user, and makes it cryptographically secure, encrypted and puts it on IPFS, and makes sure the SELF user gets the data. */

        const response = await offloadGranularClaims(
            { publicKey, requestorDID },
            claimData,
            helper, // ipfs helper
            true,
        ); // should be true
    }
    // to reflect the login on frontend with needed data
    authorizeLogin(clientId, atoms, claimData); 

    ```
    Decrypt and Verifying Claim Integrity (Verify the Verifiable Presentation)
    ---
    ```js
    const { userVP, clientId, requestorDID } = req.body;
    const response = await verifyVPGetData(userVP, helper);

    if (verifyClaim) authorizeLogin(clientID);
    ```
    Checking if required data is present
    --
    ```js
    const reqforLoginPass = success
        ? await fetchAndCheckRequiredForLoginVPData(response.data)
        : false;
    if (!reqforLoginPass) {
        res.send({
        success: false,
        data: null,
        error: 'Required for login molecules absent',
        });
        return;
    }else {
        authorizeLogin(clientId);
        console.log("Authorize the web UI and pass any token that's needed")
    }
    ```

    authorizeLogin Method
    --
    ```js
    const authorizeLogin = (clientId, atoms = {}, claimData = {}) => {
    // if client is available, generate a new AccessToken for the
    // client and send needed information to frontend via WebSocket
    if (wsc[clientId]) {
        wsc[clientId].send(
            JSON.stringify({
                success: true,
                token: 'af6s8df67sd6fa8fsasdgh78687a',
                sharedData: atoms,
                claimData,
            }),
        );
    } else {
        debug('ClientId not found in hash table', clientId);
    }
    };

    ```

13. Backend : WebSocket Handling 
    ```js
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
    ```js
    module.exports = {};
    ```
