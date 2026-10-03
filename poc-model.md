# Topaz E&I Inventory Movement POC

## Context

Topaz E&I works in Singapore MRT electrical construction environments where site activity is governed by work permits, method statements, safety procedures, supervisors, inspection/testing records, and movement across store, site, vehicle, and worker custody.

The first POC should not try to become a full ERP. It should solve one narrow problem: every tool, consumable, and test instrument movement must be faster to log than to skip.

## First Workflow

1. Worker scans item QR/barcode.
2. Worker taps the action that applies to the scanned item: `TAKE` or `RETURN` for returnable tools/equipment; `USE` for consumables.
3. App defaults quantity, person, project, item type, and location.
4. Worker adds photo only when required.
5. CEO/admin sees movement history, current stock, low stock, and unreturned items.

For the first POC, the worker enters their name/team once and the browser remembers it on that phone. For production, this should become a real login, PIN, QR badge, or WhatsApp/phone-number identity mapped to a `Worker` record so the audit trail cannot be changed casually.

## Item Types

| Type | Examples | Tracking Rule |
| --- | --- | --- |
| Asset/tool | drill, crimper, ladder, cable puller | `TAKE` puts item under worker custody; `RETURN` releases custody to a location |
| Test equipment | insulation tester, clamp meter, multimeter | Same as asset/tool plus calibration date |
| Consumable | lugs, ferrules, glands, conduits | Quantity goes down when used or moved |
| Controlled item | expensive tool, calibrated equipment, restricted material | Photo or supervisor approval on return/issue |

## Core Data Model

### Item

- `id`
- `code`
- `name`
- `type`: `asset`, `test_equipment`, `consumable`, `controlled`
- `category`
- `unit`
- `reorder_level`
- `calibration_due_at`
- `serial_number`
- `photo_url`
- `active`

### Location

- `id`
- `name`
- `type`: `store`, `vehicle`, `site_zone`, `worker`, `supplier`, `scrap`
- `project_id`
- `active`

### Worker

- `id`
- `name`
- `role`
- `team`
- `phone`
- `pin_or_badge_id`
- `active`

### Project / Work Package

- `id`
- `name`
- `client`
- `site`
- `permit_ref`
- `work_order_ref`
- `status`

### Movement

- `id`
- `item_id`
- `action`: `take`, `return`, `use`, `move`, `adjust`, `scrap`
- `quantity`
- `stock_location_id`
- `destination_location_id`
- `worker_id`
- `worker_display_name`
- `project_id`
- `permit_ref`
- `work_order_ref`
- `remarks`
- `created_at`
- `created_by`
- `photo_required`
- `photo_url`
- `approval_status`

### Stock Balance

This can be calculated from movement history, then cached for speed.

- `item_id`
- `location_id`
- `quantity`
- `last_movement_at`

## Rules For The POC

- Returnable tools and test equipment cannot be `USE`d; they must be `TAKE`n or `RETURN`ed.
- When a returnable item is `TAKE`n, stock moves to `Out` and remains assigned to that worker until `RETURN`.
- When a returnable item is `RETURN`ed, stock moves from `Out` to the selected return location.
- Consumables are `USE`d and do not require return.
- Location-to-location transfer can be an admin/inventory function, not part of the worker quick log.
- Test equipment shows calibration warning when due within 30 days.
- High-value items remain in `Pending Return` until returned.
- Photo is required when returning damaged items or adjusting stock.
- CEO can export movement history to CSV.

## Recommended Implementation Path

Phase 1 should be custom and tiny: one mobile-first web form, one admin dashboard, QR labels, and CSV export.

Phase 2 can either stay custom or integrate with an open-source backend:

- Snipe-IT if tools/assets dominate.
- InvenTree if consumable stock and parts dominate.
- ERPNext only if Topaz wants purchasing, accounting, stock, and project workflows in one system.

## Hosting Recommendation

For the first field pilot, cloud hosting in Singapore is cleaner than a local PC because workers use phones and the CEO may need remote dashboard access.

Local hosting is acceptable only for a very short proof of concept on office Wi-Fi. If local hosting is used, expose it through VPN or a secure tunnel and back it up daily.
