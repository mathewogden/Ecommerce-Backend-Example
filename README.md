# Ecommerce-Backend-Example

## Table of Contents

- [Description](#Description)
<!-- - [Deployed Link](#Deployed-Link) -->
- [Technologies](#Technologies)
- [Code](#Code)
- [Gif](#Gif)
- [Author](#Author)
- [Credits](#Credits)
- [License](#License)

## Description

Here is the backend to my ecommerce example platform!

<!-- ## Deployed-Link -->
<!-- add deployed link in the paranthesis -->
<!-- - [Click Here]() -->

## Technologies

<!-- left examples here put edit to put your technologies -->

- [JavaScript](https://www.w3schools.com/js/)
- [ExpressJs](https://expressjs.com)
- [MySQL2](https://www.npmjs.com/package/mysql2)
- [Sequelize](https://sequelize.org/master/)
- [JSONWebToken](https://www.npmjs.com/package/jsonwebtoken)

## Gif

<!-- path to gif in paranthesis  -->

![Gif](https://media.giphy.com/media/vdpQgerqFaAAQM9hwB/giphy.gif)

## Code

### Checkout Route

- Let's look at the orders/checkout route (when our user finally submits an order). The req.body is received as an array of objects. I'll note that "productsOrdered" also contains an array of the products that were purchased.

```
  const {
    productsOrdered,
    totalPrice,
    buyerFirstName,
    buyerLastName,
    buyerEmail,
    buyerPhoneNumber,
    streetAddress,
    city,
    state,
    zipcode,
    nameOnCard,
    cardNumber,
    cardExpirationDate,
    cardCvv } = req.body;
```

- Then we do some validations. First we'll check if the "productsOrdered" being sent is indeed an array.

```
  if (!productsOrdered || !Array.isArray(req.body.productsOrdered)) {
    res.status(400).send({ message: 'No Products Ordered' });
    return;
  }
```

- Then, we'll check if the products being ordered exist in our Product table from our database.

```
  let productIds = productsOrdered.map(a => a.productId);

  Product.findAll({
    where: {
      id: { [Op.in]: productIds }
    }
  }).then(function (result) {
    const productDbIds = result;

    let productIdCheck = productDbIds.map(b => b.id);

    console.log('Amount of items verified in DB: ' + productIdCheck.length);
    console.log('Amount of items ordered: ' + productIds.length);

    if (productIds.length !== productIdCheck.length) {
      res.status(400).send({ message: 'Something is not right' });
      return;
    }

  });
```

- After our validations, we create our Order. When creating a new Order, we create a new ProductOrdered instance, and we also update the quantity on the Product table accordingly. (Be sure to checkout my models folder to see how I setup my database).

```
  Order.create({
    totalPrice: totalPrice,
    buyerFirstName: buyerFirstName,
    buyerLastName: buyerLastName,
    buyerEmail: buyerEmail,
    buyerPhoneNumber: buyerPhoneNumber,
    streetAddress: streetAddress,
    city: city,
    state: state,
    zipcode: zipcode,
    nameOnCard: nameOnCard,
    cardNumber: cardNumber,
    cardExpirationDate: cardExpirationDate,
    cardCvv: cardCvv,
    UserId: user.id
  }).then(newOrder => {

    productsOrdered.forEach(function (product) {

      ProductOrdered.create({
        ProductId: product.productId,
        productName: product.productName,
        quantity: product.quantity,
        price: product.price,
        OrderId: newOrder.id,
        UserId: user.id
      }).then(newProductOrdered => {
      }).catch(() => {
        res.status(400).send();
      });
      // Update Product quantity
      Product.update({
        quantity: Sequelize.literal(`quantity - ${product.quantity}`)
      }, {
        where: { id: product.productId }
      });
    });
    // newOrders.push(newOrder);
    res.json(newOrder);
  }).catch(() => {
    res.status(400).send();
  });
```

### Authentication

- Let's briefly look at the authentication method when using JWT Token Auth. Of course we have our auth.js you can see below.

```
const jwt = require('jsonwebtoken');
const { User } = require('../models');

const secretKey = 'ThisIsTheSecretKey';

module.exports = {
    createJWT: (user) => {
        const token = jwt.sign({
            username: user.username,
            id: user.id
        },
            secretKey,
            {
                expiresIn: '1y'
            });

        return token;
    },
    verifyUser: (token) => {
        try {
            const decodedPayload = jwt.verify(token, secretKey);
            return User.findByPk(decodedPayload.id);
        } catch (err) {
            return null;
        }


    }
};
```

- Now, I setup the JWT Token Authentication as middleware in the main app.js. I'm grabbing the token from the request and validating it before we hit any routes.

```
app.use(async (req, res, next) => {
  // Get the token from the request
  const header = req.headers.authorization;

  if (!header) {
    return next();
  }

  const token = header.split(" ")[1];

  // Validate token / get the user
  const user = await auth.verifyUser(token);
  req.user = user;
  next();
});
```

- Finally, for every route that I would like required authentication for, I'll validate the token like so:

```
  const user = req.user;

  if (!user) {
    res.status(403).send();
    return;
  }
```

## Author

Mathew Ogden

<!-- add linnks to your social media -->

- [GitHub](https://github.com/mathewogden)
- [linkedIn](https://www.linkedin.com/in/mathew-ogden-b85688220/)
- [Instagram](https://www.instagram.com/matogden_/?hl=en)

## License]

[![MIT License](https://img.shields.io/badge/License-MIT-blue.svg)](https://www.mit.edu/~amini/LICENSE.md)
