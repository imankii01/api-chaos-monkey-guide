# API Chaos Monkey - Complete Beginner's Guide

## What is API Chaos Monkey?

Think of it like a **troublemaker** for your APIs. It makes your normally well-behaved API act up randomly - just like what happens in the real world!

When you build an API, it usually works perfectly on your computer. But when real users start using it, things go wrong:
- Internet is slow
- Servers crash
- Data gets corrupted

**API Chaos Monkey** helps you test these problems BEFORE your users face them.

---

## Why Do You Need This?

### The Problem
Most developers only test the "happy path":
```javascript
// This is what we usually test
fetch('/api/users')
  .then(response => response.json()) // ✅ Always works in testing
  .then(users => displayUsers(users)) // ✅ Perfect data every time
```

### The Reality
But in real life, this happens:
```javascript
// What actually happens in production
fetch('/api/users')
  .then(response => {
    // Sometimes takes 10 seconds to respond 😱
    // Sometimes returns 500 error 💥
    // Sometimes returns broken JSON 🤯
  })
```

**API Chaos Monkey solves this by making your development environment act like the real world!**

---

## What Can It Do? (With Real Examples)

### 1. **Random Delays** 🐌
**What it does:** Makes your API respond slowly sometimes

**Real-life scenario:** Your user is on a slow mobile network or your server is busy

**Before Chaos Monkey:**
```bash
GET /api/products
✅ Response: 50ms (always fast)
```

**After Chaos Monkey:**
```bash
GET /api/products
⏰ Response: 3.2 seconds (sometimes slow!)
⏰ Response: 150ms (sometimes normal)
⏰ Response: 8 seconds (sometimes very slow!)
```

**What you learn:** Does your frontend show a loading spinner? Do users get confused?

### 2. **Random Errors** 💥
**What it does:** Makes your API fail sometimes with different error codes

**Real-life scenario:** Your database is down, server is overloaded, or rate limits hit

**Before Chaos Monkey:**
```bash
GET /api/user/profile
✅ 200 OK: { "name": "John", "email": "john@example.com" }
```

**After Chaos Monkey:**
```bash
GET /api/user/profile
❌ 500 Internal Server Error (database crashed!)
❌ 503 Service Unavailable (server overloaded!)
❌ 429 Too Many Requests (rate limited!)
✅ 200 OK: { "name": "John", "email": "john@example.com" } (sometimes works)
```

**What you learn:** Does your app show proper error messages? Does it retry failed requests?

### 3. **Broken Data** 🤯
**What it does:** Returns corrupted or weird JSON data

**Real-life scenario:** Network corruption, database issues, or encoding problems

**Before Chaos Monkey:**
```bash
GET /api/products
✅ { "products": [{"name": "iPhone", "price": 999}] }
```

**After Chaos Monkey:**
```bash
GET /api/products
🤯 { "products": [{"name": "iPh#@$%", "price": "ERROR"}] } (corrupted!)
🤯 { "products": null } (missing data!)
🤯 Invalid JSON response (parser breaks!)
```

**What you learn:** Does your app crash when data is weird? Do you handle parsing errors?

---

## Installation (Super Easy!)

```bash
npm install api-chaos-monkey
```

That's it! No complex setup needed.

---

## Basic Usage (5 Minutes Setup)

### Step 1: Add to Your Express App
```javascript
const express = require('express');
const apiChaosMonkey = require('api-chaos-monkey');

const app = express();

// Add this ONE line to enable chaos!
app.use(apiChaosMonkey({ chaos: 'mild' }));

// Your normal API routes
app.get('/api/users', (req, res) => {
  res.json([
    { id: 1, name: 'John' },
    { id: 2, name: 'Jane' }
  ]);
});

app.listen(3000, () => {
  console.log('Server running with chaos enabled!');
});
```

### Step 2: Test Your API
```bash
# Try calling your API multiple times
curl http://localhost:3000/api/users
curl http://localhost:3000/api/users
curl http://localhost:3000/api/users
```

**You'll see different responses:**
```bash
# Sometimes normal (fast)
✅ [{"id":1,"name":"John"},{"id":2,"name":"Jane"}] (100ms)

# Sometimes slow
⏰ [{"id":1,"name":"John"},{"id":2,"name":"Jane"}] (2.5 seconds)

# Sometimes error
❌ 500 Internal Server Error
```

---

## Chaos Levels (Choose Your Adventure!)

| Level | How Chaotic? | Good For |
|-------|--------------|----------|
| `mild` | 10% chaos | Daily development - won't annoy you too much |
| `wild` | 30% chaos | Testing phase - finds more problems |
| `ape-pocalypse` | 70% chaos | Stress testing - everything breaks! |

