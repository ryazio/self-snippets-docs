1. Create a new file ws-provider.js and put the following content in it:

```javascript
const WebSocket = require('ws');

const dataProvider = (httpServer, authService) => {
    const wss = new WebSocket.Server({ server: httpServer });
    const wsc_pool = {};
    wss.on('connection', (ws, req) => {
        const userID = req.url.substr(1);
        wsc_pool[userID] = ws;
        const randChallenge = Math.random();
        authService(randChallenge);
        // send immediate feedback to the incoming connection
        ws.send(JSON.stringify({
            "endpoint": '/user/handle',
            "token": "token",
            "challenge": randChallenge,
            "domain": process.env.DOMAIN || '<http://localhost:3000>',
        }));
    });
}

module.exports = dataProvider;
```
2. The following is a sample code of how it can be used. It may be different for you, depending upon the way you implemented auth etc.

```javascript
const dataProvider = require('./ws-provider');
// the auth model here
const Auth = require('../models/auth');
// HTTP server the node app is listening on
const httpServer = http.createServer(app);
// Change the auth here according to your own implementation
const authService = (challenge) => {
    // save challenge to db
    const newAuth = new Auth({ challenge });
    newAuth.save();
}
// Initialize the websocket stuff
dataProvider(httpServer, authService);
```

3. Add the following to your app's routes:

```javascript
// Some variables needed for the route (configure as per your use case)
const mdipURL = process.env.MDIP_URL || '<http://3.18.200.195:7445>';
const blockchain = "mongodb";
const attestorName = 'Platform X';
const PDB_publicKey = '03c0a6e783322b828aa12c0a227755e4bb57958355e18cf1941bf0dfa0e78faf49';
const PDB_privateKey = 'b19383d3e911f8c509178ed6336ea93d2b25d62b7de50221e5f201b145604b27';
const PDB_attestorDID = 'did:mdip:mongodb-605306337399a2c914b3fc35';

router.post('/user/handle', async (req, res) => {
  const { challenge, token, requestorDID, userVP, clientId } = req.body;
  const getPlatformXUser = await Auth.findOne({ requestorDID });
  // Verify VP and send auth details to user
  if (userVP) {
    if (verifyVP(userVP)) {
      authorizeLogin(clientId);
      res.send('Logged in');
    }
  } else if (getPlatformXUser) {
    res.send("You're already signed up! Please send a VP.");
  } else {
    const claimType = 'isPlatformXUser';
    const claimData = { isPlatformXUser: true }; // obtained in a prev step
    try {
      const issuedClaim = await issueNewClaim({
        mdipURL,
        blockchain,
        requestorDID,
        claimType,
        claimData,
        attestorName,
        attestorPublicKey: PDB_publicKey,
        attestorPrivateKey: PDB_privateKey,
        attestorDID: PDB_attestorDID
      });
      // send this data as response
      res.send(issuedClaim);
    } catch (err) {
      console.log('error :>> ', err);
    }
  }
});
```

Run the server, and the route endpoint should be available to be hit from the frontend/app.