

## **Cryptex: MVP Specification Document**

**Version:** 1.0
**Date:** July 31, 2025
**Project Goal:** To build a feature-complete Minimum Viable Product (MVP) of the Cryptex platform. The primary purpose of this MVP is to serve as a high-fidelity demo for potential users, partners, and investors, showcasing the platform's unique value proposition.

### 1. Project Overview & Motivation

Cryptex is a modern, AI and Hashgraph-enabled ERP platform designed to bring unprecedented transparency, efficiency, and trust to the textile supply chain. The platform aims to connect Suppliers, Manufacturers, Merchandisers, and Brands in a single, unified system.

The core motivation is to solve key industry pain points: opaque supply chains, cumbersome documentation processes, and a lack of verifiable trust between parties. By leveraging AI for automation and the Hedera Hashgraph for immutable transaction logging, Cryptex will provide a single source of truth for the entire product lifecycle.

### 2. Core User Roles

*   **Supplier:** Provides raw materials to Manufacturers.
*   **Manufacturer:** Assembles raw materials into finished goods.
*   **Merchandiser:** Acts as a project manager and communication bridge between Manufacturers and Brands.
*   **Brand:** Designs products and sells them to the market.
*   **Global Admin:** A trusted platform administrator with limited, specific powers to ensure platform integrity.

### 3. The Core MVP User Flow (The "Happy Path" Demo)

This is the central narrative the MVP must support end-to-end:

1.  **Onboarding:** A new Manufacturer signs up with their email or Google account and sets up their company profile.
2.  **Order Creation:** A Merchandiser, acting on behalf of a Brand, uploads a Purchase Order (PO) PDF into the Cryptex chat interface. The AI Agent parses the document, extracts the key details (product, quantity, delivery date), and pre-fills an order form. The Merchandiser verifies and confirms the order.
3.  **Hedera Record:** The creation of this order is submitted to the Hedera Consensus Service (HCS), generating a permanent, verifiable transaction ID which is linked to the order.
4.  **Production Tracking:** The Manufacturer views the order and uses the built-in **TNA (Time and Action) Calendar** to track production steps (e.g., "Fabric Cutting," "Stitching," "Quality Control"). Each completed step is a `Lifecycle_Event`.
5.  **Shipment & Documentation:** The Manufacturer creates a **Digital Product Passport (DPP)** for the shipment. They upload required documents (e.g., Bill of Lading, Commercial Invoice, Transaction Certificate) into the DPP's **Compliance Binder**.
6.  **Verification:** The Brand receives the shipment, verifies the contents against the DPP, and confirms receipt. This action is also logged on Hedera.
7.  **Trust Score Update:** Upon successful completion of the order and a positive review from the Brand, the Manufacturer's **Cryptex Trust Token (CTT)** balance is increased according to the reward algorithm. This publicly visible score enhances their reputation on the platform.

### 4. Technical Architecture & Stack

*   **Package Manager:** `pnpm`
*   **Frontend:**
    *   **Framework:** Next.js
    *   **Hosting:** Vercel
    *   **UI Components:** `shadcn/ui` (light and dark mode support required)
    *   **Data Grids:** AG Grid (Community Version) for the TNA Calendar.
    *   **Gantt Charts:** Frappe Gantt for the TNA timeline view.
    *   **Maps:** Leaflet.js with OpenStreetMap for map visualizations.
