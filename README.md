# MSLI Integrated Management System

An **inventory management system** built for Montenegro Shipping Lines Inc. (MSLI) — a shipping enterprise operating a fleet of roughly **77 vessels** — to track and manage inventory across the operation from a single database-backed application.

**Tech stack:** VB.NET · MySQL

---

## Overview

MSLI Integrated Management System is a Windows desktop application that centralizes inventory record-keeping for a large shipping operation — replacing manual, spreadsheet-based tracking with a structured, database-backed tool. It was designed and built end-to-end, from the database schema through the user-facing application, to handle the data volume and operational complexity of a fleet-scale business.

> [Optional: add 1–2 sentences on the business context — e.g., which parts of the operation it supported and roughly when it was built.]

## Features

- **Inventory / item tracking** — record, search, and update items with full history via a dedicated `item_tracking` table.
- **Fleet-aware records** — inventory organized across vessels and company units. *[confirm/adjust]*
- **Reporting** — generate operational inventory reports and summaries. *[confirm/adjust]*
- **User management** — role-based access for staff. *[confirm/adjust]*
- **Data integrity** — relational schema with validation to keep records consistent.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Application / UI | VB.NET (.NET Framework) |
| Database | MySQL |
| Platform | Windows desktop |

## Project Structure

```
MSLI-Integrated-Management-System/
├── src/            # Application source (forms, modules)
├── database/       # Schema and SQL scripts
├── docs/           # Documentation / notes
└── README.md
```
> [Adjust to match your actual folder layout.]

## Getting Started

### Prerequisites
- Visual Studio (with VB.NET / .NET Framework) [specify version]
- MySQL Server [version]

### Setup
1. Clone the repository:
   ```bash
   git clone https://github.com/ryanclaret/MSLI-Integrated-Management-System.git
   ```
2. Create the database and import the schema from `database/` into MySQL.
3. Update the database connection settings in [config file / connection string location].
4. Open the solution in Visual Studio and build/run.

## Database

The core of the system is the `item_tracking` table, which records each inventory item and its movement/status over time, organized across the fleet's vessels and units. [Add a short note on other key tables — vessels/departments, users, etc.]

## Screenshots

> [Optional but recommended: add 1–2 screenshots of the main screens — they make the repo far more engaging. Drop images in `docs/` and link them here.]

## Status

[e.g., "Functional / in active use" or "Independent project built for a shipping-industry client — maintained as a portfolio piece."]

## Author

**Ryan Claret** — Mainframe Software Engineer (COBOL · CICS · DB2), 19+ years of experience, who also builds full-stack database applications with VB.NET and MySQL.

- LinkedIn: [linkedin.com/in/ryanclaret](https://www.linkedin.com/in/ryanclaret)

## License

[Optional — e.g., MIT, or "All rights reserved." Add a LICENSE file if you want to make reuse terms explicit.]
