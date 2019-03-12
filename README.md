# Self-Signed-With-Chrome
A tutorial to setup a HTTPS server trusted by Google Chrome running on localhost.

1. Create a config file named req.cnf, this is the file openssl will rely on, to generate your certificate:
    
        [req]
        distinguished_name = req_distinguished_name
        x509_extensions = v3_req
        prompt = no
        [req_distinguished_name]
        C = Country initials like US, CA, etc
        ST = State, province, etc
        L = Location
        O = Organization Name
        OU = Organizational Unit 
        CN = www.localhost.com, localhost, foo, bar, whatever
        [v3_req]
        keyUsage = critical, digitalSignature, keyAgreement
        extendedKeyUsage = serverAuth
        subjectAltName = @alt_names
        [alt_names]
        DNS.1 = www.localhost.com
        DNS.2 = localhost.com
        DNS.3 = localhost
        
Be willing to explore and add some fields if you need.

2. Open your terminal and enter:

        openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout cert.key -out cert.pem -config req.cnf -sha256
        
2048 is the key's size, you can set the size you want, but it is not recommanded to go under 1024 bits (March 2019). You can change the cipher suite for the one you prefer.

3. Now you have a self-signed certificate that we will use in our express node.js server. 

        let express = require('express');
        let app = express();
        let https = require('https');
        let fs = require('fs');
        let port = 4343;

        app.get('/', (req, res) => {
          res.send("index route");
        });

        let httpsOptions = {
          key: fs.readFileSync('{YOUR_PATH}/cert.key'),
          cert: fs.readFileSync('{YOUR_PATH}/cert.pem')
        };

        let server = https.createServer(httpsOptions, app)
        server.listen(port, () => {
          console.log('server running at ' + port);
        });
    
4. Start the node server.

5. Open Chrome and hit your server, you will see that your certificate isn't trust. It is normal because the certificate is signed by yourself and you are not a valid authority.

6. Hit "chrome://settings" in your browser and go in advanced settings.

7. Click on manage certificates and import your certificate in "Trusted Root Certification Authorities" tab.

8. Restart your browser, hit your server and you will see a green lock. :)

**** This is for development purpose only, do not use this certificate in production ****
