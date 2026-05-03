# JNR — Guide de lancement & déploiement

## 🚀 Lancement rapide

Ce site est un fichier HTML autonome (single-page application).  
**Aucune installation requise pour la démo locale.**

### Option 1 : Ouverture directe
Double-cliquez sur `index.html` → s'ouvre dans votre navigateur. C'est tout.

### Option 2 : Serveur local (recommandé)
```bash
# Avec Python (préinstallé sur macOS/Linux)
python3 -m http.server 8080
# → Ouvrez http://localhost:8080

# Avec Node.js
npx serve .
# → Ouvrez l'URL indiquée

# Avec VS Code : installez l'extension "Live Server" et cliquez sur "Go Live"
```

---

## 💳 Intégration Stripe (paiement réel)

### Étape 1 — Créer un compte Stripe
1. Rendez-vous sur https://stripe.com
2. Créez un compte gratuit
3. Allez dans **Développeurs → Clés API**
4. Copiez votre **clé publique** (commence par `pk_test_...`)

### Étape 2 — Installer Stripe.js
Ajoutez dans `<head>` de `index.html` :
```html
<script src="https://js.stripe.com/v3/"></script>
```

### Étape 3 — Remplacer le formulaire de carte
Remplacez la section `.stripe-box` dans le HTML par :
```html
<div id="stripe-card-element" style="background:var(--dark3);border:1px solid var(--border);border-radius:8px;padding:1rem;"></div>
<div id="stripe-errors" style="color:#ff4d6d;font-size:0.82rem;margin-top:0.5rem;"></div>
```

### Étape 4 — Initialiser Stripe dans le JS
```javascript
const stripe = Stripe('pk_test_VOTRE_CLE_PUBLIQUE_ICI');
const elements = stripe.elements();
const cardElement = elements.create('card', {
  style: {
    base: {
      color: '#e8e8e8',
      fontFamily: 'DM Sans, sans-serif',
      fontSize: '15px',
      '::placeholder': { color: '#555' }
    }
  }
});
cardElement.mount('#stripe-card-element');
```

### Étape 5 — Créer un PaymentIntent (backend requis)
```javascript
// Dans processPayment(), remplacez la simulation par :
async function processPayment() {
  const {paymentMethod, error} = await stripe.createPaymentMethod({
    type: 'card',
    card: cardElement,
  });
  if (error) {
    document.getElementById('stripe-errors').textContent = error.message;
    return;
  }
  // Envoyez paymentMethod.id à votre backend
  const response = await fetch('/create-payment-intent', {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify({
      payment_method_id: paymentMethod.id,
      amount: getTotalInCents(), // ex: 4999 pour 49.99€
      currency: 'eur'
    })
  });
  const result = await response.json();
  if (result.success) confirmOrder();
}
```

### Étape 6 — Backend Node.js minimal
```javascript
// server.js
const express = require('express');
const stripe = require('stripe')('sk_test_VOTRE_CLE_SECRETE');
const app = express();
app.use(express.json());
app.use(express.static('.'));

app.post('/create-payment-intent', async (req, res) => {
  const { payment_method_id, amount, currency } = req.body;
  try {
    const paymentIntent = await stripe.paymentIntents.create({
      amount, currency,
      payment_method: payment_method_id,
      confirm: true,
      return_url: 'http://localhost:3000/confirmation'
    });
    res.json({ success: true, clientSecret: paymentIntent.client_secret });
  } catch (error) {
    res.json({ success: false, error: error.message });
  }
});

app.listen(3000, () => console.log('Serveur sur http://localhost:3000'));
```

```bash
npm init -y
npm install express stripe
node server.js
```

### Cartes de test Stripe
| Numéro | Résultat |
|--------|----------|
| 4242 4242 4242 4242 | ✅ Succès |
| 4000 0025 0000 3155 | 🔐 Authentification 3DS |
| 4000 0000 0000 9995 | ❌ Refus |

---

## 🌐 Déploiement en production

### Option A — Vercel (gratuit, recommandé)
```bash
npm install -g vercel
vercel deploy
# Suivez les instructions → URL publique en 2 minutes
```

### Option B — Netlify (gratuit)
1. Allez sur https://netlify.com
2. Faites glisser le dossier `jnr-store/` dans l'interface
3. Votre site est en ligne immédiatement

### Option C — GitHub Pages (gratuit)
```bash
git init
git add .
git commit -m "JNR Store"
git remote add origin https://github.com/VOTRE_USER/jnr-store.git
git push -u origin main
# → Activez GitHub Pages dans Settings → Pages → main branch
```

### Option D — Hébergement traditionnel (OVH, Ionos...)
- Uploadez `index.html` via FTP dans le dossier `public_html/`
- C'est tout (pour la version sans backend)

---

## 🎁 Codes promo disponibles

| Code | Réduction |
|------|-----------|
| `JNR10` | 10% |
| `WELCOME20` | 20% |
| `FIRST15` | 15% |
| `VAPE5` | 5% |

Pour ajouter un code, modifiez l'objet `PROMO_CODES` dans le JS.

---

## ⚖️ Obligations légales (France)

- ✅ Vérification d'âge 18+ implémentée
- ✅ Avertissement nicotine en bannière permanente
- ✅ Mentions légales dans le footer
- ✅ Confirmation d'âge lors de l'inscription
- ✅ Conformité RGPD (cookies localStorage uniquement)

**À ajouter avant mise en production :**
- [ ] Bandeau cookies RGPD complet
- [ ] Politique de confidentialité complète  
- [ ] CGV rédigées par un juriste
- [ ] Déclaration TPD (Tobacco Products Directive) auprès de l'ANSM
- [ ] Numéro SIRET visible sur le site

---

## 📁 Structure du projet

```
jnr-store/
├── index.html          ← Site complet (tout-en-un)
├── README.md           ← Ce guide
└── server.js           ← Backend optionnel (Stripe)
```

---

## 🔧 Personnalisation rapide

| Élément | Localisation dans le code |
|---------|--------------------------|
| Produits | Objet `PRODUCTS` dans le JS |
| Codes promo | Objet `PROMO_CODES` dans le JS |
| Avis clients | Tableau `REVIEWS` dans le JS |
| Couleurs | Variables CSS `:root` |
| Nom de marque | Rechercher "JNR" dans tout le fichier |
| Frais de livraison | Variable `shipping` dans `renderCart()` |
| Seuil livraison gratuite | `total >= 40` dans `renderCart()` |
