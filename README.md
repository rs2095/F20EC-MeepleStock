# F20EC-eCommerce-Technologies-MeepleStock

MeepleStock - Unattended RPA Inventory Replenishment Bot

Unattended RPA bot (UiPath) that automates nightly stock replenishment with dynamic reorder-point calculation, PDF purchase orders, and audit logging - paired with a cosine-similarity recommender system for product discovery.


Overview

MeepleStock is an unattended, scheduled RPA bot that monitors MeepleHaven's stock position nightly, computes a dynamic reorder point for every SKU, groups flagged items by supplier, generates PDF purchase orders, dispatches them via Outlook, and logs every decision for audit. Human attention is reserved only for exceptions.
It is the supply-side complement to a User-Based Collaborative Filtering recommender system: the recommender predicts which games customers will want; MeepleStock ensures those games are in stock when they arrive to buy them.


Features

-Nightly scheduled trigger at 02:00, plus a secondary event trigger on any write to inventory.xlsx for mid-day stock updates
-Dynamic reorder-point calculation per SKU: ReorderPoint = AvgDailySales × LeadTime × SafetyFactor
     Safety factor: 1.5 for fast-moving titles, 2.0 for slow-moving or long-lead titles
-Three-tier SKU classification: Critical (stock = 0), Warning (stock ≤ reorder point), Healthy
-Supplier consolidation: flagged SKUs grouped by supplier, reorder quantities rounded up to each supplier's minimum order multiple
-PDF purchase order generation: one PDF per flagged supplier, produced by filling a Word template and exporting via Save Document as PDF
-Outlook email dispatch with PDF attached
-Dual-channel exception escalation: Microsoft Teams (primary) -> Outlook email (fallback)
-Full audit trail: every classification and decision appended to audit_trail.xlsx
-Run summary dashboard: Critical/Warning/Healthy counts written to restock_dashboard.xlsx
-Healthy-run logging: on nights with no flagged SKUs, a "Healthy Run" entry is written and the bot terminates without a dashboard update


Project Structure

.
├── Main.xaml               # Main workflow - all bot logic lives here
├── project.json            # UiPath project metadata and package dependencies
├── project.uiproj          # UiPath project file
├── entry-points.json       # Defines bot entry points
└── README.md


Inputs

-inventory.xlsx (Excel) Current stock on hand per SKU
-sales_velocity.xlsx (Excel) Rolling 30-day sales per SKU
-suppliers.csv (CSV) Supplier email addresses and minimum order multiples
The bot merges these three sources into a single working view before processing.


Outputs

-PDF purchase orders (One per flagged supplier, emailed via Outlook)
-restock_dashboard.xlsx (Run summary with Critical / Warning / Healthy counts)
-audit_trail.xlsx (Per-SKU decision log including exceptions)
-Exception alerts (Teams alert (primary) or Outlook email (fallback))

Note: Generated PDF purchase orders are runtime artifacts and are not committed to this repository.


Tech Stack

UiPath Studio - workflow design and execution
UiPath Workbook Activities - Excel read/write (no Excel installation required on host)
Microsoft Outlook - purchase order email dispatch and exception fallback alerts
Microsoft Word + Save as PDF - purchase order document generation
Microsoft Teams (Workflows webhook) - primary exception escalation channel


