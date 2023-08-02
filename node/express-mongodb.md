## Guide for project with Express and MongoDB
- [Express](https://expressjs.com/) as node.js framework
- [Mongoose](https://mongoosejs.com/) as Database
### Create app
```sh
npm init -y
```

Add Express js
```sh
npm install express
```
Create index.js
```javascript
const express = require('express')
const app = express()
const port = process.env.PORT || 3000

app.get('/', (req, res) => {
  res.send('Hello World!')
})

app.listen(port, () => {
  console.log(`Example app listening on port ${port}`)
})
```
Add src directory
```tree
src
├── 
    ├── controllers
        ├── user.control.js
    ├── database
        ├── mongodb.js
    ├── middlewares
        ├── authorization.js
    ├── models
        ├── user.model.js
    └── routes
        ├── user.routes.js
    └── services
        ├── user.service.js
```
Add required packages
```sh
npm i body-parser cors mongoose nodemon jsonwebtoken
```

Add User Controller
```javascript
/** user.control.js */
const  UserService  = require("../services/user.service")
const service = new UserService()
module.exports = class UserController {
    constructor() {}
    static async AddUser(req, res) {
        try {
            const result = await service.AddUser(req.body)
            return res.json(result)
        } catch (error) {
            return res.json({ error: true, data: error })
        }
    }

    static async GetUsers(req, res) {
        try {
            const result = await service.GetUsers()
            return res.json(result)
        } catch (error) {
            return res.json({ error: true, data: error })
        }
    }
}
```
Add Database with mongodb config
```javascript
/** mongodb.js */
const mongoose = require('mongoose');
const atlasUrl = 'atlas url here'
const localUrl = 'mongodb://127.0.0.1/testdb'

const useAtlas = false
let url = useAtlas ? atlasUrl : localUrl
module.exports = class MongoDB {
    constructor() { }
    connect() {
        mongoose.connect(url, {
        }).then(() => {
            console.log('DB Connected! ' + url)
        }).catch(err => {
            console.log(err);
            console.log('CONNECTION ERROR!');
        });
    }
}
```

Add authorization middleware
```javascript
/** authorization.js */
//axios.defaults.headers.common['Authorization'] = acc_token

const jwt = require("jsonwebtoken");
const tokenkey = "tokensercret123456";
class Authorization {
    constructor() {}
    /*
     *parameter data is an object that hold any data  
     * you want to embed in your token
     */
    static getAccessToken(data) {
        var acc_token = jwt.sign({
            data
        }, tokenkey, {
            expiresIn: "12h"
        });
        return acc_token;
    }

    static routeEnter(req, res, next) {
        console.log(req)
        next()
    }

    /*
     * to be authorized create an api ang get an access token
     * add the access token to the client header 
     * axios.defaults.headers.common['Authorization'] = acc_token => sample using axios
     * getAccessToken is the function to generate token.
     */
    static authorized(req, res, next) {
        const token = req.headers.authorization;
        if (token) {
            jwt.verify(token, tokenkey, (err, user) => {
                if (err) {
                    return res.json({
                        error: true,
                        message: "You are forbidden."
                    });
                }
                req.user = user;
                next();
            });
        } else {
            return res.json({
                error: true,
                message: "You are not authenticated."
            });
        }
    };
}


module.exports = Authorization;
```

Add Sample model
```javascript
/** user.model.js */
const mongoose = require('mongoose');

const USERSSchema = new mongoose.Schema({
    fname: {
        type: String,
        required: false
    },
    lname: {
        type: String,
        required: null
    }
});

const Users = mongoose.model('Users', USERSSchema);

module.exports = Users;
```

Add user routes
```javascript
/** user.routes.js */
const express = require("express");
const router = express.Router();

const Authorization = require('../middlewares/authorization.js');
const UserController = require('../controllers/user.control')

router.get("/users", /* Authorization.authorized, */ UserController.GetUsers)
router.post("/users", /* Authorization.authorized, */ UserController.AddUser)

module.exports = router;
```

Add user service
```javascript
/** user.service.js */
const Users = require('../models/user.model')

module.exports = class UserService {
    constructor() { }
    async AddUser(user) {
        try {
            const saved = await Users.create(user)
            return { error: false, data: saved }
        } catch (error) {
            return { error: true, data: error }
        }
    }

    async GetUsers() {
        try {
            const list = await Users.find({})
            return { error: false, data: list }
        } catch (error) {
            return { error: true, data: error }
        }
    }
}
```

Update index.js
```javascript
const express = require('express')
const cors = require('cors');
const bodyParser = require('body-parser');
const MongoDB = require('./src/database/mongodb')
const app = express()
const port = process.env.PORT || 3000

const db = new MongoDB()
db.connect()

app.use(cors());
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({
    extended: false
}));

app.get("/", (req, res)=>{
    res.json({
        sms: "Hello from express."
    })
})

const userRoutes = require('./src/routes/user.routes')
app.use('/api', userRoutes)

app.listen(port, () => {
  console.log(`Example app listening on port ${port}`)
})
```