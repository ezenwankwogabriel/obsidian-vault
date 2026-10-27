## Why not index every column

Here is the simple, straightforward explanation:

Adding an index to a column is like adding an **alphabetical index at the back of a book**. It makes finding specific pages fast, but every time you add or edit a page, you have to update the index too.

If you put an index on every single column, three major problems happen:

1. **Slower Writes (`INSERT` / `UPDATE` / `DELETE`):**
Every single time you save or change a row, the database has to update every single index tree one by one. A single insert turns into dozens of background write operations.
2. **Wasted Memory and Disk Space:**
Indexes take up huge amounts of storage—often larger than the actual table itself. They fill up your server's RAM, leaving less room to cache actual data.
3. **Multi-Column Queries Still Run Slow:**
Real-world searches usually filter on multiple fields at once (e.g., `WHERE status = 'active' AND date > 'today'`). Two individual single-column indexes don't help much here—a single combined (composite) index is what's actually needed.

---

### The Golden Rule

* **Index:** Columns you frequently search, filter (`WHERE`), or join on (`JOIN`).
* **Don't Index:** Columns you rarely search, columns with only a few values (like `Yes/No` or `Gender`), or tables that are constantly being written to.