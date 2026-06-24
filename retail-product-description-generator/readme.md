## **Retail Product Description Generator — Bulk + Download**

This is a two-workflow system for generating marketing copy for ecommerce products at scale. The first workflow accepts a form submission with brand voice, tone preferences, and bulk product data (CSV/XLSX). It generates three types of marketing content per product using Google Gemini LLM — then stores results in a data table. The second workflow provides a webhook download endpoint that retrieves and exports all generated content as CSV.

### **System Architecture**

**Workflow 1: `retail-product-description-generator-bulk`**

Accepts user input via form, processes bulk product uploads, and generates descriptions.

**Workflow 2: `retail-product-description-download-webhook`**

Provides a GET endpoint that downloads generated content as CSV.

### **Workflow 1: Product Description Generator — Bulk**

#### **Components**

1. **Form Trigger (`On form submission`)**

HTML form collecting:
- Brand Voice (textarea) — how the brand speaks
- Tone (radio: Casual, Professional, Informative, Custom) — messaging style
- Custom Brand Tone (text field) — optional custom tone definition
- Bulk Upload (file) — CSV, XLS, or XLSX with product data

2. **File Type Router (`Switch`)**

Routes file uploads (CSV/XLSX) to appropriate extractor node.

3. **Data Extraction**

`Extract from CSV` and `Extract from XLSX` parse bulk uploads into structured records. Expects columns:
- `product_name`
- `product_specs`
- `product_category`

4. **Tone Resolution**

`Check for custom Tone` branches to either `Set Preset Tone` or `set Custom Tone`, ensuring consistent tone instruction passed to LLM.

5. **LLM Content Generation**

`Basic LLM Chain` prompts Google Gemini with brand voice, tone, and product details to generate:
- `pdp_description` (150–300 words optimized for SEO and mobile)
- `social_caption` (short, engaging social media copy)
- `meta_description` (search-engine-optimized snippet)

Includes **batching**: 10 products per batch with 1-second delay between batches to respect API limits.

6. **Output Parsing**

`Structured Output Parser` enforces JSON response shape with the three content fields.

7. **Data Table Storage**

`Insert row` writes generated content plus metadata (product name, tone applied) to the data table.

8. **Completion Form**

`Form` completion page displays success and provides a download link.

#### **Data Flow**

```
Form Submission
  ↓
File Type Check (CSV/XLSX)
  ↓
Parse Bulk Product Data
  ↓
Extract Fields (name, specs, category)
  ↓
Resolve Tone (preset or custom)
  ↓
Generate Content (Gemini LLM)
  ↓
Parse Structured Output
  ↓
Store in Data Table
  ↓
Show Completion Page
```

### **Workflow 2: Product Description Download Webhook**

#### **Components**

1. **Webhook Trigger (`On GET request`)**

Listens on path `/download-product-content`. Called by the "Download" link from the generator form.

2. **Data Retrieval (`Get row(s)`)**

Queries the data table for all generated content (disabled by default — enable if needed to filter).

3. **File Conversion (`Convert to File`)**

Converts data table rows to CSV binary format.

4. **Webhook Response (`Respond with attachment`)**

Returns CSV as downloadable attachment with filename `my_document_<YYYY-MM-DD>.csv`.

#### **Data Flow**

```
GET /download-product-content
  ↓
Fetch Generated Content from Data Table
  ↓
Convert to CSV
  ↓
Return as Attachment
```

### **Setup Instructions**

#### **Workflow 1 Setup**

1. **Create Data Table**

Create a data table named `retail-product-description-generator` with columns:
- `product_name` (string)
- `pdp_description` (string, long text)
- `social_caption` (string)
- `meta_description` (string)
- `tone` (string)

2. **Configure Google Gemini Credentials**

Open `Google Gemini Chat Model` node and set your Google Gemini (PaLM) API credentials.

3. **Validate Form Webhook**

Open `On form submission` to confirm form configuration and webhook ID.

4. **Test with Sample CSV**

Prepare a test CSV with headers: `product_name,product_specs,product_category` and at least one row.

#### **Workflow 2 Setup**

1. **Link Data Table**

Open `Get row(s)` node and point to the same data table (`retail-product-description-generator`).

2. **Enable Node** (if filtering needed)

By default `Get row(s)` is disabled. Enable if you need custom filtering before export.

3. **Validate Webhook Path**

Open `On GET request` and confirm path is `/download-product-content`.

4. **Activate Workflow**

Set workflow to active so webhook remains listening.

### **End-to-End Workflow**

1. **User submits form** with brand voice, tone, and bulk product file
2. **Workflow 1 processes** each product:
   - Extracts product name, specs, category
   - Applies tone instructions
   - Calls Gemini LLM to generate PDP description, social caption, meta description
   - Stores all output in data table
3. **Completion page displays** download link
4. **User clicks download** → triggers Workflow 2
5. **Workflow 2 retrieves** all generated content from data table and serves as CSV
6. **User receives** timestamped CSV file with all marketing copy

### **Key Features**

- **Bulk Processing**: CSV/XLSX upload supports batch generation
- **Batched API Calls**: Respects rate limits with 10-product batches and 1-second delays
- **Consistent Branding**: Brand voice and tone applied uniformly across all outputs
- **Flexible Tone**: Choose preset tones or define custom brand personality
- **Structured Output**: LLM response enforced to JSON schema, ensuring reliable parsing
- **Downloadable Export**: Generated content exported as CSV for use in ecommerce systems

### **Configuration Notes**

- **Batch Size**: Currently set to 10. Adjust in `Basic LLM Chain` if needed.
- **Batch Delay**: 1 second between batches. Increase for tighter API rate limits.
- **LLM Model**: Using `models/gemini-2.5-flash-lite`. Swap for `gemini-2.5-flash` if higher quality output is needed.
- **Max Retries**: LLM chain retries up to 4 times with 1.5-second delays on failure.

### **Troubleshooting**

- **File parsing fails**: Ensure CSV/XLSX headers match expected column names (`product_name`, `product_specs`, `product_category`).
- **LLM rate limit errors**: Increase batch delay or reduce batch size in `Basic LLM Chain`.
- **Download returns empty**: Ensure Workflow 1 has inserted rows into the data table before calling Workflow 2.
- **Custom tone not applied**: Check that custom tone value is non-empty in form submission.
