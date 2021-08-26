1. Import these scripts in your `index.html` file.

```html
<script type="text/javascript" src="<https://davidshimjs.github.io/qrcodejs/qrcode.min.js>"></script>
<script type="text/javascript" src="btn.js"></script>
```

2. Import `btn.css` file in your `index.html` file.
```html
<link rel="stylesheet" href="btn.css">
```

3. Use the script like this to create `login with self` button in your App.

```html
<script>
    // The URL in function argument is of backend snippet Websocket
    createBtn("wss://demo-attestor-backend.byrana.xyz");

    // Another example:
    // createBtn("ws://localhost:3001");
</script>
```

4. Visit the `index.html` and click the button `login with self` and it will try to connect to the given WebSocket host and return the data which will further be rendered as QR code.