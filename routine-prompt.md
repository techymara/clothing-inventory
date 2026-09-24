# Weekly routine prompt

Copy the prompt below into a Claude Code routine (a scheduled task) that runs once a week and has the **Gmail** connector attached. Replace `YOUR_ARTIFACT_URL` with your own artifact link before saving it.

---

You are running my recurring weekly task for my wardrobe inventory. This is a fresh session with no memory of prior runs, so follow these steps exactly. Everything runs in the cloud through the ArtifactData and Artifact tools (load ArtifactData via ToolSearch with query "select:ArtifactData" if it isn't already available). No browser, laptop, or device connection is needed.

Target artifact: YOUR_ARTIFACT_URL

DATA MODEL
- Collection "items": one document per closet item, doc_id "item-NNNN" (zero-padded, 4 digits). Fields: itemDescription, category (Clothing / Shoes / Accessories / Handbags / Intimates), item (sub-type, e.g. Sneakers, Blazer), colors, size, brand, attire (Casual / Comfort / Business / Cold Weather / Formal / Sportswear, or ""), approxPrice, links, notes, notes2, addedAt, addedVia, imageUrl (optional, "/_blob/<32-hex asset id>"), status ("active" | "listed" | "gone"; missing means active), listedAt, listing {where, askingPrice}, goneAt (ISO timestamp), gone {reason: Sold/Donated/Gave away/Returned/Tossed, date: YYYY-MM-DD, salePrice (number), where}.
- Collection "pending": my review tray. You write here; I approve each entry on the page. NEVER write new purchases directly into "items" and never mark items gone yourself. The page does that when I tap Keep.

STEPS

1. Read everything: ArtifactData action "list" on "items" with query.limit 1000 (follow next_cursor if present), and "list" on "pending" with query.limit 1000.

2. New purchases. Search Gmail for order / shipping confirmations and receipts from roughly the past 8 days (e.g. "newer_than:8d (subject:order OR subject:shipped OR subject:confirmation OR subject:receipt)" and similar), across any clothing, shoe, accessory, handbag, or intimates retailer. Skip promotional or marketing emails; only real purchases count. For each purchased item that is NOT already in "items" (compare itemDescription/brand/colors) and NOT already in "pending", write a pending document:
   doc_id "p-" + current UTC timestamp digits + "-" + n (e.g. "p-20260927T151200Z-1"), data:
   {kind: "new", source: "Gmail", emailSubject: "<subject>", foundAt: "<ISO now>", data: {itemDescription, category, item, colors, size, brand, attire, approxPrice, links, notes: "Purchased via <retailer> email confirmation <date>", notes2: "", addedVia: "weekly-task"}}
   Leave fields as "" rather than guessing.

3. Resale sales. Search Gmail from the past 8 days for "your item sold" / sale notifications from resale sites (Poshmark, TheRealReal, Vestiaire Collective, Depop, eBay, Grailed, consignment shops, etc.). For each sale, find the matching item in "items" (prefer ones with status "listed"; match on brand + description + color). If it isn't already represented in "pending", write:
   doc_id "p-..." as above, data: {kind: "sold", source: "<site>", emailSubject, foundAt, targetId: "<matched item doc_id, or \"\" if no confident match>", summary: "<what sold, from the email>", gone: {reason: "Sold", date: "YYYY-MM-DD of the sale", salePrice: <number: what I earned if stated, else the sale price>, where: "<site>"}}
   Only set targetId when the match is confident; otherwise leave it "" (the page tells me to handle it by hand).

4. One-year cleanup. For every item with status "gone" whose goneAt is more than 365 days before now: if it has an imageUrl of the form "/_blob/<id>", delete that photo with the Artifact tool (action "delete", url = the artifact URL, path = the 32-hex id); then delete the document with ArtifactData action "delete" on "items". Never delete anything else, and never delete an item that isn't status "gone" or is gone less than 365 days.

5. Write all new pending documents in one ArtifactData "batch" call (op "set", collection "pending"). If there's nothing new and nothing to clean up, write nothing.

6. Never fabricate a purchase or sale that isn't clearly evidenced by an actual email. Never modify existing "items" documents except the step-4 deletions.

End with a short summary: how many new purchases and sales were added to the review tray (with their descriptions), and how many expired gone items were deleted. If any write fails, say so plainly.
