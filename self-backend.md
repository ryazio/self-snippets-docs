
 Backend code  ( Node.js )
=    
1. install SDK in your node.js backend
    ```bash
    npm i git+https://git@gitlab.com:self-id/self-platform-sdk.git#attestor-sdk
    ```
2. add ENV VARS, change values accordingly.
    ```bash
    SELF_MARKETPLACE_EMAIL=yourEmail@attestorSite.io
    SELF_MARKETPLACE_PASSWORD="Passwd123!@#"
    ATTESTOR_NAME='AttestorName'
    ATTESTOR_DID_IPFS=/ipfs/QmSesN2rtufDCWodxfWELDq5CTWJ2VKHQ5ELBYJT4VpvQH
    ATTESTOR_DID=did:mdip:omni-xqmn-fyzz-qufd-r5l
    PUBLIC_KEY=moeF4PQx829nAN9f3kDG88XFwSaWYDL3JX
    PRIVATE_KEY=cUsZuFZ5vY18DBNtstMAa4HvucrAhpE3kTPSdSWeHBGxAgvxHkUi
    PUBLIC_KEY_HEX=031eb429c1d63ef1505e2e99ee5ede671271d7f369242ac603a3af72852eb0a135
    PRIVATE_KEY_HEX=d989dba8574f3a50613f0c826b6dfa9d85f4eefdd16c8a111a43e7cb38546305
    ```
2. needed imports
    ```js
    const { verifyVPGetData } = require('self/lib/attestations/flow');
    const { offloadGranularClaims } = require('self/lib/attestations/offload');
    const {
    fetchAndCheckRequiredForLoginVPData,
    } = require('self/lib/attestations/cid');
    const { IPNSHelper } = require('self/lib/ipns/helper');
    ```
3. Initialize IPFS Helper
    ```js
    const helper = new IPNSHelper();
    helper.initializeIPNS(CONFIG.ipfsNode, CONFIG.ipfsPort);
    ```
4. Backend code  ( Node.js )
    
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
    ```
    module.exports = {};
    ```
