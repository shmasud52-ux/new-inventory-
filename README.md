# Luggage Warehouse Inventory — Simple Liquid Glass

An offline-first inventory PWA intended for Android and Acode. Data is stored locally in IndexedDB; no server or account is required.

## Main features
- One profile per Model Number (case-insensitive duplicate prevention).
- Add brand, category, any number of custom materials, colors, and sizes with `+ Add`.
- Each Material × Color × Size combination gets its own stock record.
- Separate Stock In / Stock Out page; stock out cannot make quantities negative.
- Model profile view shows every variant and its current quantity.
- Stock history with Undo; Undo writes a reversal record instead of silently deleting history.
- CSV import for many models and variants. Recommended headers: `Model,Brand,Category,Material,Color,Size,Stock,Low,Location,Barcode`.
- Versioned JSON backup and validated restore with Merge New Records or Replace All.
- Offline PWA shell and local IndexedDB persistence.
- Existing LuggageFlow v1 records are migrated from `products` to model profiles/variants on first launch.

## CSV example
```csv
Model,Brand,Category,Material,Color,Size,Stock,Low,Location,Barcode
202,Leather Store,Luggage,Fabric,Light Gray,20,0,5,A-01,
202,Leather Store,Luggage,Fabric,Light Gray,24,0,5,A-01,
202,Leather Store,Luggage,Fabric,Black,20,12,5,A-01,
217,Leather Store,Luggage,Fabric,Navy,28,0,5,B-02,
```

A model may appear on multiple CSV rows; rows are grouped into one profile. Existing model numbers are skipped and reported as duplicates. Each profile creates all unique combinations of the materials, colors, and sizes in its rows; any combination not explicitly listed in CSV begins at zero stock.

## JSON backup schema (version 2)
Top-level keys: `format`, `schemaVersion`, `appVersion`, `exportedAt`, `data`.

`data` contains:
- `profiles`: unique model profile records with brand, model, category, material/color/size option lists.
- `variants`: individual stock records with `id`, `profileId`, `material`, `color`, `size`, `stock`, `low`, `location`, `barcode`, optional `photo` and `note`, timestamps.
- `stockHistory`: every movement with unique ID, variant ID, type, delta, before/after quantities, reason/note, timestamp and reversal references.
- `settings`: low-stock defaults, default location, theme and preferences.
- `identifiers`: app-generated identifier metadata.
- `metadata`: backup/integrity metadata.

Restore validates IDs, profile references, required fields, non-negative whole-number stock, and duplicate model numbers before enabling restore. Merge adds missing profiles/variants and leaves matching current variants unchanged. Replace All overwrites the local dataset only after a confirmation prompt.

## Install / run
1. Extract the ZIP.
2. Upload the contents of `luggageflow-final` to a static HTTPS host (e.g. GitHub Pages), or open locally for basic testing.
3. For PWA install, offline service worker, and camera-related browser permissions, use HTTPS. Stock operations and local database work offline after the app has been loaded.
4. Download JSON backups regularly. Clearing browser storage can erase local data.
