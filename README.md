# api documentation for  [iron (v4.0.4)](https://github.com/hueniverse/iron#readme)  [![npm package](https://img.shields.io/npm/v/npmdoc-iron.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-iron) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-iron.svg)](https://travis-ci.org/npmdoc/node-npmdoc-iron)
#### Encapsulated tokens (encrypted and mac'ed objects)

[![NPM](https://nodei.co/npm/iron.png?downloads=true)](https://www.npmjs.com/package/iron)

[![apidoc](https://npmdoc.github.io/node-npmdoc-iron/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-iron_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-iron/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-iron/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-iron/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Eran Hammer",
        "email": "eran@hammer.io",
        "url": "http://hueniverse.com"
    },
    "bugs": {
        "url": "https://github.com/hueniverse/iron/issues"
    },
    "dependencies": {
        "boom": "4.x.x",
        "cryptiles": "3.x.x",
        "hoek": "4.x.x"
    },
    "description": "Encapsulated tokens (encrypted and mac'ed objects)",
    "devDependencies": {
        "code": "3.x.x",
        "lab": "11.x.x"
    },
    "directories": {},
    "dist": {
        "shasum": "c1f8cc4c91454194ab8920d9247ba882e528061a",
        "tarball": "https://registry.npmjs.org/iron/-/iron-4.0.4.tgz"
    },
    "engines": {
        "node": ">=4.0.0"
    },
    "gitHead": "dc84162054fed7d093cf0344d540798464f095c1",
    "homepage": "https://github.com/hueniverse/iron#readme",
    "keywords": [
        "authentication",
        "encryption",
        "data integrity"
    ],
    "license": "BSD-3-Clause",
    "main": "lib/index.js",
    "maintainers": [
        {
            "name": "hueniverse",
            "email": "eran@hueniverse.com"
        }
    ],
    "name": "iron",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git://github.com/hueniverse/iron.git"
    },
    "scripts": {
        "test": "lab -a code -t 100 -L",
        "test-cov-html": "lab -a code -r html -o coverage.html"
    },
    "version": "4.0.4"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module iron](#apidoc.module.iron)
1.  [function <span class="apidocSignatureSpan">iron.</span>decrypt (password, options, data, callback)](#apidoc.element.iron.decrypt)
1.  [function <span class="apidocSignatureSpan">iron.</span>encrypt (password, options, data, callback)](#apidoc.element.iron.encrypt)
1.  [function <span class="apidocSignatureSpan">iron.</span>generateKey (password, options, callback)](#apidoc.element.iron.generateKey)
1.  [function <span class="apidocSignatureSpan">iron.</span>hmacWithPassword (password, options, data, callback)](#apidoc.element.iron.hmacWithPassword)
1.  [function <span class="apidocSignatureSpan">iron.</span>seal (object, password, options, callback)](#apidoc.element.iron.seal)
1.  [function <span class="apidocSignatureSpan">iron.</span>unseal (sealed, password, options, callback)](#apidoc.element.iron.unseal)
1.  object <span class="apidocSignatureSpan">iron.</span>algorithms
1.  object <span class="apidocSignatureSpan">iron.</span>defaults
1.  string <span class="apidocSignatureSpan">iron.</span>macFormatVersion
1.  string <span class="apidocSignatureSpan">iron.</span>macPrefix



# <a name="apidoc.module.iron"></a>[module iron](#apidoc.module.iron)

#### <a name="apidoc.element.iron.decrypt"></a>[function <span class="apidocSignatureSpan">iron.</span>decrypt (password, options, data, callback)](#apidoc.element.iron.decrypt)
- description and source-code
```javascript
decrypt = function (password, options, data, callback) {

    exports.generateKey(password, options, (err, key) => {

        if (err) {
            return callback(err);
        }

        const decipher = Crypto.createDecipheriv(options.algorithm, key.key, key.iv);
        let dec = decipher.update(data, null, 'utf8');
        dec = dec + decipher.final('utf8');

        callback(null, dec);
    });
}
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.iron.encrypt"></a>[function <span class="apidocSignatureSpan">iron.</span>encrypt (password, options, data, callback)](#apidoc.element.iron.encrypt)
- description and source-code
```javascript
encrypt = function (password, options, data, callback) {

    exports.generateKey(password, options, (err, key) => {

        if (err) {
            return callback(err);
        }

        const cipher = Crypto.createCipheriv(options.algorithm, key.key, key.iv);
        const enc = Buffer.concat([cipher.update(data, 'utf8'), cipher.final()]);

        callback(null, enc, key);
    });
}
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.iron.generateKey"></a>[function <span class="apidocSignatureSpan">iron.</span>generateKey (password, options, callback)](#apidoc.element.iron.generateKey)
- description and source-code
```javascript
generateKey = function (password, options, callback) {

    const callbackTick = Hoek.nextTick(callback);

    if (!password) {
        return callbackTick(Boom.internal('Empty password'));
    }

    if (!options ||
        typeof options !== 'object') {

        return callbackTick(Boom.internal('Bad options'));
    }

    const algorithm = exports.algorithms[options.algorithm];
    if (!algorithm) {
        return callbackTick(Boom.internal('Unknown algorithm: ' + options.algorithm));
    }

    const generate = () => {

        if (Buffer.isBuffer(password)) {
            if (password.length < algorithm.keyBits / 8) {
                return callbackTick(Boom.internal('Key buffer (password) too small'));
            }

            const result = {
                key: password,
                salt: ''
            };

            return generateIv(result);
        }

        if (password.length < options.minPasswordlength) {
            return callbackTick(Boom.internal('Password string too short (min ' + options.minPasswordlength + ' characters required
)'));
        }

        if (options.salt) {
            return generateKey(options.salt);
        }

        if (options.saltBits) {
            return generateSalt();
        }

        return callbackTick(Boom.internal('Missing salt or saltBits options'));
    };

    const generateSalt = () => {

        const randomSalt = Cryptiles.randomBits(options.saltBits);
        if (randomSalt instanceof Error) {
            return callbackTick(Boom.wrap(randomSalt));
        }

        const salt = randomSalt.toString('hex');
        return generateKey(salt);
    };

    const generateKey = (salt) => {

        Crypto.pbkdf2(password, salt, options.iterations, algorithm.keyBits / 8, 'sha1', (err, derivedKey) => {

            if (err) {
                return callback(Boom.wrap(err));
            }

            const result = {
                key: derivedKey,
                salt
            };

            return generateIv(result);
        });
    };

    const generateIv = (result) => {

        if (algorithm.ivBits &&
            !options.iv) {

            const randomIv = Cryptiles.randomBits(algorithm.ivBits);
            if (randomIv instanceof Error) {
                return callbackTick(Boom.wrap(randomIv));
            }

            result.iv = randomIv;
            return callbackTick(null, result);
        }

        if (options.iv) {
            result.iv = options.iv;
        }

        return callbackTick(null, result);
    };

    generate();
}
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.iron.hmacWithPassword"></a>[function <span class="apidocSignatureSpan">iron.</span>hmacWithPassword (password, options, data, callback)](#apidoc.element.iron.hmacWithPassword)
- description and source-code
```javascript
hmacWithPassword = function (password, options, data, callback) {

    exports.generateKey(password, options, (err, key) => {

        if (err) {
            return callback(err);
        }

        const hmac = Crypto.createHmac(options.algorithm, key.key).update(data);
        const digest = hmac.digest('base64').replace(/\+/g, '-').replace(/\//g, '_').replace(/\=/g, '');

        const result = {
            digest,
            salt: key.salt
        };

        return callback(null, result);
    });
}
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.iron.seal"></a>[function <span class="apidocSignatureSpan">iron.</span>seal (object, password, options, callback)](#apidoc.element.iron.seal)
- description and source-code
```javascript
seal = function (object, password, options, callback) {

    const now = Date.now() + (options.localtimeOffsetMsec || 0);                 // Measure now before any other processing

    const callbackTick = Hoek.nextTick(callback);

    // Serialize object

    const objectString = internals.stringify(object);
    if (objectString instanceof Error) {
        return callbackTick(objectString);
    }

    // Obtain password

    let passwordId = '';
    password = internals.normalizePassword(password);
    if (password.id) {
        if (!/^\w+$/.test(password.id)) {
            return callbackTick(Boom.internal('Invalid password id'));
        }

        passwordId = password.id;
    }

    // Encrypt object string

    exports.encrypt(password.encryption, options.encryption, objectString, (err, encrypted, key) => {

        if (err) {
            return callback(err);
        }

        // Base64url the encrypted value

        const encryptedB64 = Hoek.base64urlEncode(encrypted);
        const iv = Hoek.base64urlEncode(key.iv);
        const expiration = (options.ttl ? now + options.ttl : '');
        const macBaseString = exports.macPrefix + '*' + passwordId + '*' + key.salt + '*' + iv + '*' + encryptedB64 + '*' + expiration
;

        // Mac the combined values

        exports.hmacWithPassword(password.integrity, options.integrity, macBaseString, (err, mac) => {

            if (err) {
                return callback(err);
            }

            // Put it all together

            // prefix*[password-id]*encryption-salt*encryption-iv*encrypted*[expiration]*hmac-salt*hmac
            // Allowed URI query name/value characters: *-. \d \w

            const sealed = macBaseString + '*' + mac.salt + '*' + mac.digest;
            return callback(null, sealed);
        });
    });
}
```
- example usage
```shell
...
    d: {
        e: 'f'
    }
};

var password = 'some_not_random_password_that_is_at_least_32_characters';

Iron.seal(obj, password, Iron.defaults, function (err, sealed) {

    console.log(sealed);
});
'''

The result 'sealed' object is a string which can be sent via cookies, URI query parameter, or an HTTP header attribute.
To unseal the string:
...
```

#### <a name="apidoc.element.iron.unseal"></a>[function <span class="apidocSignatureSpan">iron.</span>unseal (sealed, password, options, callback)](#apidoc.element.iron.unseal)
- description and source-code
```javascript
unseal = function (sealed, password, options, callback) {

    const now = Date.now() + (options.localtimeOffsetMsec || 0);                 // Measure now before any other processing

    const callbackTick = Hoek.nextTick(callback);

    // Break string into components

    const parts = sealed.split('*');
    if (parts.length !== 8) {
        return callbackTick(Boom.internal('Incorrect number of sealed components'));
    }

    const macPrefix = parts[0];
    const passwordId = parts[1];
    const encryptionSalt = parts[2];
    const encryptionIv = parts[3];
    const encryptedB64 = parts[4];
    const expiration = parts[5];
    const hmacSalt = parts[6];
    const hmac = parts[7];
    const macBaseString = macPrefix + '*' + passwordId + '*' + encryptionSalt + '*' + encryptionIv + '*' + encryptedB64 + '*' +
expiration;

    // Check prefix

    if (macPrefix !== exports.macPrefix) {
        return callbackTick(Boom.internal('Wrong mac prefix'));
    }

    // Check expiration

    if (expiration) {
        if (!expiration.match(/^\d+$/)) {
            return callbackTick(Boom.internal('Invalid expiration'));
        }

        const exp = parseInt(expiration, 10);
        if (exp <= (now - (options.timestampSkewSec * 1000))) {
            return callbackTick(Boom.internal('Expired seal'));
        }
    }

    // Obtain password

    if (password instanceof Object &&
        !(Buffer.isBuffer(password))) {

        password = password[passwordId || 'default'];
        if (!password) {
            return callbackTick(Boom.internal('Cannot find password: ' + passwordId));
        }
    }
    password = internals.normalizePassword(password);

    // Check hmac

    const macOptions = Hoek.clone(options.integrity);
    macOptions.salt = hmacSalt;
    exports.hmacWithPassword(password.integrity, macOptions, macBaseString, (err, mac) => {

        if (err) {
            return callback(err);
        }

        if (!Cryptiles.fixedTimeComparison(mac.digest, hmac)) {
            return callback(Boom.internal('Bad hmac value'));
        }

        // Decrypt

        const encrypted = Hoek.base64urlDecode(encryptedB64, 'buffer');
        if (encrypted instanceof Error) {
            return callback(Boom.wrap(encrypted));
        }

        const decryptOptions = Hoek.clone(options.encryption);
        decryptOptions.salt = encryptionSalt;

        decryptOptions.iv = Hoek.base64urlDecode(encryptionIv, 'buffer');
        if (decryptOptions.iv instanceof Error) {
            return callback(Boom.wrap(decryptOptions.iv));
        }

        exports.decrypt(password.encryption, decryptOptions, encrypted, (ignoreErr, decrypted) => {         // Cannot fail since
 all errors covered by hmacWithPassword()

            // Parse JSON

            let object = null;
            try {
                object = JSON.parse(decrypted);
            }
            catch (err) {
                return callback(Boom.internal('Failed parsing sealed object JSON: ' + err.message));
            }

            return callback(null, object);
        });
    });
}
```
- example usage
```shell
...
});
'''

The result 'sealed' object is a string which can be sent via cookies, URI query parameter, or an HTTP header attribute.
To unseal the string:

'''javascript
Iron.unseal(sealed, password, Iron.defaults, function (err, unsealed) {

    // unsealed has the same content as obj
});
'''

### Options
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
