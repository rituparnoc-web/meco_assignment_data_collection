# Grocery Basket Data Collection Instructions

## Purpose

Collect the same basket snapshot from Blinkit, Zepto, and Instamart for:

- Bengaluru: Wipro Sarjapur / Hadosiddapura
- Chennai: Wipro Elcot SEZ, Karapakkam

Use one unit each of:

- A non-Coca-Cola-brand soft drink, 2.25 L (Pepsi 2.25 L is the preferred match when available)
- Modern whole wheat bread, preferably 400 g
- Bingo Original Style Chilli Sprinkled chips, preferably 85 g

Availability and pack sizes vary by platform and location. Record the exact title, size, and price displayed. If the preferred size is unavailable, use the closest same-product variant only when clearly identified, and record its actual size. Never present a different size as the requested size.

Never submit or place an order. If logged in, stop at the cart summary; do not press Place order, Proceed, Pay, or equivalent.

## CSV Schema

The current header in `basket_screenshots.csv` is:

```csv
bucket_id,platform,city,currency,item_name,item_size,item_price,quantity,item_line_total,items_subtotal,handling_fee,delivery_fee,partner_fee,rain_fee,cart_total,collected_at_approx
```

Use one row per cart item. Repeat the bucket ID, platform, location, currency, basket subtotal, fees, cart total, and collection time on each row in that basket.

- `bucket_id`: unique ID for a single platform/location capture, e.g. `BLR-WIPRO-SARJAPUR-INSTAMART-2026-10-16-1100`.
- `platform`: exactly `Blinkit`, `Zepto`, or `Instamart`.
- `city`: descriptive delivery location; use the selected address, not a guessed city center.
- `item_price`: actual current unit price shown in the cart, after discount.
- `item_line_total`: unit price multiplied by quantity.
- `items_subtotal`: cart's discounted item total.
- `handling_fee`, `delivery_fee`, `partner_fee`, `rain_fee`: record only the corresponding fee shown by the platform. Use `0` when explicitly waived or free; leave blank when not shown or unavailable. Do not infer a fee.
- `cart_total`: the displayed final amount to pay, before any order submission.
- `collected_at_approx`: local India time, rounded to the nearest minute, with `IST` and date.

If a platform does not show a fee breakdown without login, leave those fee and total cells blank. Do not copy fees from another platform, location, or earlier capture.

## Workflow

### 1. Prepare the capture

1. Open the existing `basket_screenshots.csv`; read its header and existing rows first.
2. Collect one platform and one location at a time. Recommended order: Blinkit Bengaluru, Blinkit Chennai, Zepto Bengaluru, Zepto Chennai, Instamart Bengaluru, Instamart Chennai.
3. Set the delivery location on the platform and verify the address shown in its header/cart. For Chennai, use the Wipro Elcot SEZ / Karapakkam result, not another Wipro location.
4. Start the approximate collection timestamp when the basket is inspected. Use IST.

### 2. Inspect the cart before searching

1. Open the cart before using the product search field.
2. If it already contains the three intended items at quantity 1 each, do not search or add duplicates. Read the item lines and bill summary directly.
3. If the cart is empty, search for and add the three intended products at quantity 1 each.
4. If it contains unrelated items, unexpected quantities, or a mixture that cannot be confidently separated, do not clear or alter it. Stop and ask the user how to proceed.
5. If changing location causes the cart to retain stale prices, refresh the cart and use the values shown for the selected location. Never reuse a prior location's prices.

### 3. Collect the basket details

For each cart, capture:

- Selected delivery address / city location
- Exact product title, pack size, quantity, and current price for every item
- Items subtotal
- Handling fee
- Delivery fee and/or delivery partner fee
- Rain or other mandatory fee if shown
- Final cart total / amount to pay
- Approximate collection time

Read prices from the cart where possible, not just search cards. Distinguish the discounted price from the MRP. Do not select optional tips, donations, coupons, or add-ons; if an optional amount is already selected, do not silently treat it as a mandatory fee.

### 4. Login and site errors

1. Do not ask the user to send credentials, OTPs, or passwords in chat. If sign-in is required, ask them to complete it directly in the browser.
2. Zepto may show item prices and subtotal while hiding exact fees behind a login prompt. Record verified item data, leave unavailable fees and cart total blank, and note the blocker to the user.
3. Instamart may require an address selection before showing local items. Use the visible location picker and verify the selected address. If the location picker or search stays inaccessible, stop; do not estimate prices.
4. If a site is stuck loading, rate-limited, or returns an API/CORS error, make at most one refresh or one alternate in-site navigation attempt. If it still fails, stop work on that site and location, preserve completed rows, and report what remains.

### 5. Append without modifying existing data

1. Never edit, replace, reorder, or delete existing rows. Existing rows are immutable historical captures.
2. Before appending, inspect the file's last row and retain the exact existing header.
3. Append new rows after the final row only. Do not rewrite the CSV from scratch.
4. Use a new unique `bucket_id` for every new platform/location capture. If the same platform/location/date was already collected, add a time suffix or sequence number (for example `-02`) rather than reusing or overwriting its rows.
5. Quote any CSV field containing commas, especially `city`; preserve the number and order of columns in the header.
6. If a platform/location cannot be collected, do not append placeholder item rows. Report the missing platform/location instead.

### 6. Verify after every basket

Before moving to the next location or platform:

1. Re-read the newly appended lines and confirm the header has not changed unexpectedly.
2. Confirm there are exactly three item rows for the bucket, unless a product was unavailable; in that case document the limitation and do not invent a row.
3. Confirm each item has the intended quantity and actual size/price from that location.
4. Check that the sum of `item_line_total` values equals `items_subtotal`.
5. Check that the displayed total reconciles with the item subtotal and mandatory charged fees. Allow platform rounding only when the displayed cart total confirms it.
6. Confirm all rows in the bucket share the same location, platform, bucket ID, and timestamp.
7. Only after verification, move to the next location/platform.

## Current Platform Notes

- Blinkit: the cart exposed item and fee details without proceeding to order submission.
- Zepto: product listings may load by category even when search fails. Its cart can show `Item Total` but may require login to show exact charges; do not infer fees.
- Instamart: the cart page can show exact item lines and a bill summary. Changing location may require returning to the home page, selecting the new address, and reopening the cart. Verify the location again before reading prices.

## Safety Stop

At every point, stop before order submission. A visible `Proceed` or `Login to Proceed` control is not permission to continue. Leave the cart unsubmitted and report any blocked fields clearly.
