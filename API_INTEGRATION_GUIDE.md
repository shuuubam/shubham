# API Integration Guide for Auraa Jewels

This document explains how to connect the frontend to your backend API.

## Environment Configuration

Create a `.env` file in the root of your project (same level as `package.json`):

```env
REACT_APP_API_URL=http://localhost:3000/api
```

Or for production:
```env
REACT_APP_API_URL=https://your-api-domain.com/api
```

The frontend will automatically use this URL. If not set, it defaults to `http://localhost:3000/api`.

## API Endpoints Required

### 1. Get All Products
**Endpoint:** `GET /api/products`

**Response Format:**
```json
[
  {
    "id": 1,
    "title": "Elegant Necklace",
    "price": 9499,
    "img": "https://example.com/image.jpg",
    "category": "Necklaces",
    "description": "Optional: product description"
  },
  {
    "id": 2,
    "title": "Classic Diamond Ring",
    "price": 14999,
    "img": "https://example.com/image2.jpg",
    "category": "Rings",
    "description": "Optional: product description"
  }
]
```

**Error Response:**
```json
{
  "error": "Failed to fetch products"
}
```

---

### 2. Get Product by ID
**Endpoint:** `GET /api/products/:id`

**Response Format:**
```json
{
  "id": 1,
  "title": "Elegant Necklace",
  "price": 9499,
  "img": "https://example.com/image.jpg",
  "category": "Necklaces",
  "description": "Optional: product description",
  "stock": 10
}
```

---

### 3. Filter Products by Category
**Endpoint:** `GET /api/products?category=Necklaces`

**Response:** Same as "Get All Products" but filtered by category

---

### 4. Get All Categories
**Endpoint:** `GET /api/categories`

**Response Format:**
```json
[
  "Necklaces",
  "Earrings",
  "Rings",
  "Bracelets"
]
```

---

### 5. Subscribe to Newsletter
**Endpoint:** `POST /api/newsletter/subscribe`

**Request Format:**
```json
{
  "email": "user@example.com"
}
```

**Success Response:**
```json
{
  "success": true,
  "message": "Subscription successful"
}
```

**Error Response:**
```json
{
  "success": false,
  "error": "Email already subscribed"
}
```

---

### 6. Submit Contact Form
**Endpoint:** `POST /api/contact`

**Request Format:**
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "message": "I have a question about your products"
}
```

**Success Response:**
```json
{
  "success": true,
  "message": "Message received successfully"
}
```

---

### 7. Add to Cart
**Endpoint:** `POST /api/cart/add`

**Request Format:**
```json
{
  "productId": 1,
  "quantity": 1
}
```

**Success Response:**
```json
{
  "success": true,
  "cart": {
    "id": "cart-123",
    "items": [
      {
        "productId": 1,
        "quantity": 1,
        "price": 9499
      }
    ],
    "total": 9499
  }
}
```

---

## Minimal Express.js Backend Example

Here's a minimal Node.js/Express backend to get you started:

```javascript
const express = require('express');
const cors = require('cors');
const app = express();

app.use(cors());
app.use(express.json());

// Sample data (replace with database)
const products = [
  { id: 1, title: 'Elegant Necklace', price: 9499, img: 'https://...', category: 'Necklaces' },
  { id: 2, title: 'Classic Diamond Ring', price: 14999, img: 'https://...', category: 'Rings' },
  { id: 3, title: 'Modern Hoop Earrings', price: 4299, img: 'https://...', category: 'Earrings' },
  { id: 4, title: 'Delicate Bracelet', price: 6999, img: 'https://...', category: 'Bracelets' },
];

// GET all products
app.get('/api/products', (req, res) => {
  const { category } = req.query;
  if (category) {
    const filtered = products.filter(p => p.category === category);
    return res.json(filtered);
  }
  res.json(products);
});

// GET product by ID
app.get('/api/products/:id', (req, res) => {
  const product = products.find(p => p.id === parseInt(req.params.id));
  if (!product) return res.status(404).json({ error: 'Product not found' });
  res.json(product);
});

// GET categories
app.get('/api/categories', (req, res) => {
  const categories = [...new Set(products.map(p => p.category))];
  res.json(categories);
});

// POST newsletter subscribe
app.post('/api/newsletter/subscribe', (req, res) => {
  const { email } = req.body;
  if (!email) return res.status(400).json({ error: 'Email required' });
  console.log('Subscribed:', email);
  res.json({ success: true, message: 'Subscription successful' });
});

// POST contact form
app.post('/api/contact', (req, res) => {
  const { name, email, message } = req.body;
  if (!name || !email || !message) {
    return res.status(400).json({ error: 'All fields required' });
  }
  console.log('Contact:', { name, email, message });
  res.json({ success: true, message: 'Message received' });
});

// POST add to cart
app.post('/api/cart/add', (req, res) => {
  const { productId, quantity } = req.body;
  console.log('Added to cart:', { productId, quantity });
  res.json({ 
    success: true, 
    cart: { 
      items: [{ productId, quantity }],
      total: 0 
    } 
  });
});

app.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});
```

**Installation:**
```bash
npm init -y
npm install express cors
node app.js
```

---

## Frontend Error Handling

The frontend automatically handles:
- **Network errors** — displays error message in UI
- **Loading states** — shows spinner while fetching
- **Fallback data** — empty arrays if API fails (so app doesn't crash)

All API calls include try-catch and error logging in browser console.

---

## Testing the Integration

1. **Start your backend** on `http://localhost:3000`
2. **Start the frontend** (already running on `http://localhost:5173`)
3. **Check browser console** (F12) for any errors
4. **Navigate to Home** — should load products
5. **Go to Shop** — should fetch and display products
6. **Test filters** — category/price filters should work
7. **Newsletter/Contact** — submit and check backend logs

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| API not connecting | Check `.env` file and ensure backend is running |
| Products not loading | Check browser console for errors; verify API URL |
| CORS errors | Add `cors` middleware to Express backend |
| Empty products | Ensure backend endpoint returns correct JSON format |
| Newsletter fails | Check network tab in DevTools for request/response |

---

## Next Steps

- Set up a database (MongoDB, PostgreSQL, MySQL)
- Implement authentication (JWT tokens)
- Add more endpoints (order history, user profiles, etc.)
- Deploy frontend to Vercel/Netlify and backend to Heroku/Railway