*   **Backend:**
    *   **Platform:** Appwrite (self-hosted or via a free tier that doesn't require a credit card).
    *   **Services:** Appwrite Auth, Database, Functions, and Storage.
*   **Blockchain:**
    *   **Ledger:** Hedera Hashgraph (Testnet for the MVP).
    *   **Services:** Hedera Consensus Service (HCS) for logging and Hedera Token Service (HTS) for the CTT.
*   **Artificial Intelligence:**
    *   **LLM Provider:** Requesty AI API.
    *   **Document Parsing:** A suitable open-source library for PDF text extraction (e.g., `pdf-parse` for Node.js).

### 5. Backend Specification (Appwrite)

#### 5.1. Database Collections (Schemas)

**`Companies`**
*   `company_name` (String, Required)
*   `role` (String, Required, Enum: 'Supplier', 'Manufacturer', 'Merchandiser', 'Brand')
*   `description` (String, Optional)
*   `country_of_operation` (String, Required)
*   `ctt_balance` (Number, Default: 0) - *The Cryptex Trust Token score*

**`Products`**
*   `product_name` (String, Required)
*   `sku` (String, Required, Unique)
*   `description` (String, Optional)
*   `materials` (String, Required)
*   `unit` (String, Required, Enum: 'pcs', 'kg', 'm')
*   `unit_price` (Number, Required)
*   `country_of_origin` (String, Required)
*   `brand_id` (Relationship to `Companies`, Required)
*   `manufacturer_id` (Relationship to `Companies`, Required)

**`Orders`**
*   `order_id` (String, Required, Unique)
*   `buyer_id` (Relationship to `Companies`, Required)
*   `seller_id` (Relationship to `Companies`, Required)
*   `order_date` (Datetime, Required)
*   `expected_delivery_date` (Datetime, Required)
*   `line_items` (JSON, Required) - `[{product_id, quantity, unit_price}]`
*   `total_amount` (Number, Required)
*   `status` (String, Required, Enum: 'Pending', 'Confirmed', 'In Production', 'Shipped', 'Completed')
*   `initial_transaction_id` (String, Optional) - *Stores the Hedera transaction ID for order creation.*

**`Digital_Product_Passports` (DPP)**
*   `dpp_id` (String, Required, Unique)
*   `order_id` (Relationship to `Orders`, Required)
*   `batch_number` (String, Required)
*   `quantity_in_batch` (Number, Required)
*   `manufacturing_date` (Datetime, Required)
*   `data_carrier_url` (String, Optional) - *URL for the product's QR code.*
*   `documents` (JSON, Optional) - *Array of objects for the Compliance Binder: `[{doc_name, doc_url, status: 'Valid' | 'Invalid'}]`*

**`Lifecycle_Events`**
*   `dpp_id` (Relationship to `Digital_Product_Passports`, Required)
*   `step_name` (String, Required, e.g., 'Stitching')
*   `status` (String, Required, Enum: 'Pending', 'In Progress', 'Completed')
*   `target_date` (Datetime, Required)
*   `completion_date` (Datetime, Optional)
*   `notes` (String, Optional)
*   `hedera_transaction_id` (String, Optional)

**`Invoices`**
*   `invoice_id` (String, Required, Unique)
*   `order_id` (Relationship to `Orders`, Required)
*   `issue_date` (Datetime, Required)
*   `due_date` (Datetime, Required)
*   `status` (String, Required, Enum: 'Unpaid', 'Paid', 'Pending', 'Overdue')
*   `payment_transaction_id` (String, Optional)

**`Admin_Audit_Log`**
*   `admin_user_id` (String, Required)
*   `action` (String, Required, e.g., 'SUSPEND_USER', 'FLAG_TRANSACTION')
*   `target_id` (String, Required, e.g., the user ID or order ID)
*   `reason` (String, Required)
*   `timestamp` (Datetime, Required)
*   `hedera_transaction_id` (String, Required)

#### 5.2. Appwrite Functions (Serverless Logic)

*   **`parse-po-document`**:
    *   **Trigger:** HTTP call from the frontend with a file.
    *   **Action:** Extracts text from the PDF, sends it with the `Orders` schema (in MCP format) to the Requesty AI API, returns the structured JSON.
*   **`submit-to-hcs`**:
    *   **Trigger:** Called by other functions (e.g., after an order is created).
    *   **Action:** Takes a JSON payload, submits it to the Cryptex HCS Topic, and returns the resulting `transactionId`.
*   **`calculate-ctt-reward`**:
    *   **Trigger:** Database event when an `Order` status changes to 'Completed'.
    *   **Action:** Executes the Hybrid Performance algorithm. Uses the Hedera SDK to transfer the calculated CTT amount from the platform treasury to the user's wallet via HTS. Updates the `ctt_balance` in the `Companies` collection.
*   **`generate-invoice-pdf`**:
    *   **Trigger:** HTTP call from the frontend.
    *   **Action:** Fetches invoice data, renders it into an HTML template, uses Puppeteer to convert to PDF, and returns the file.

#### 5.3. Authentication

*   **Providers:** Configure Appwrite to allow **Google OAuth** and **Magic Link (Passwordless) Email** authentication.
*   **Onboarding Flow:** A two-step process. First, create the user account. Second, force a redirect to a "Company Setup" form, which must be completed before accessing the dashboard.

### 6. Frontend Specification

#### 6.1. Naming & Structure

*   **Component Naming:** PascalCase (e.g., `OrderCard.tsx`).
*   **Directory Structure:** Use the Next.js `app` directory convention for routing. Create a `/components` directory for reusable components and a `/lib` directory for helper functions.

#### 6.2. Key Reusable Components (with Props)

*   **`OrderForm`**:
    *   `props`: `onSubmit` (function), `initialData` (object, optional for editing).
*   **`TNAGrid`**:
    *   `props`: `dppId` (string), `events` (array of `Lifecycle_Event` objects).
    *   **Functionality:** Renders the TNA steps in an AG Grid. Allows for in-line editing of status and dates, which should trigger backend updates.
*   **`LifecycleTimeline`**:
    *   `props`: `events` (array of `Lifecycle_Event` objects).
    *   **Functionality:** Renders a visual Gantt chart of the TNA using Frappe Gantt.
*   **`DocumentUploader`**:
    *   `props`: `dppId` (string), `onUploadComplete` (function).
    *   **Functionality:** A component for uploading files to the DPP's Compliance Binder.
*   **`ToastNotification`**:
    *   **Functionality:** Use `shadcn/ui`'s Sonner/Toaster to display non-blocking notifications for success (`Order Created!`) or error messages.

### 7. Global Admin Functionality

*   An "Admin Controls" section will be visible in the UI only to users in the `CryptexAdmins` team.
*   This section will provide the simple, specified tools: Suspend User, Flag Transaction, and Flag Document. All actions must be logged to the `Admin_Audit_Log`.

### 8. Error Handling Strategy

*   **Frontend:** All API calls to Appwrite functions must be wrapped in `try...catch` blocks. User-facing errors should be displayed using the `ToastNotification` component (e.g., "Failed to parse document. Please try again."). Never expose raw technical errors to the user.
*   **Backend:** Appwrite Functions must include error handling. On failure, they should log the detailed error within Appwrite's logging system and return a clear HTTP error status (e.g., 400 for bad input, 500 for an internal server error).

### 9. Testing Plan

*   **Unit Tests:** Use a framework like Jest to test critical business logic functions (e.g., a local mock of the CTT calculation).
*   **Integration Tests:** The primary focus should be here. Verify that frontend components correctly fetch data from and send data to the Appwrite backend.
*   **End-to-End (E2E) Testing:** Manually test the full "Happy Path" user flow (Section 3) on a regular basis. Ensure the flow is smooth and all integrations (AI, Hedera) are working as expected. Test the registration flow for both Google and email. Test at least one admin action.