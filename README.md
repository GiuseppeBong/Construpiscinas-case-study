Construpiscinas — Business Management System

A custom management system built for a pool construction and equipment company in Aragua, Venezuela. This repository is a case study: the source code belongs to the client and stays private.

Stack: Next.js 15 · React 19 · TypeScript · Tailwind CSS 4 · Supabase (PostgreSQL) · Cloudflare Workers

The problem

The company builds swimming pools and sells pool equipment and chemicals. Before this system, it ran on paper: inventory quantities were written by hand on sheets taped to the warehouse wall, erased and rewritten as stock moved. There was no prior software of any kind — not a spreadsheet, not an off-the-shelf POS.

The owner had looked at SaaS options and turned them down. He did not want a monthly subscription for something this central to his business; he wanted a system that was his. He also could not absorb unpredictable hosting costs.

Those two constraints shaped every technical decision below.

What the system does

Inventory — Products, stock levels and movements, including items the business repackages from bulk (for example, a 100-litre drum of chlorine broken down into 25 gallons for resale).

Point of sale — A cashier station on a desktop machine. Each sale records what was sold and how it was paid, deducts stock automatically, and prints a small receipt on a thermal printer. The receipt exists as a control: it gives the customer proof and makes unrecorded sales visible.

Consignment suppliers — A large part of the inventory is held on consignment. For each supplier, the owner can see which products are in stock, what has sold, and how much is owed for what sold.

Construction projects — Pool builds, which carry the higher margin. A project can also draw pumps, machinery and store products from the same inventory, so those items deduct from the same stock. Projects track how much the customer has paid and optional payment milestones the owner sets when creating the project.

Sales commissions — Every sale is attributed to the employee who made it or to the owner. The finance view totals what each employee billed so commissions can be calculated as a percentage of that total.

Finance — Net profit and per-employee sales totals.

Payment records and receipts support uploading a photo of the proof of payment.

Technical decisions
Hosting the client would never pay for

The requirement was zero recurring cost. Supabase's free tier covers the database for a single-owner business with one cashier station, and the app started on Vercel's free tier. Both scale to paid plans only at volumes this business will not reach, which mattered more to the client than raw performance.

Moving the deployment to Cloudflare Workers

The app was built and shipped on Vercel first, then moved to Cloudflare Workers using OpenNext (@opennextjs/cloudflare with Wrangler). The project keeps the standard Next.js scripts alongside a separate set for the Workers build, preview and deploy, so both paths stay runnable.

The move is not free: the build goes through an adapter rather than the framework's native target, configuration lives in Wrangler, and environment values become Worker bindings. It is the kind of change that is easy to underestimate and worth doing while the app is small.

Stock that moves from two directions

Store sales and construction projects both consume the same inventory. Rather than maintaining separate stock counts, both paths write to one movement ledger, so a pump sold over the counter and a pump installed on a job site reduce the same number.

Consignment as a first-class concept

Consignment stock is physically in the warehouse but not owned by the business, so it cannot be treated as ordinary inventory. Items carry their consignor, and settlement is calculated from what actually sold rather than from what was received.

Access

The owner is the only full user. The cashier station signs in with a shared cashier account rather than individual logins — a deliberate simplification agreed with the client, since attribution of a sale to a seller is recorded on the sale itself, not derived from who is logged in.

Data model

The core is a products table plus an inventory movement ledger: every change in stock is a row there rather than a number edited in place, so a counter sale, a project consumption and a restock all read the same way, and the history stays reconstructable.

Sales are a header plus line items, with the seller recorded on the header — that is what the commission totals are built from. Suppliers carry a consignment relationship to the products they own, so settlement is computed from movements that actually sold, not from what was delivered.

Projects hold the customer, the agreed amount and optional payment milestones. Payments, both on sales and on projects, record method, amount and an optional image of the proof of payment.

How I worked

The project was built with Claude Code as part of the daily workflow, with a CLAUDE.md at the repository root documenting the stack, design decisions, the state of each module and known pitfalls — so that context survived between sessions. I read and understood generated code before committing it; the business rules here (consignment settlement, commission attribution, stock deduction across two sales paths) are the kind that look plausible and are wrong.

What I would do differently
Write tests for the inventory and commission rules from the start. Those are the parts where wrong behaviour stays invisible until money is missing.
Settle the full data model with the client before building module by module. Starting by modules meant reworking tables more than once.
Evaluate the hosting target at the beginning rather than moving the deployment after the app was already running.

Built for a real client, in production. The source code is private; this document describes the work.
