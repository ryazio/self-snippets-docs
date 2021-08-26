1. Install WS Package using the command below: 
```
npm install ws
```

2. Create a new file `ws-provider.js` and put the following content in it:
```javascript
const WebSocket = require('ws');

const dataProvider = (httpServer) => {
    const wss = new WebSocket.Server({ port: 3001 });
    console.log('wss:', wss);
    const wsc_pool = {};
    wss.on('connection', (ws, req) => {
        const userID = req.url.substr(1);
        wsc_pool[userID] = ws;
        const randChallenge = Math.random();
        // send immediate feedback to the incoming connection
        ws.send(JSON.stringify({
            "endpoint": '/user/handle',
            "token": "token1",
            "challenge": randChallenge,
            "domain": process.env.DOMAIN || 'http://localhost:3000',
        }));
    });
}
```

3. In the app.js or wherever you have express() app initialized, import and call above function:
```javascript
// import function
const dataProvider = require('../bin/ws-provider');
// existing express app instance
const app = express();
// calling the function
dataProvider(app);
```

4. In the routes file, add a new route:
```javascript
router.post('/user/handle', async (req, res) => {
	const { challenge, token, requestorDID, userVP, clientId } = req.body;
	// Check here auth and approve if the request is authenticated.

	res.send('Logged in');
}
```