```javascript
// Gentle chaos for development
app.use(apiChaosMonkey({ chaos: 'mild' }));

// More chaos for testing
app.use(apiChaosMonkey({ chaos: 'wild' }));

// Maximum chaos for stress testing
app.use(apiChaosMonkey({ chaos: 'ape-pocalypse' }));
```

---

## Advanced Settings (When You're Ready)

```javascript
app.use(apiChaosMonkey({
  chaos: 'wild',
  probability: 0.3,                    // 30% chance of chaos per request
  delayRange: [1000, 5000],           // Delays between 1-5 seconds
  errorCodes: [500, 503, 502, 429],   // Which errors to throw
  enabledRoutes: ['/api/'],           // Only mess with API routes
  disabledRoutes: ['/health'],        // Never break health checks
  logging: true                       // See what chaos happened
}));
```

---

## Real-World Example: E-commerce App

Let's say you have an online store API:

```javascript
// Your normal e-commerce API
app.get('/api/products', (req, res) => {
  res.json([
    { id: 1, name: 'iPhone 15', price: 999, stock: 50 },
    { id: 2, name: 'MacBook Pro', price: 2499, stock: 25 }
  ]);
});

app.post('/api/checkout', (req, res) => {
  res.json({ success: true, orderId: '12345' });
});
```

**With Chaos Monkey enabled, here's what happens:**

### Scenario 1: Product Loading
```bash
# User tries to see products
GET /api/products

# Possible outcomes:
✅ Normal: Fast response with products (70% of time)
⏰ Slow: Takes 4 seconds to load (20% of time)  
❌ Error: 503 Service Unavailable (10% of time)
```

**What you discover:**
- Does your website show "Loading..." when it's slow?
- What happens when products don't load? Do you show an error message?
- Do users get frustrated and leave?

### Scenario 2: Checkout Process
```bash
# User tries to buy something
POST /api/checkout

# Possible outcomes:
✅ Success: Order placed normally
❌ 500 Error: Payment processing failed
⏰ Timeout: Takes 10 seconds (user thinks it's broken)
🤯 Weird response: { "success": "maybe", "orderId": null }
```

**What you discover:**
- Do you prevent double-clicking the "Buy" button?
- Do you show proper error messages when payment fails?
- Do you handle weird responses gracefully?

---

## Monitoring Your Chaos

See what chaos is happening:

```javascript
const chaos = apiChaosMonkey({ chaos: 'wild' });

// Check chaos statistics every 10 seconds
setInterval(() => {
  const stats = chaos.chaosEngine.getStats();
  console.log('Chaos Stats:', {
    totalRequests: stats.totalRequests,
    chaosApplied: stats.chaosCount,
    delaysAdded: stats.delays,
    errorsThrown: stats.errors
  });
}, 10000);
```

**Example output:**
```
Chaos Stats: {
  totalRequests: 100,
  chaosApplied: 32,      // 32% of requests had chaos
  delaysAdded: 18,       // 18 requests were slowed down
  errorsThrown: 14       // 14 requests returned errors
}
```

---

## When NOT to Use This

❌ **Never in production** (unless you really know what you're doing)
❌ **Not on health check endpoints** (breaks monitoring)
❌ **Not during live demos** (unless you want chaos! 😅)

✅ **Perfect for:**
- Development environment
- Testing phase
- QA testing
- Learning how your app handles problems

---

## Quick Start Checklist

1. ✅ Install: `npm install api-chaos-monkey`
2. ✅ Add one line: `app.use(apiChaosMonkey({ chaos: 'mild' }))`
3. ✅ Test your API multiple times
4. ✅ See what breaks in your frontend
5. ✅ Fix the problems you discover
6. ✅ Your app is now more resilient!

---

## Common Problems You'll Discover

After using API Chaos Monkey, you'll probably find:

1. **Your loading states suck** - No spinners when APIs are slow
2. **Error handling is missing** - App crashes when API returns errors  
3. **No retry logic** - One failure = permanent failure
4. **Poor user feedback** - Users don't know what's happening
5. **Race conditions** - Multiple API calls cause weird behavior

**That's the point!** Now you can fix these BEFORE users experience them.

---

## Examples You Can Run

```bash
# Try the basic example
npm run example

# Try advanced configuration  
npm run demo

# Run tests to see it working
npm test
```

---

## Help & Support

- 📖 [Full Documentation](https://github.com/imankii01/api-chaos-monkey)
- 🐛 [Report Issues](https://github.com/imankii01/api-chaos-monkey/issues)
- 💬 [Ask Questions](https://github.com/imankii01/api-chaos-monkey/discussions)

---

## Fun Fact

Netflix uses a similar tool called "Chaos Monkey" in production to randomly break their own services. This helps them build systems that can handle real failures. 

You're using the API version to make your APIs as resilient as Netflix's! 🎬

---

**Ready to break things and make them better? Install API Chaos Monkey and start testing the chaos! 🐒**
