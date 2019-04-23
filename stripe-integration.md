# An integration of stripe with react.js and node.js

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)


First, the most obvious one. Create stripe account. visit https://stripe.com.
Here we are going to use react.js as front-end with node.js as back-end.

### prerequisites
- Node.js
### Server side
Create a folder server, to split fron-end and back-end clearly.
```sh 
$ cd server
$ npm init
```
Fill the required steps following the above command.
A package.json file will be created.Add the following packges to the file.
> "body-parser": "^1.18.3"
> "cookie-parser": "^1.4.4"
> "dotenv": "^6.2.0"
> "express": "^4.16.4"
> "express-session": "^1.15.6"
> "morgan": "^1.9.1"
> "stripe": "^6.25.1"

and 
> "main": "server.js"

line shoud be there in package.json file. Now run

```sh 
$ npm install
```
This will install all the required packages.
create server.js file
 ##### server.js
 
 ```js
 require("dotenv").config();
var express = require("express");
var bodyParser = require("body-parser");

var http = require("http");
SESSION_SECRET = "justfortesting";
CLIENT_ORIGIN = "http://localhost:3000";
const stripe = require("stripe")("sk_test_XXXXXXXXXXXXXXXXXX"); // Add stripe secret key here.

const port = 5000;
var app = express();
app.use(require("morgan")("combined"));
app.use(require("cookie-parser")());
app.use(require("body-parser").urlencoded({ extended: true }));
app.use(
  require("express-session")({
    secret: SESSION_SECRET,
    resave: true,
    saveUninitialized: true
  })
);

const httpServer = http.createServer(app).listen(5000, () => {
  console.log(`Running on :${port}`);
});


// create application/json parser
var jsonParser = bodyParser.json();



app.post("/api/payment", jsonParser, (req, response) => {
  let amount = Math.round(req.body.amount) * 100;
  stripe.customers
    .create({
      email: req.body.email,
      card: req.body.id
    })
    .then(customer => {
      return stripe.charges.create({
        amount,
        description: "Customer Charge",
        currency: "usd",
        customer: customer.id
      });
    })
    .then(charge => {
		// You may save the info to Database.
    })
    .catch(err => {
      console.log("Error:", err);
      response.status(500).send({ error: "Purchase Failed" });
    });
});
```
Now run server inside server folder.

```sh
$ npm start
```
thus concludes server side part.

### Client side (react.js)

Here we use the react-stripe-elements libary to add all the required stripe components.

create client folder in the project root and run npm init command again here. this will create package.json file for client side.Add the following to package.json file

```json
{
  "name": "react-shop",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@fortawesome/react-fontawesome": "^0.1.4",
    "axios": "^0.18.0",
    "bootstrap": "^4.3.1",
    "immutable": "^4.0.0-rc.12",
    "querystringify": "^2.1.0",
    "react": "^16.8.2",
    "react-bootstrap": "^1.0.0-beta.5",
    "react-dom": "^16.8.2",
    "react-redux": "^6.0.1",
    "react-router-dom": "^4.3.1",
    "react-router-redux": "^4.0.8",
    "react-scripts": "2.1.5",
    "react-stripe-elements": "^2.0.3",
    "redux": "^4.0.1",
    "redux-immutable": "^4.0.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "server": "node server.js"
  },
  "eslintConfig": {
    "extends": "react-app"
  },
  "browserslist": [
    ">0.2%",
    "not dead",
    "not ie <= 11",
    "not op_mini all"
  ],
  "devDependencies": {
  }
}

```
Now run npm install.
create an index.html file and add the following

```html
<script src="https://js.stripe.com/v3/"></script>
```
Then create the following js files 

```js
// index.js
import React from 'react';
import {render} from 'react-dom';
import {StripeProvider} from 'react-stripe-elements';

import MyStoreCheckout from './MyStoreCheckout';

const App = () => {
  return (
    <StripeProvider apiKey="pk_test_12345"> // use the stripe's public key here
      <MyStoreCheckout />
    </StripeProvider>
  );
};

render(<App />, document.getElementById('root'));
```
```js
// MyStoreCheckout.js
import React from 'react';
import {Elements} from 'react-stripe-elements';

import InjectedCheckoutForm from './CheckoutForm';

class MyStoreCheckout extends React.Component {
  render() {
    return (
      <Elements>
        <InjectedCheckoutForm />
      </Elements>
    );
  }
}

export default MyStoreCheckout;
```
```js
// CheckoutForm.js
import React from 'react';
import {
  CardNumberElement,
  CardExpiryElement,
  CardCVCElement
} from "react-stripe-elements";
import axios from "axios";
import { injectStripe } from "react-stripe-elements";


class CheckoutForm extends React.Component {
  handleSubmit = (ev) => {
    // We don't want to let default form submission happen here, which would refresh the page.
    ev.preventDefault();

    this.props.stripe.createToken({name: 'Jonny Bravo'}).then(({token}) => {
       axios
            .post(
            "http://localhost:5000/api/payment",
              {
                email: jonnybravo@gmail.com,
                id: token.id,
                amount:500
              }
            )
            .then(function(res) {
              if (res.data.status === "error") {
                console.log(res.data.msg);
              } 
            })
    });

  };

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <div className="col-md-6">
            <div className="form-group">
              <label className="">Card Number*</label>
              <CardNumberElement placeholder="Test card: 4242 4242 4242 4242" />
            </div>
          </div>
          <div className="col-md-3">
            <div className="form-group">
              <label className="">Vaild Thru*</label>
              <CardExpiryElement />
            </div>
          </div>
          <div className="col-md-3">
            <div className="form-group">
              <label className="">CVV/CVC *</label>
              <CardCVCElement />
            </div>
          </div>
        <button>Confirm order</button>
      </form>
    );
  }
}

export default injectStripe(CheckoutForm);
```

Now run 

```sh
$ npm start
```

 


