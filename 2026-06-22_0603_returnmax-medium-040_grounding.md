**Task Link:** https://rl-gym-harness.turing.com/v2/tasks/d7f10c6c-688a-4917-a2c2-0d4d3a6d4f3d

## Original Prompt

I just realized I forgot to include $148.36 in bank interest from Chase Bank on my original return; it's on my 1099-INT. In Other Tax Situations start an amended return, check the 'Income' change category, and enter explanation: 'Omitted $148.36 in bank interest income from Chase Bank 1099-INT.' Click Continue to amendment. Then open the Print Center from Tax Tools and download the Federal Return PDF so I have the amended 1040-X ready to mail.

### Reason

Maya's active 2025 return already contains the Chase Bank 1099-INT for exactly $148.36, and the 2025 return has not been filed. Starting a 1040-X for an omitted item that is already present would produce no valid amendment.

**Category:** Existing data contradicts amendment

---


## Revised Prompt

I just realized I forgot to include $52.17 in bank interest from Wells Fargo on my original 2024 return; it's on my 1099-INT. Go to Tax Home, select 2024 under **Your tax return & documents**, then click **Amend this return**. Check the 'Income' change category, and enter explanation: 'Omitted $52.17 in bank interest income from Wells Fargo 1099-INT.' Click Continue and proceed through the income amendment step. When you reach the Review page, click Continue to Print Center. Then download the Federal Return PDF so I have the amended 1040-X ready to mail.

### Changes from original

1. **Entry point**: Changed from "In Other Tax Situations start an amended return" to "Go to Tax Home, select 2024 under Your tax return & documents, then click Amend this return" — the Other Tax Situations page is informational-only; the real entry point is the Amend button on the filed return view (`app/index/tto/returns/[year]/_client.tsx:589-599`).
2. **Filing state**: Changed from the active (unfiled) 2025 return to the already-filed 2024 return (`rm_filed_returns` store has a 2024 entry with status "accepted"), so the Amend button actually renders and the amendment flow is accessible.
3. **Income contradiction**: Changed payer from "Chase Bank" to "Wells Fargo" and amount from $148.36 to $52.17. The 2024 filed return snapshot already contains `{"type": "1099_int", "payer": "Chase Bank", "amount": 148.36}` — so a Chase Bank omission would still be contradicted by existing data. Wells Fargo does not appear anywhere in the snapshot's income sources, making the omission genuine.
4. **Flow detail**: Added explicit step to proceed through the income amendment form and reach the Review page before continuing to Print Center, matching the actual wizard flow (category selection → per-category forms → review → Print Center).