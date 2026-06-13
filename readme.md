# 🌍 WanderLust — Full-Stack Vacation Rental Platform

WanderLust is a **full-stack web application** inspired by Airbnb that allows users to browse, list, review, and manage vacation rental properties. It follows the **MVC (Model-View-Controller)** architecture pattern and implements complete **CRUD operations**, **user authentication & authorization**, **image uploads to the cloud**, **interactive maps**, and **server-side validation**.

> **Tech Stack:** Node.js · Express.js · MongoDB · Mongoose · EJS · Passport.js · Cloudinary · MapTiler · Bootstrap 5 · Joi

---

## 📑 Table of Contents

- [Features](#-features)
- [Architecture Overview (MVC)](#-architecture-overview-mvc)
- [Folder Structure](#-folder-structure)
- [Database Schemas](#-database-schemas)
- [API Routes](#-api-routes)
- [Middleware](#-middleware)
- [Authentication & Authorization](#-authentication--authorization)
- [Third-Party Integrations](#-third-party-integrations)
- [Frontend Details](#-frontend-details)
- [Environment Variables](#-environment-variables)
- [Getting Started](#-getting-started)
- [Key Concepts for Interview](#-key-concepts-for-interview)

---

## ✨ Features

| Feature | Description |
|---|---|
| **View All Listings** | Browse a grid of vacation rental cards with images, prices, and location info |
| **View Listing Details** | See full listing info including description, price, location, owner, reviews, and an interactive map |
| **Create New Listing** | Authenticated users can add properties with image upload (stored on Cloudinary) |
| **Edit Listing** | Only the listing owner can edit their property (authorization check) |
| **Delete Listing** | Only the listing owner can delete; associated reviews are cascade-deleted |
| **User Signup / Login / Logout** | Full authentication via Passport.js with session management |
| **Add Reviews** | Authenticated users can leave star ratings (1–5) and comments |
| **Delete Reviews** | Only the review author can delete their own review |
| **Interactive Maps** | Each listing page shows a MapTiler map pinpointing the property location |
| **Geocoding** | Listing location is auto-geocoded to coordinates using the MapTiler Geocoding API |
| **Image Upload** | Images uploaded via Multer, stored on Cloudinary (not local filesystem) |
| **Tax Toggle** | On the index page, a switch displays prices with/without 18% GST |
| **Flash Messages** | Success/error flash messages for user feedback on every action |
| **Form Validation** | Client-side (Bootstrap) + server-side (Joi schema) validation |
| **Responsive Design** | Mobile-friendly UI using Bootstrap 5 grid system |
| **Session Store** | Sessions persisted in MongoDB via `connect-mongo` (production-ready) |

---

## 🏗 Architecture Overview (MVC)

The project follows the **Model-View-Controller** pattern to separate concerns:

```
┌──────────────────────────────────────────────────────────────────┐
│                        CLIENT (Browser)                          │
│   Sends HTTP requests (GET, POST, PUT, DELETE)                   │
└─────────────────────────────┬────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│                     ROUTES (routes/*.js)                          │
│   Maps URL patterns to controller functions                      │
│   Applies middleware: isLoggedIn, isOwner, validateListing, etc. │
└─────────────────────────────┬────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│                  CONTROLLERS (controllers/*.js)                   │
│   Contains business logic for each route                         │
│   Interacts with Models, renders Views                           │
└──────────┬──────────────────────────────────┬────────────────────┘
           │                                  │
           ▼                                  ▼
┌────────────────────────┐      ┌──────────────────────────────────┐
│   MODELS (models/*.js) │      │       VIEWS (views/**/*.ejs)      │
│   Mongoose schemas     │      │   EJS templates rendered with     │
│   Data layer           │      │   data from controllers           │
└────────────────────────┘      └──────────────────────────────────┘
```

### Request Lifecycle (Example: Creating a Listing)

```
1. User submits form → POST /listings
2. Router matches route → applies middleware chain:
   a. validateListing (Joi schema check)
   b. isLoggedIn (authentication check)
   c. upload.single("image") (Multer/Cloudinary upload)
3. Controller: listingController.createListing
   a. Extracts form data from req.body
   b. Calls MapTiler Geocoding API to get coordinates
   c. Creates new Listing document with owner = req.user._id
   d. Saves to MongoDB
   e. Sets flash message → redirects to /listings
4. Browser renders the listing index page with success flash
```

---

## 📁 Folder Structure

```
WANDERLUST/
├── app.js                    # Application entry point — Express config, DB connection, middleware, routes
├── cloudConfig.js            # Cloudinary + Multer-Storage-Cloudinary configuration
├── middleware.js              # Custom middleware: auth checks, validation, authorization
├── schema.js                  # Joi validation schemas for Listing and Review
├── package.json               # Dependencies and project metadata
│
├── models/                    # Mongoose schemas (Data Layer)
│   ├── listing.js             # Listing schema — title, description, image, price, location, reviews, owner, geometry
│   ├── review.js              # Review schema — comment, rating (1-5), author, createdAt
│   └── user.js                # User schema — email + passport-local-mongoose plugin (adds username, hash, salt)
│
├── routes/                    # Express Router definitions (URL → Controller mapping)
│   ├── listing.js             # All /listings routes with middleware chains
│   ├── review.js              # All /listings/:id/reviews routes (mergeParams: true)
│   └── user.js                # /signup, /login, /logout routes
│
├── controllers/               # Business logic (Controller Layer)
│   ├── listing.js             # CRUD handlers for listings + geocoding logic
│   ├── reviews.js             # Create and delete review handlers
│   └── users.js               # Signup, login, logout handlers
│
├── views/                     # EJS templates (View Layer)
│   ├── layouts/
│   │   └── boilerplate.ejs    # Master layout — HTML head, Bootstrap/Font Awesome CDN, navbar, footer
│   ├── includes/
│   │   ├── navbar.ejs         # Navigation bar with search, conditional login/logout links
│   │   ├── flash.ejs          # Success and error flash message alerts
│   │   └── footer.ejs         # Footer with social links and copyright
│   ├── listings/
│   │   ├── index.ejs          # All listings page — card grid, category filters, tax toggle
│   │   ├── show.ejs           # Single listing detail — image, info, reviews, map
│   │   ├── new.ejs            # Create new listing form (with file upload)
│   │   ├── edit.ejs           # Edit listing form (pre-filled, with image preview)
│   │   └── error.ejs          # Error display page
│   └── users/
│       ├── signup.ejs         # User registration form
│       └── login.ejs          # User login form
│
├── public/                    # Static assets served by Express
│   └── css/
│       ├── style.css          # Custom CSS — navbar, footer, cards, show page, map
│       ├── rating.css         # Starability CSS library for star rating widget
│       └── js/
│           ├── script.js      # Bootstrap client-side form validation
│           └── map.js         # MapTiler SDK — geocode location, render interactive map
│
├── utils/                     # Utility classes
│   ├── ExpressError.js        # Custom Error class with statusCode and message
│   └── wrapAsync.js           # Async error wrapper — catches promise rejections, passes to next()
│
├── init/                      # Database seeding
│   ├── data.js                # Array of 28 sample listings with Unsplash images
│   └── index.js               # Seed script — connects to MongoDB, inserts sample data
│
├── .env                       # Environment variables (NOT committed to git)
└── .gitignore                 # Ignores: .env, node_modules/, .DS_Store
```

### File-by-File Breakdown

| File | Purpose | Key Concepts |
|---|---|---|
| `app.js` | Entry point. Sets up Express, connects to MongoDB, configures sessions (with MongoStore), flash, Passport, and mounts all routers | Express middleware chain, `express-session`, `connect-mongo`, `ejs-mate` |
| `cloudConfig.js` | Configures Cloudinary SDK and creates a `CloudinaryStorage` instance for Multer | Cloud storage, file upload pipeline |
| `middleware.js` | Exports 5 middleware functions for authentication, authorization, and validation | `req.isAuthenticated()`, Joi validation, `$pull` operator |
| `schema.js` | Defines Joi schemas for server-side request body validation | Schema validation, type coercion |
| `models/listing.js` | Mongoose schema with GeoJSON `geometry` field and a `post("findOneAndDelete")` hook for cascade-deleting reviews | Mongoose hooks, GeoJSON, `ObjectId` references |
| `models/review.js` | Review schema with rating (1-5), comment, author reference, and timestamps | Population, referencing |
| `models/user.js` | Minimal schema with `email`; `passport-local-mongoose` plugin auto-adds `username`, `hash`, `salt` fields | Passport plugin pattern |
| `utils/wrapAsync.js` | HOF that wraps async route handlers: `fn(req, res, next).catch(next)` | Error propagation in Express |
| `utils/ExpressError.js` | Extends `Error` with a `statusCode` property for HTTP error responses | Custom error classes |

---

## 🗄 Database Schemas

### Listing (`models/listing.js`)

```javascript
{
  title:       { type: String, required: true },
  description: String,
  image: {
    url:      String,    // Cloudinary URL
    filename: String     // Cloudinary public_id (used for deletion)
  },
  price:       Number,
  location:    String,   // City name (e.g., "Malibu")
  country:     String,   // Country name (e.g., "United States")
  reviews:     [ObjectId → Review],   // Array of Review references
  owner:       ObjectId → User,       // Reference to the User who created it
  geometry: {                          // GeoJSON Point for map coordinates
    type:        { type: String, enum: ["Point"], required: true },
    coordinates: { type: [Number], required: true }  // [longitude, latitude]
  }
}
```

**Mongoose Middleware (Post Hook):**
```javascript
listingSchema.post("findOneAndDelete", async (listing) => {
  // When a listing is deleted, cascade-delete all its reviews
  if (listing) {
    await Review.deleteMany({ _id: { $in: listing.reviews } });
  }
});
```
> **Interview Tip:** This is a Mongoose `post` hook on `findOneAndDelete`. It runs _after_ the document is deleted. The `$in` operator deletes all reviews whose IDs are in the listing's `reviews` array. This prevents orphan review documents.

---

### Review (`models/review.js`)

```javascript
{
  comment:   String,
  rating:    { type: Number, min: 1, max: 5 },
  createdAt: { type: Date, default: Date.now },
  author:    ObjectId → User   // Reference to the User who wrote the review
}
```

---

### User (`models/user.js`)

```javascript
{
  email: { type: String, required: true }
  // passport-local-mongoose plugin automatically adds:
  //   username: String (unique)
  //   hash:     String (hashed password)
  //   salt:     String (random salt)
}
```

> **Interview Tip:** `passport-local-mongoose` is a Mongoose plugin that simplifies building username/password login. It adds `.register()`, `.authenticate()`, `.serializeUser()`, `.deserializeUser()` static methods to the User model — so you never store plain-text passwords.

---

### Relationships Diagram

```
┌─────────┐       ┌──────────┐       ┌────────┐
│  User   │◄──────│ Listing  │──────►│ Review │
│         │ owner │          │reviews│        │
│username │       │ title    │       │ rating │
│email    │       │ price    │       │comment │
│hash     │       │ location │       │        │
│salt     │       │ image    │       │ author │──► User
└─────────┘       │ geometry │       └────────┘
                  └──────────┘
```

- A **User** can own many **Listings** (one-to-many via `owner` field)
- A **Listing** can have many **Reviews** (one-to-many via `reviews` array of ObjectIds)
- A **Review** has one **author** (many-to-one reference to User)

---

## 🛣 API Routes

### Listing Routes — `routes/listing.js` (mounted at `/listings`)

| Method | Endpoint | Middleware | Controller | Description |
|--------|----------|------------|------------|-------------|
| `GET` | `/listings` | — | `listingController.index` | Fetch all listings from DB, render index page with card grid |
| `GET` | `/listings/new` | `isLoggedIn` | `listingController.renderNewForm` | Render the "Create New Listing" form |
| `POST` | `/listings` | `validateListing` → `isLoggedIn` → `upload.single("image")` | `listingController.createListing` | Validate body → check auth → upload image to Cloudinary → geocode location → save listing → redirect |
| `GET` | `/listings/:id` | — | `listingController.showListing` | Find listing by ID, populate reviews (nested populate for author) and owner, render show page |
| `GET` | `/listings/:id/edit` | `isLoggedIn` → `isOwner` | `listingController.renderEditForm` | Check ownership → render edit form with pre-filled data & thumbnail |
| `PUT` | `/listings/:id` | `validateListing` → `isLoggedIn` → `isOwner` → `upload.single("image")` | `listingController.updateListing` | Validate → auth → ownership → re-geocode → update listing → optionally update image |
| `DELETE` | `/listings/:id` | `isLoggedIn` → `isOwner` | `listingController.destroyListing` | Auth + ownership check → delete listing (triggers cascade review deletion via post hook) |

> **Interview Tip — Route Ordering:** The `/listings/new` GET route is defined _before_ `/listings/:id` GET route. This is critical because Express matches routes top-down, and `/new` would otherwise be treated as an `:id` parameter.

### Review Routes — `routes/review.js` (mounted at `/listings/:id/reviews`)

| Method | Endpoint | Middleware | Controller | Description |
|--------|----------|------------|------------|-------------|
| `POST` | `/listings/:id/reviews` | `isLoggedIn` → `validateReview` | `reviewController.createReview` | Create review, push to listing's reviews array, save both |
| `DELETE` | `/listings/:id/reviews/:reviewId` | `isLoggedIn` → `isReviewAuthor` | `reviewController.destroyReview` | `$pull` review from listing's array, delete review document |

> **Interview Tip — `mergeParams: true`:** The review router uses `express.Router({ mergeParams: true })`. This allows the review routes to access `:id` from the parent route (`/listings/:id/reviews`), which is necessary because the review router is mounted as a sub-router.

### User Routes — `routes/user.js` (mounted at `/`)

| Method | Endpoint | Middleware | Controller | Description |
|--------|----------|------------|------------|-------------|
| `GET` | `/signup` | — | `userController.renderSignupForm` | Render signup form |
| `POST` | `/signup` | — | `userController.signup` | Create user via `User.register()`, auto-login via `req.login()` |
| `GET` | `/login` | — | `userController.renderLoginForm` | Render login form |
| `POST` | `/login` | `saveRedirectUrl` → `passport.authenticate("local")` | `userController.login` | Authenticate with Passport → redirect to saved URL or `/listings` |
| `GET` | `/logout` | — | `userController.logout` | Call `req.logout()`, flash message, redirect |

### Root Route (defined in `app.js`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/` | Redirects to `/listings` |
| `*` | `/*path` (catch-all) | Throws `ExpressError(404, "Page Not Found")` |

---

## 🔒 Middleware

All custom middleware is defined in `middleware.js`:

### 1. `isLoggedIn`
```javascript
if (!req.isAuthenticated()) {
  req.session.redirectUrl = req.originalUrl;  // Save where user was trying to go
  req.flash("error", "You must be signed in first!");
  return res.redirect("/login");
}
next();
```
> Checks if the user has an active session via Passport's `req.isAuthenticated()`. Saves the intended URL for post-login redirect.

### 2. `saveRedirectUrl`
```javascript
if (req.session.redirectUrl) {
  res.locals.redirectUrl = req.session.redirectUrl;
}
next();
```
> Runs _before_ `passport.authenticate()` on the login POST route. Passport resets the session on login, so this copies the redirect URL to `res.locals` before the session is cleared.

### 3. `isOwner`
```javascript
let listing = await Listing.findById(id);
if (!listing.owner._id.equals(res.locals.currentUser._id)) {
  req.flash("error", "You are not the owner of this listing!");
  return res.redirect(`/listings/${id}`);
}
next();
```
> **Authorization middleware** — ensures only the listing creator can edit/delete it. Uses Mongoose's `.equals()` method to compare ObjectIds.

### 4. `validateListing`
```javascript
let { error } = listingSchema.validate(req.body);
if (error) {
  let msg = error.details.map(el => el.message).join(",");
  throw new ExpressError(400, msg);
}
next();
```
> **Server-side validation** using Joi. Validates that `title`, `description`, `location`, `price`, and `country` are present and correctly typed. Runs _before_ the controller to reject bad data early.

### 5. `validateReview`
> Same pattern as `validateListing` but for review data (rating 1-5, comment required).

### 6. `isReviewAuthor`
> Same pattern as `isOwner` but checks if the currently logged-in user is the author of the review.

---

## 🔐 Authentication & Authorization

### Authentication Flow (Passport.js)

```
                    ┌─────────────┐
                    │   Signup    │
                    │ POST /signup│
                    └──────┬──────┘
                           │
                    User.register(user, password)
                    (passport-local-mongoose hashes password)
                           │
                    req.login(registeredUser)
                    (creates session automatically)
                           │
                           ▼
                    ┌─────────────┐
                    │  Session    │
                    │  Created    │
                    │(MongoStore) │
                    └──────┬──────┘
                           │
                    ┌──────┴──────┐
                    │             │
              ┌─────┴─────┐ ┌────┴─────┐
              │  Login    │ │  Logout  │
              │POST /login│ │GET /logout│
              └─────┬─────┘ └────┬─────┘
                    │            │
          passport.authenticate  req.logout()
          ("local", {...})       (destroys session)
                    │
          saveRedirectUrl middleware
          (preserves intended destination)
```

**Session Configuration (in `app.js`):**
```javascript
const sessionOptions = {
  store: MongoStore.create({
    mongoUrl: process.env.DB_URL,
    touchAfter: 24 * 60 * 60,     // Lazy session update — only once per 24h
    crypto: { secret: process.env.SECRET }  // Encrypt session data in MongoDB
  }),
  secret: process.env.SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    expires: Date.now() + 7 * 24 * 60 * 60 * 1000,  // 7-day expiry
    maxAge: 7 * 24 * 60 * 60 * 1000,
    httpOnly: true,   // Prevents client-side JS from reading cookie (XSS protection)
  },
};
```

### Authorization Matrix

| Action | Guest | Logged-In User | Listing Owner | Review Author |
|--------|-------|---------------|---------------|---------------|
| View all listings | ✅ | ✅ | ✅ | ✅ |
| View listing details | ✅ | ✅ | ✅ | ✅ |
| Create listing | ❌ | ✅ | ✅ | ✅ |
| Edit listing | ❌ | ❌ | ✅ | — |
| Delete listing | ❌ | ❌ | ✅ | — |
| Add review | ❌ | ✅ | ✅ | ✅ |
| Delete review | ❌ | ❌ | — | ✅ |

---

## 🔌 Third-Party Integrations

### 1. Cloudinary (Image Storage)

**Why?** Storing images on a cloud service instead of the local filesystem makes the app deployment-ready (e.g., on Render, Railway, Heroku where filesystem is ephemeral).

**Flow:**
```
User uploads file → Multer processes multipart/form-data
                   → multer-storage-cloudinary sends to Cloudinary
                   → Returns { path: cloudinaryURL, filename: publicId }
                   → Controller stores url + filename in Listing.image
```

**Configuration (`cloudConfig.js`):**
```javascript
cloudinary.config({
  cloud_name: process.env.CLOUD_NAME,
  api_key: process.env.CLOUD_API_KEY,
  api_secret: process.env.CLOUD_API_SECRET,
});

const storage = new CloudinaryStorage({
  cloudinary,
  params: {
    folder: "wanderlust-dev",            // All images go into this Cloudinary folder
    allowedFormats: ["jpeg", "png", "jpg"],
  },
});
```

### 2. MapTiler (Maps + Geocoding)

**Geocoding (Server-Side, in `controllers/listing.js`):**
```
User enters location "Malibu, United States"
→ Controller calls MapTiler Geocoding API:
  GET https://api.maptiler.com/geocoding/Malibu%2C%20United%20States.json?key=MAP_TOKEN
→ Response contains coordinates [lng, lat]
→ Stored in listing.geometry.coordinates
```

**Map Rendering (Client-Side, in `public/css/js/map.js`):**
```
show.ejs injects mapToken + placeName as global JS variables
→ map.js fetches geocoding API for coordinates
→ Creates MapTiler map centered on listing location
→ Renders in the #map div on the show page
```

### 3. Passport.js (Authentication)

- **Strategy:** `passport-local` (username + password)
- **Plugin:** `passport-local-mongoose` on the User model
- **Session Serialization:** `User.serializeUser()` / `User.deserializeUser()` (auto-provided by plugin)
- **Session Store:** MongoDB via `connect-mongo` (production-safe, survives server restarts)

### 4. Joi (Validation)

Server-side validation schemas defined in `schema.js`:

```javascript
// Listing validation
listingSchema = Joi.object({
  title:       Joi.string().required(),
  description: Joi.string().required(),
  image:       Joi.string().allow("", null),
  location:    Joi.string().required(),
  price:       Joi.number().required().min(0),
  country:     Joi.string().required(),
});

// Review validation
reviewSchema = Joi.object({
  review: Joi.object({
    rating:  Joi.number().required().min(1).max(5),
    comment: Joi.string().required(),
  }).required(),
});
```

---

## 🎨 Frontend Details

### Templating Engine: EJS + `ejs-mate`

- **EJS** renders dynamic HTML with embedded JavaScript (`<%= %>` for output, `<% %>` for logic)
- **ejs-mate** adds layout support. `boilerplate.ejs` is the master layout; pages use `<% layout("/layouts/boilerplate") %>` and their content is injected via `<%- body %>`

### CSS Framework: Bootstrap 5.3

- Grid system for responsive layouts (`row-cols-lg-3`, `col-8 offset-2`, etc.)
- Components: Navbar, Cards, Forms, Alerts, Switches
- Loaded via CDN in `boilerplate.ejs`

### Icons: Font Awesome 6

- Used for category filter icons, search icon, social media icons, brand logo (plane icon)

### Typography: Plus Jakarta Sans (Google Fonts)

### Key UI Components

| Component | File | Description |
|-----------|------|-------------|
| **Navbar** | `views/includes/navbar.ejs` | Sticky top navbar with brand logo, "Explore" link, search bar, "Become host" link, conditional Sign Up/Login/Logout buttons |
| **Flash Messages** | `views/includes/flash.ejs` | Bootstrap dismissible alerts for success (green) and error (red) messages |
| **Footer** | `views/includes/footer.ejs` | Social icons (Facebook, Instagram, LinkedIn), copyright, Privacy/Terms links |
| **Listing Cards** | `views/listings/index.ejs` | Card grid with image, title, price with INR formatting, hover overlay effect |
| **Category Filters** | `views/listings/index.ejs` | Icon filters: Trending, Rooms, Iconic Cities, Mountains, Castles, Pools, Camping, Farms, Arctic |
| **Tax Toggle** | `views/listings/index.ejs` | Bootstrap switch that recalculates all prices with 18% GST in real-time (client-side JS) |
| **Star Ratings** | `views/listings/show.ejs` | Starability CSS library for animated star rating input and display |
| **Map** | `views/listings/show.ejs` | MapTiler SDK interactive map showing listing location |

### Client-Side JavaScript

| File | Purpose |
|------|---------|
| `public/css/js/script.js` | Bootstrap form validation — prevents submission of invalid forms, adds `.was-validated` class |
| `public/css/js/map.js` | Fetches MapTiler geocoding API, creates interactive map with listing coordinates |
| Inline in `index.ejs` | Tax toggle logic — recalculates prices by multiplying base price × 1.18, formats in INR locale |

---

## 🔑 Environment Variables

Create a `.env` file in the project root with:

```env
DB_URL=mongodb://127.0.0.1:27017/wanderlust       # MongoDB connection string
SECRET=your_session_secret_here                     # Session encryption secret
CLOUD_NAME=your_cloudinary_cloud_name               # Cloudinary cloud name
CLOUD_API_KEY=your_cloudinary_api_key               # Cloudinary API key
CLOUD_API_SECRET=your_cloudinary_api_secret          # Cloudinary API secret
MAP_TOKEN=your_maptiler_api_key                      # MapTiler API key for maps + geocoding
```

> **Note:** The `.env` file is listed in `.gitignore` and should never be committed to version control.

> **How it works:** In development (when `NODE_ENV !== "production"`), the `dotenv` package loads these variables into `process.env` automatically at app startup.

---

## 🚀 Getting Started

### Prerequisites

- **Node.js** (v20+ recommended)
- **MongoDB** (local or cloud — MongoDB Atlas)
- **Cloudinary** account (free tier works)
- **MapTiler** account (free tier works)

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-username/wanderlust.git
   cd wanderlust/WANDERLUST
   ```

2. **Install dependencies:**
   ```bash
   npm install
   ```

3. **Set up environment variables:**
   - Create a `.env` file in the `WANDERLUST/` directory
   - Add all variables listed in the [Environment Variables](#-environment-variables) section

4. **Start MongoDB** (if running locally):
   ```bash
   mongod
   ```

5. **Seed the database with sample data:**
   ```bash
   node init/index.js
   ```
   > This inserts 28 sample listings from around the world.

6. **Start the server:**
   ```bash
   node app.js
   # Or with auto-restart on file changes:
   nodemon app.js
   ```

7. **Open in browser:**
   ```
   http://localhost:8080
   ```

---

## 🗂 Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| `express` | ^5.1.0 | Web framework |
| `mongoose` | ^8.16.0 | MongoDB ODM |
| `ejs` | ^3.1.10 | Templating engine |
| `ejs-mate` | ^4.0.0 | EJS layout support |
| `express-session` | ^1.18.1 | Session management |
| `connect-mongo` | ^5.1.0 | MongoDB session store |
| `connect-flash` | ^0.1.1 | Flash messages |
| `passport` | ^0.7.0 | Authentication framework |
| `passport-local` | ^1.0.0 | Username/password strategy |
| `passport-local-mongoose` | ^8.0.0 | Mongoose plugin for Passport |
| `multer` | ^2.0.1 | Multipart form-data handling (file uploads) |
| `multer-storage-cloudinary` | ^4.0.0 | Cloudinary storage engine for Multer |
| `cloudinary` | ^1.41.3 | Cloudinary Node.js SDK |
| `@maptiler/sdk` | ^3.5.0 | MapTiler SDK |
| `node-fetch` | ^3.3.2 | Server-side HTTP requests (for geocoding API) |
| `joi` | ^17.13.3 | Schema validation |
| `method-override` | ^3.0.0 | Support PUT/DELETE in HTML forms |
| `dotenv` | ^17.0.1 | Load .env variables |
| `cookie-parser` | ^1.4.7 | Cookie parsing |

---

<!-- ## 🎯 Key Concepts for Interview

### 1. MVC Architecture
> **Q: Why use MVC?**
> Separation of concerns. Models handle data, Views handle presentation, Controllers handle logic. This makes code maintainable, testable, and scalable. Each layer can change independently.

### 2. RESTful Routing
> **Q: Is this app RESTful?**
> Yes. It follows REST conventions: `GET /listings` (Index), `GET /listings/:id` (Show), `POST /listings` (Create), `PUT /listings/:id` (Update), `DELETE /listings/:id` (Destroy). HTML forms only support GET/POST, so `method-override` is used to simulate PUT and DELETE via `?_method=PUT`.

### 3. Authentication vs Authorization
> **Q: What's the difference?**
> - **Authentication** = "Who are you?" → Handled by Passport.js (`isLoggedIn` middleware)
> - **Authorization** = "Are you allowed to do this?" → Handled by `isOwner` and `isReviewAuthor` middleware

### 4. Password Security
> **Q: How are passwords stored?**
> Never in plain text. `passport-local-mongoose` uses **PBKDF2** hashing with a random salt. The `hash` and `salt` fields are stored in MongoDB. During login, the submitted password is hashed with the same salt and compared.

### 5. Session Management
> **Q: Why use MongoStore for sessions?**
> Default Express sessions use in-memory storage (MemoryStore), which:
> - Leaks memory over time
> - Loses all sessions on server restart
> - Can't scale across multiple server instances
>
> `connect-mongo` persists sessions in MongoDB, solving all three problems.

### 6. Cascade Delete Pattern
> **Q: What happens to reviews when a listing is deleted?**
> A Mongoose `post("findOneAndDelete")` hook on the Listing schema automatically deletes all associated reviews. This prevents orphan documents in the database.

### 7. `wrapAsync` Utility
> **Q: Why not just use try-catch in every route?**
> `wrapAsync` is a higher-order function that wraps async handlers: `fn(req, res, next).catch(next)`. If any `await` throws an error, it's automatically forwarded to Express's error-handling middleware. This eliminates repetitive try-catch blocks.

### 8. `mergeParams` in Express Router
> **Q: Why does the review router need `mergeParams: true`?**
> Review routes are mounted at `/listings/:id/reviews`. Without `mergeParams`, the review router can't access `:id` from the parent. `mergeParams: true` merges parent route params into the child router.

### 9. Error Handling Strategy
> The app has a two-layer error strategy:
> 1. **`wrapAsync`** — catches async errors and passes to `next()`
> 2. **Global error handler** (in `app.js`) — catches all errors, renders `error.ejs` with message and status code
> 3. **Catch-all 404** — `app.all("/*path")` throws `ExpressError(404, "Page Not Found")` for undefined routes

### 10. Image Upload Pipeline
> ```
> HTML Form (enctype="multipart/form-data")
>   → Multer parses the file from the request body
>   → CloudinaryStorage uploads to Cloudinary cloud
>   → Returns { path: URL, filename: publicId }
>   → Controller saves URL + filename in MongoDB
> ```

### 11. Geocoding Flow
> When creating/updating a listing:
> 1. Controller concatenates `location + country` → e.g., `"Malibu, United States"`
> 2. Calls MapTiler Geocoding API with the place name
> 3. Extracts `[longitude, latitude]` from the first result
> 4. Stores as GeoJSON `Point` in `listing.geometry`
> 5. On the show page, client-side JS uses these coordinates to render a map

### 12. Joi vs Mongoose Validation
> - **Mongoose validation** catches bad data at the _database_ level (e.g., `required: true`)
> - **Joi validation** catches bad data at the _request_ level (before it reaches the model)
> - Using both provides defense-in-depth: Joi rejects clearly invalid requests early, Mongoose ensures data integrity even if Joi is bypassed

### 13. Flash Messages
> Flash messages use `connect-flash`, which stores messages in the session. They are consumed once — after being displayed, they're automatically removed. In `app.js`, `res.locals.success` and `res.locals.error` make them available to all EJS templates.

### 14. Method Override
> HTML forms only support `GET` and `POST`. To perform `PUT` (update) and `DELETE` operations, the app uses `method-override`:
> ```html
> <form method="post" action="/listings/<%= listing._id %>?_method=PUT">
> ```
> The middleware intercepts the query parameter and rewrites the request method.

### 15. Cloudinary Image Transformation
> In the edit form, the original image thumbnail is shown using Cloudinary's URL transformation API:
> ```javascript
> originalImage = listing.image.url.replace("/upload", "/upload/w_250");
> ```
> This dynamically generates a 250px-wide version without storing a separate file.

---

## 📜 License

ISC

--- -->

**Author:** Ayush
