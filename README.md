### **1. Structure Globale du Projet**

```plaintext
usdt-bot-economic-rentable/
│
├── public/
│   ├── index.html
│   ├── styles.css
│   └── script.js
│
├── server/
│   ├── server.js
│   ├── models/
│   │   ├── User.js
│   │   ├── VIPPlan.js
│   │   └── Affiliation.js
│   ├── routes/
│   │   ├── auth.js
│   │   ├── vip.js
│   │   └── affiliate.js
│   └── config.js
│
├── .gitignore
├── package.json
├── Procfile
└── README.md
```

### **2. Contenu des Fichiers**

#### **`server/config.js`**

Ce fichier contient les configurations de base :

```js
// server/config.js
module.exports = {
    db: 'mongodb://localhost/usdt-bot',
    port: 3000,
    usdtAddress: 'TGtAxvAoZJktmZRaA2vXbKPb4kuWA1b33d',
    email: 'constanthubert39@gmail.com'
};
```

#### **`server/server.js`**

Le point d'entrée principal pour ton serveur Express :

```js
// server/server.js
const express = require('express');
const mongoose = require('mongoose');
const bodyParser = require('body-parser');
const authRoutes = require('./routes/auth');
const vipRoutes = require('./routes/vip');
const affiliateRoutes = require('./routes/affiliate');
const config = require('./config');

const app = express();

app.use(bodyParser.json());
app.use(express.static('public'));

// Connexion à la base de données MongoDB
mongoose.connect(config.db, { useNewUrlParser: true, useUnifiedTopology: true });

// Routes
app.use('/auth', authRoutes);
app.use('/vip', vipRoutes);
app.use('/affiliate', affiliateRoutes);

app.get('/', (req, res) => {
    res.sendFile(__dirname + '/../public/index.html');
});

app.listen(config.port, () => {
    console.log(`Server is running on port ${config.port}`);
});
```

#### **`server/models/User.js`**

Modèle pour les utilisateurs :

```js
// server/models/User.js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
    name: String,
    surname: String,
    username: { type: String, unique: true },
    password: String,
    kyc: Boolean,
    usdtBalance: Number,
    vipPlan: { type: mongoose.Schema.Types.ObjectId, ref: 'VIPPlan' }
});

module.exports = mongoose.model('User', userSchema);
```

#### **`server/models/VIPPlan.js`**

Modèle pour les plans VIP :

```js
// server/models/VIPPlan.js
const mongoose = require('mongoose');

const vipPlanSchema = new mongoose.Schema({
    name: String,
    amountRequired: Number,
    dailyIncome: Number,
    status: { type: String, enum: ['available', 'coming soon'] }
});

module.exports = mongoose.model('VIPPlan', vipPlanSchema);
```

#### **`server/models/Affiliation.js`**

Modèle pour les affiliations :

```js
// server/models/Affiliation.js
const mongoose = require('mongoose');

const affiliationSchema = new mongoose.Schema({
    level: String,
    percentage: Number
});

module.exports = mongoose.model('Affiliation', affiliationSchema);
```

#### **`server/routes/auth.js`**

Routes pour l'authentification :

```js
// server/routes/auth.js
const express = require('express');
const User = require('../models/User');
const router = express.Router();

// Route pour l'inscription
router.post('/signup', async (req, res) => {
    try {
        const user = new User(req.body);
        await user.save();
        res.status(201).send(user);
    } catch (error) {
        res.status(400).send(error);
    }
});

// Route pour la connexion
router.post('/login', async (req, res) => {
    try {
        const user = await User.findOne({ username: req.body.username, password: req.body.password });
        if (!user) {
            return res.status(401).send('Invalid credentials');
        }
        res.status(200).send(user);
    } catch (error) {
        res.status(500).send(error);
    }
});

module.exports = router;
```

#### **`server/routes/vip.js`**

Routes pour gérer les plans VIP :

```js
// server/routes/vip.js
const express = require('express');
const VIPPlan = require('../models/VIPPlan');
const User = require('../models/User');
const router = express.Router();

// Route pour obtenir les plans VIP
router.get('/', async (req, res) => {
    try {
        const plans = await VIPPlan.find();
        res.status(200).send(plans);
    } catch (error) {
        res.status(500).send(error);
    }
});

// Route pour activer un plan VIP
router.post('/activate', async (req, res) => {
    try {
        const user = await User.findById(req.body.userId);
        const plan = await VIPPlan.findById(req.body.planId);
        
        if (user.usdtBalance < plan.amountRequired) {
            return res.status(400).send('Insufficient balance');
        }

        user.usdtBalance -= plan.amountRequired;
        user.vipPlan = plan._id;
        await user.save();
        
        res.status(200).send(user);
    } catch (error) {
        res.status(500).send(error);
    }
});

module.exports = router;
```

#### **`server/routes/affiliate.js`**

Routes pour gérer les affiliations :

```js
// server/routes/affiliate.js
const express = require('express');
const Affiliation = require('../models/Affiliation');
const router = express.Router();

// Route pour obtenir les niveaux d'affiliation
router.get('/', async (req, res) => {
    try {
        const affiliations = await Affiliation.find();
        res.status(200).send(affiliations);
    } catch (error) {
        res.status(500).send(error);
    }
});

module.exports = router;
```

### **Frontend**

#### **`public/index.html`**

Page d'accueil avec plans VIP :

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>USDT Bot Economic Rentable</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <h1>Bienvenue sur USDT Bot Economic Rentable</h1>
    <div id="vip-plans">
        <h2>Plans VIP</h2>
        <div id="vip-list"></div>
    </div>
    <script src="script.js"></script>
</body>
</html>
```

#### **`public/styles.css`**

Styles de base pour le site :

```css
/* styles.css */
body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
    background-color: #f4f4f4;
}

h1 {
    text-align: center;
    margin-top: 50px;
}

#vip-plans {
    margin: 20px;
}

.vip-plan {
    background-color: #fff;
    border: 1px solid #ddd;
    padding: 15px;
    margin-bottom: 10px;
}
```

#### **`public/script.js`**

JavaScript pour charger les plans VIP :

```js
// script.js
document.addEventListener('DOMContentLoaded', async () => {
    const response = await fetch('/vip');
    const plans = await response.json();
    
    const vipList = document.getElementById('vip-list');
    plans.forEach(plan => {
        const div = document.createElement('div');
        div.className = 'vip-plan';
        div.innerHTML = `
            <h3>${plan.name}</h3>
            <p>Montant requis: ${plan.amountRequired} USDT</p>
            <p>Revenu quotidien: ${plan.dailyIncome} USDT</p>
            <p>Status: ${plan.status}</p>
        `;
        vipList.appendChild(div);
    });
});
```

### **Fichiers Additionnels**

**`.gitignore`**

```plaintext
node_modules/
.env
```

**`package.json`**

```json
{
  "name": "usdt-bot-economic-rentable",
  "version": "1.0.0",
  "description": "Site pour investir en USDT avec des fonctionnalités d'affiliation.",
  "main": "server/server.js",
  "scripts": {
    "start": "node server/server.js"
  },
  "dependencies": {
    "express": "^4.17.1",
    "mongoose": "^6.0.12",
    "body-parser": "^1.19.0"
  }
}
```

**`Procfile`**

```txt
web: node server
