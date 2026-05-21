# Error Handling and Workflow Validation

## Input Validation Behavior

To improve workflow reliability and prevent malformed data from propagating through downstream AI components, the ingestion workflow includes early validation checkpoints that intentionally stop execution when unsupported file types or invalid document structures are detected.

---

## Unsupported Image Files (PNG / Image Uploads)

The system is designed to process **text-based PDF study materials only**.

If a user uploads a PNG, JPG, or other image-based file instead of a supported PDF document:

- the workflow reaches the **Extract File From Binary** stage
- extraction fails because the file does not contain valid text content in the expected format
- the workflow stops immediately
- no cleaned text is generated or populates in the Airtable
- no chunking occurs in the Airtable
- no AI quiz generation is triggered
- no quiz records are created

This behavior is intentional to prevent invalid input from reaching downstream workflows.

---

## Unsupported PDF Structures

Some PDF files may technically be valid PDFs but still fail processing if they contain unsupported content structures, including:

- scanned/image-based PDFs
- PDFs containing screenshots instead of selectable text
- heavily formatted layouts
- complex headers or footers
- embedded images
- unusual document encoding
- non-standard formatting artifacts

When this occurs:

1. extraction may succeed partially
2. validation checks determine the content is unsuitable for downstream AI processing
3. the workflow routes to the **false branch of the Update Records validation check**
4. execution stops at the ingestion/integration boundary
5. the AI generation workflow is never triggered

This prevents malformed or incomplete text from being passed into question generation workflows.

---

## Design Rationale

This validation approach was intentionally implemented to enforce cleaner input handling and reduce downstream system failures.

Rather than allowing unsupported files to continue through the pipeline and create broken quiz outputs, the ingestion workflow acts as an early safeguard.

This ensures:

- cleaner AI inputs
- fewer workflow failures
- more reliable question generation
- better end-to-end system stability
- easier debugging during workflow validation

---

## Current Supported Input

Supported:
- text-based PDF files

Not currently supported:
- PNG files
- JPG files
- scanned PDFs
- image-based PDFs
- heavily formatted academic documents with non-text content

---

## Future Improvement Opportunity

Future iterations could expand support by integrating OCR (Optical Character Recognition), allowing the system to process scanned documents and image-based study materials rather than rejecting them at ingestion.
