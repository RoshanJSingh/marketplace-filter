# Vibe Marketplace: filter sidebar for an online store

Live demo: [quantiphi2.vercel.app](https://quantiphi2.vercel.app/)

A product browsing page where you can filter by **category**, **price range** and **minimum star rating** at the same time, and sort the results. I built it for the Quantiphi Vibe Coding round (Set-E).

All the filtering and sorting happens on the server. The React frontend only shows the results and sends the user's choices back to the API.

## Layout

```
marketplace-filter/
├── backend/                         Express API, all the business logic
│   └── src/
│       ├── data/products.js         the product list (30 items, kept in memory)
│       ├── services/filterService.js  filtering and sorting
│       ├── routes/products.js       /api/products and /api/meta, with input checks
│       └── server.js                entry point
└── frontend/                        React (Vite), display and user input only
    └── src/
        ├── api.js                   small fetch wrapper; no filtering happens here
        ├── App.jsx                  holds the filter state and re-queries the API
        └── components/
            ├── FilterSidebar.jsx    sidebar that holds the filters
            ├── CategoryFilter.jsx   category checkboxes
            ├── PriceRangeSlider.jsx slider with a min and a max handle
            ├── RatingFilter.jsx     1 to 5 star options
            ├── SortDropdown.jsx     Sort By menu
            ├── ProductGrid.jsx      product cards, or an empty state with a Reset button
            └── ProductCard.jsx      image, name, rating and price
```

## How the filtering works

`backend/src/services/filterService.js` goes through the product list once and keeps a product only if it passes every filter that is switched on:

1. **Category:** the product's category is one of the ticked boxes. If nothing is ticked, this filter is skipped.
2. **Price:** the price is between the minimum and the maximum. Either limit can be left empty.
3. **Rating:** the rating is at least the chosen minimum. Skipped when "Any rating" is selected.

A filter that is not in use is simply `null` or empty and gets ignored, so clearing everything gives you the full list back.

Sorting happens after filtering. `queryProducts(criteria, sortBy)` filters the original list first, then sorts what is left by `price-asc`, `price-desc` or `rating-desc`. If the sort key is empty or unknown, the original order is kept.

## Instant updates

There is no Submit button. Every click, slider move or sort change updates the React state, which calls `GET /api/products` again. Slider changes are debounced a little so dragging does not flood the API. If nothing matches, the grid is replaced by a "No items match your criteria" message with a button that resets all filters.

## API

| Endpoint | What it returns |
|---|---|
| `GET /api/products?categories=Footwear,Apparel&minPrice=20&maxPrice=150&minRating=4&sortBy=price-asc` | The filtered and sorted products. Every parameter is optional. |
| `GET /api/meta` | The list of categories, the lowest and highest price, and the total count. The sidebar is built from this. |

Bad input is handled on the server. Price limits that are not numbers are ignored, the rating is kept between 1 and 5, unknown sort keys are dropped, and a minimum price above the maximum returns `400`.

## Running it locally

```bash
# Terminal 1: backend on port 3001
cd backend && npm install && npm start

# Terminal 2: frontend on port 5173 (forwards /api to the backend)
cd frontend && npm install && npm run dev
```

Then open http://localhost:5173.

## Deploying on Vercel

The whole repo deploys as a single Vercel project with no extra setup:

- `vercel.json` builds `frontend/` with Vite as the static site and sends `/api/*` to a serverless function.
- `api/index.js` wraps the same Express app (`backend/src/app.js`) that runs locally. The app is exported without calling `listen()`, so one codebase works in both places.
- The root `package.json` lists the Express and CORS packages that Vercel bundles into the function.

Import the repo at vercel.com/new, keep the root directory as the repo root, and deploy.

## Stack

- **Backend:** Node.js and Express, with the data kept in memory and the code split into routes, services and data
- **Frontend:** React 19 with Vite and plain CSS, no UI library
