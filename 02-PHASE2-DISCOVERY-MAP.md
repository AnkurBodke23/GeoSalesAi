# Phase 2: Business Discovery & Interactive Map (Weeks 4-6)

## 2.1 Product Catalogue

### Description
Implement the core product catalog enabling companies to list their offerings with detailed specifications, media, and pricing.

### Mongoose Schemas

```javascript
// backend/models/Product.js
const mongoose = require('mongoose');

const productSchema = new mongoose.Schema({
  company_id: { type: mongoose.Schema.Types.ObjectId, ref: 'Company', required: true },
  name: { type: String, required: true },
  category: { type: String, required: true },
  description: { type: String },
  specifications: [{ key: String, value: String }],
  images: [{ 
    url: String, 
    thumbnail_url: String, 
    alt: String 
  }],
  video_url: { type: String },
  models_variants: [{ name: String, sku: String, price: Number }],
  price_min: { type: Number },
  price_max: { type: Number },
  pricing_visible: { type: Boolean, default: true },
  unit: { type: String },
  min_order_qty: { type: Number, default: 1 },
  availability: { type: String, enum: ['in_stock', 'made_to_order', 'out_of_stock'] },
  service_coverage: [String],
  warranty: { type: String },
  documents: [{ name: String, url: String, type: { type: String } }],
  sku: { type: String },
  status: { type: String, enum: ['draft', 'published', 'archived'], default: 'published' }
}, { timestamps: true });

productSchema.index({ name: 'text', description: 'text', category: 'text' });
module.exports = mongoose.model('Product', productSchema);
```

---

## 2.2 Interactive Map Module

### Description
The discovery interface using MapLibre GL JS (via `react-map-gl`). The UI features a split-screen on desktop (map + list) and a bottom sheet approach on mobile devices.

### Features
- **MapLibre GL JS** integration in React.
- **Location Detection:** Browser Geolocation API to center map.
- **Geocoder:** Photon API (OpenStreetMap) for manual location searching.
- **Markers:** Custom company markers indicating industry type, with clustering for zoomed-out views (using Supercluster).
- **Popups:** Clicking a marker shows a compact business card summary.

---

## 2.3 Geospatial Search

### Description
Query APIs powering the map and list views based on physical proximity using MongoDB's spatial indexing capabilities.

### MongoDB Query Examples

**Find companies near a point (`$nearSphere`):**
```javascript
// GET /api/discovery/nearby?lng=77.2090&lat=28.6139&radius=10000
const { lng, lat, radius } = req.query; // radius in meters

const companies = await Company.find({
  'addresses.location': {
    $nearSphere: {
      $geometry: {
        type: 'Point',
        coordinates: [parseFloat(lng), parseFloat(lat)]
      },
      $maxDistance: parseInt(radius)
    }
  }
}).select('name slug industry logo verification_status addresses feed_visibility_score');
```

**Find companies within map bounds (`$geoWithin`):**
```javascript
// GET /api/discovery/bounds?swLng=77.1&swLat=28.5&neLng=77.3&neLat=28.7
const { swLng, swLat, neLng, neLat } = req.query;

const companies = await Company.find({
  'addresses.location': {
    $geoWithin: {
      $box: [
        [parseFloat(swLng), parseFloat(swLat)], // Bottom Left
        [parseFloat(neLng), parseFloat(neLat)]  // Top Right
      ]
    }
  }
});
```

---

## 2.4 Search & Filtering

### Description
Enhance discovery with text searching and granular filtering. 

- **Text Search:** Utilizing MongoDB `$text` search on the text indexes created on Company and Product models.
- **Filter State:** State stored in URL search parameters to allow deep linking to specific searches (e.g., `?industry=Pharma&verified=true`).
- **Debouncing:** Implementation of debounced input in React (e.g., using `use-debounce`) to prevent excessive API calls while typing.

---

## 2.5 Business Cards & Company Preview

### Description
UI components designed for high-density information display in search results and map popups.

- **Compact Card:** Displays Logo, Name, Distance, Verification Badge.
- **Follow System:** 
  - `POST /api/companies/:id/follow`
  - Allows users to curate a feed of relevant businesses.

---

## 2.6 Ranking & Discovery Signals

### Description
Algorithms to determine the sort order of discovered companies.

- **Configurable Sort:** Users can sort by Distance, Relevance (text match score), or Activity.
- **Visibility Score:** Companies gain a higher `feed_visibility_score` by posting updates, keeping profiles complete, and having active memberships. 

---

## 2.7 Week-by-Week Breakdown

- **Week 4:** Implement Product schema, CRUD endpoints. Build frontend product pages. Set up `react-map-gl` base map component.
- **Week 5:** Write Geospatial MongoDB queries. Implement map markers, clustering, and location bounds update triggers.
- **Week 6:** Build the search sidebar with filters. Implement the company Follow system. Finalize ranking logic and business card UI components.

---

## 2.8 Testing Checklist

**Products & Map Features:**
- [ ] Products can be created, updated, and deleted by authorized company members.
- [ ] Map renders correctly and centers on user location (if permission granted).
- [ ] `$nearSphere` query returns correct businesses within the radius, ordered by distance.
- [ ] Panning the map triggers a `$geoWithin` bounds query and updates the marker list.
- [ ] Marker clustering activates successfully on low zoom levels.
- [ ] Text search combines with geospatial filtering correctly.
- [ ] URL query parameters update on filter change and hydrate filters on page reload.
