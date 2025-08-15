### Splunk SPL Cheatsheet

---
#### 1. Syntax Rules

* **Piping (`|`)**: The pipe symbol passes the output of one command as the input to the next command. This allows for chained operations.
    * Example: `... | stats count by status | sort -count`
* **Keywords**: Commands and keywords are not case-sensitive (e.g., `stats` is the same as `STATS`).
* **Field Names**: Field names are case-sensitive (e.g., `clientIP` is different from `clientip`).
* **Quotation Marks**: Use double quotes (`" "`) for values that contain spaces or special characters.
    * Example: `sourcetype="access log"`
* **Comments**: Use `//` or `| comment` to add comments to your search.
    * Example: `... | stats count by user // This counts user logins`

#### 2. Basic Search Commands

* **Search**: Find events.
    * `sourcetype=access_combined status=200`
    * `index=web_logs "error"`
* **Time Range**: Specify a time window for your search.
    * `earliest=-1h` (last hour)
    * `earliest=10/01/2025:00:00:00 latest=10/01/2025:23:59:59` (specific day)

---

#### 3. Formatting and Aggregation Commands

#### `table`
Use **table** to display search results in a tabular format. It's great for quickly viewing specific fields.

**Syntax**: `| table <field1> <field2> ...`

**Example**: `sourcetype=web_server | table _time, clientip, status, action`

#### `stats`
Use **stats** to calculate statistics on your data. This is a powerful command for summarization and aggregation.

**Syntax**: `| stats <function>(<field>) [as <new_field_name>] by <grouping_field>`

**Functions**:
* `count`
* `sum`
* `avg`
* `min`
* `max`
* `stdev` (standard deviation)
* `values`
* `dc` (distinct count)

**Examples**:
* `sourcetype=web_server | stats count by clientip` (Count events by client IP)
* `sourcetype=web_server | stats avg(bytes) as average_bytes by status` (Calculate average bytes by status code)

#### `rename`
Use **rename** to change a field's name.

**Syntax**: `| rename <old_field> as <new_field>`

**Example**: `sourcetype=db_logs | rename db_user as user`

#### `fields`
Use **fields** to either keep or remove fields from your results.

**Syntax**:
* `| fields <field1>, <field2>` (Keep only specified fields)
* `| fields - <field3>` (Remove a specific field)

**Example**: `index=sales | fields product_id, price, quantity`

#### `sort`
Use **sort** to order your results.

**Syntax**: `| sort <field>` or `| sort -<field>` (for descending order)

**Example**: `sourcetype=transactions | sort -amount` (Sort transactions from highest to lowest amount)

---

#### 4. Modifying and Creating Fields

#### `eval`
Use **eval** to create new fields or modify existing ones using an expression.

**Syntax**: `| eval <new_field> = <expression>`

**Expressions**:
* **Arithmetic**: `+`, `-`, `*`, `/`
* **String**: `len()`, `lower()`, `upper()`, `substr()`
* **Conditional**: `if()`, `case()`

**Examples**:
* `sourcetype=sales | eval total_price = quantity * price`
* `sourcetype=logs | eval status_category = if(status >= 500, "Error", "OK")`

---

#### 5. Joining Data

#### `join`
Use **join** to combine events from different searches based on a common field. `join` is generally resource-intensive, so use it sparingly.

**Syntax**: `| join <field> [subsearch]`

**Example**:
`index=sales | join customer_id [search index=customers | table customer_id, customer_name]`

---

#### 6. Advanced Commands

#### `dedup`
Use **dedup** to remove duplicate events based on one or more fields.

**Syntax**: `| dedup <field1> <field2> ...`

**Example**: `sourcetype=auth_logs | dedup user` (Show only the first event for each user)

#### `top` and `rare`
Use **top** to find the most common values and **rare** for the least common values of a field.

**Syntax**: `| top <field>`

**Example**: `sourcetype=web_server | top clientip`

#### `streamstats`
Use **streamstats** to calculate statistics on an event-by-event basis. It's useful for real-time aggregation and trend analysis.

**Syntax**: `| streamstats <function>(<field>) [as <new_field>] by <grouping_field>`

**Example**: `index=transactions | streamstats sum(amount) as running_total by user` (Calculate a running total of purchases per user)

---

