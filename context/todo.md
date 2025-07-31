# Cryptex MVP - Development Checklist

This document outlines the full development plan for the Cryptex MVP. Follow these steps sequentially to ensure a stable, incremental build process.

---

## Milestone 0: Project Setup & Foundation

*The goal of this milestone is to establish a fully functional local development environment and a CI/CD pipeline to Vercel.*

- [X] **1. Set up Local Appwrite Instance**
    - [X] Install Docker on your machine.
    - [X] Run the official Appwrite Docker command in your terminal.
      ```bash
      docker run -it --rm \
          --volume /var/run/docker.sock:/var/run/docker.sock \
          --volume "$(pwd)"/appwrite:/usr/src/code/appwrite:rw \
          --entrypoint="install" \
          appwrite/appwrite:latest
      ```
    - [ ] Navigate to `http://localhost` and create your root admin account.
    - [ ] Create a new Project named `Cryptex`.
    - [ ] Note down the **Project ID** and the **API Endpoint** (`http://localhost/v1`).

- [X] **2. Initialize Next.js Project**
    - [X] Run the `pnpm create` command.
      ```bash
      pnpm create next-app . --typescript --tailwind --eslint --app --src-dir N --import-alias "@/*"
      ```
    - [X] Install required dependencies.
      ```bash
      pnpm install appwrite lucide-react @hashgraph/sdk next-themes
      ```

- [X] **3. Initialize UI Framework (`shadcn/ui`)**
    - [X] Run the `shadcn/ui` init command.
      ```bash
      pnpm dlx shadcn-ui@latest init
      ```
    - [X] Select the following options when prompted:
        - Style: `Default`
        - Base color: `Slate`
        - Use CSS variables: `Yes`

- [X] **4. Configure Environment and Appwrite Client**
    - [X] Create a `.env.local` file in the project root.
      ```ini
      # Appwrite
      NEXT_PUBLIC_APPWRITE_ENDPOINT="http://localhost/v1"
      NEXT_PUBLIC_APPWRITE_PROJECT_ID="[Your-Project-ID]"
      APPWRITE_API_KEY="[Your-Appwrite-API-Key-From-Settings]" # Create a new server-side API Key

      # Hedera
      HEDERA_ACCOUNT_ID="[Your-Testnet-Account-ID]"
      HEDERA_PRIVATE_KEY="[Your-Testnet-Private-Key]"

      # AI Service
      REQUESTY_AI_API_KEY="[Your-Requesty-AI-Key]"
      ```
    - [X] Create a client configuration file at `lib/appwrite.ts`.
      ```typescript
      // lib/appwrite.ts
      import { Client, Account, Databases, Functions } from 'appwrite';

      const client = new Client();
      client
          .setEndpoint(process.env.NEXT_PUBLIC_APPWRITE_ENDPOINT!)
          .setProject(process.env.NEXT_PUBLIC_APPWRITE_PROJECT_ID!);

      export const account = new Account(client);
      export const databases = new Databases(client);
      export const functions = new Functions(client);
      ```

- [ ] **5. Setup Version Control and Deployment**
    - [ ] Initialize a Git repository: `git init && git add . && git commit -m "Initial commit"`.
    - [ ] Create a new project on [Vercel](https://vercel.com/new) and link it to your Git repository.
    - [ ] Add all environment variables from `.env.local` to the Vercel project settings.

---

## Milestone 1: Authentication & Core UI

*The goal is to enable user sign-up, login, logout, and protect the main dashboard area.*

- [ ] **1. Implement Theme Provider & Toggle**
    - [ ] Create `components/ThemeProvider.tsx` using `next-themes`.
    - [ ] Add `shadcn/ui` components: `button`, `dropdown-menu`.
    - [ ] Create `components/ThemeToggle.tsx` to switch between light and dark themes.
    - [ ] Wrap the root `app/layout.tsx` with the `ThemeProvider`.

- [ ] **2. Build Authentication UI**
    - [ ] Add `shadcn/ui` components: `card`, `input`, `button`, `label`, `separator`.
    - [ ] Create `components/auth/AuthForm.tsx`.
    - [ ] Structure: `Card` containing a "Sign in with Google" `Button`, a `Separator`, and an email `Input` with a "Sign in with Email" `Button`.

- [ ] **3. Implement Authentication Client Logic**
    - [ ] In `components/auth/AuthForm.tsx`:
        - `useState` for email input.
        - `handleGoogleAuth` function to call `account.createOAuth2Session('google', 'http://localhost:3000/dashboard')`.
        - `handleEmailAuth` function to call `account.createMagicURLSession('unique()', email, 'http://localhost:3000/dashboard')`.
    - [ ] In the Appwrite Console, go to **Auth > Settings** and enable both the **Email (Magic URL)** and **Google** providers. Add `http://localhost` to the allowed client hostnames.

- [ ] **4. Create Protected Routes & User Session Hook**
    - [ ] Create `app/dashboard/page.tsx` as the main protected page.
    - [ ] Create `hooks/useAuth.ts` that provides `{ user, loading }` by calling `account.get()` on mount.
    - [ ] In `app/dashboard/page.tsx`, use the `useAuth` hook to redirect to `/` if `!loading && !user`.
    - [ ] Create `components/auth/LogoutButton.tsx` that calls `account.deleteSession('current')` and redirects to `/`.
    - [ ] Render the `LogoutButton` on the dashboard page.

---

## Milestone 2: Company Profile & Onboarding

*The goal is to ensure every new user creates a company profile before they can use the app.*

- [ ] **1. Create `Companies` Collection in Appwrite**
    - [ ] In the Appwrite Console (**Database > Create Collection**), create `Companies`.
    - [ ] Add attributes:
      ```json
      [
        { "key": "company_name", "type": "string", "required": true },
        { "key": "role", "type": "string", "required": true },
        { "key": "description", "type": "string", "required": false },
        { "key": "country_of_operation", "type": "string", "required": true },
        { "key": "ctt_balance", "type": "integer", "required": true, "default": 0 }
      ]
      ```
    - [ ] Set permissions: Allow any authenticated user to create, but only the user who created it to read/update.

- [ ] **2. Build Onboarding UI & Logic**
    - [ ] Create `app/onboarding/page.tsx`.
    - [ ] Create `components/onboarding/CompanyForm.tsx`.
        - **Props:** `userId: string`, `onSubmitSuccess: () => void`.
        - **State:** `useState` for each form field (`company_name`, `role`, etc.).
    - [ ] The form should use `shadcn/ui` `Input`, `Select`, and `Textarea`.

- [ ] **3. Create Backend Function for Company Creation**
    - [ ] In Appwrite Console (**Functions > Create Function**), create `create-company-profile` using the Node.js runtime.
    - [ ] In the function's `src/index.js`:
        - Get user ID from `process.env.APPWRITE_FUNCTION_USER_ID`.
        - Get form data from `req.body`.
        - Use the `databases.createDocument` method to create a new company.
        - Set permissions on the new document to grant the creating user read/write access.
    - [ ] In `CompanyForm.tsx`, the `onSubmit` handler should call `functions.createExecution('create-company-profile', JSON.stringify(formData))`.

- [ ] **4. Implement Onboarding Middleware**
    - [ ] Modify `hooks/useAuth.ts` to also fetch the user's related `Companies` document. Return a new boolean `hasCompany`.
    - [ ] In `app/dashboard/page.tsx`, add a `useEffect` to redirect to `/onboarding` if `user && !hasCompany`.

---

## Milestone 3: Core Data & Feature Foundation

*The goal is to set up all remaining data models and build the UI to display them.*

- [ ] **1. Create Remaining Database Schemas in Appwrite**
    - [ ] **`Products`**: `product_name`, `sku`, `description`, `materials`, `unit`, `unit_price`, `country_of_origin`, relationship to `Companies` (brand_id), relationship to `Companies` (manufacturer_id).
    - [ ] **`Orders`**: `order_id`, relationships to `Companies` (buyer & seller), `order_date`, `expected_delivery_date`, `line_items` (JSON), `total_amount`, `status`, `initial_transaction_id`.
    - [ ] **`Digital_Product_Passports`**: `dpp_id`, relationship to `Orders`, `batch_number`, `quantity_in_batch`, `manufacturing_date`, `data_carrier_url`, `documents` (JSON).
    - [ ] **`Lifecycle_Events`**: relationship to `DPPs`, `step_name`, `status`, `target_date`, `completion_date`, `notes`, `hedera_transaction_id`.
    - [ ] **`Invoices`**: `invoice_id`, relationship to `Orders`, `issue_date`, `due_date`, `status`, `payment_transaction_id`.
    - [ ] **`Admin_Audit_Log`**: `admin_user_id`, `action`, `target_id`, `reason`, `timestamp`, `hedera_transaction_id`.

- [ ] **2. Build Main Dashboard Layout**
    - [ ] Create `components/layout/Sidebar.tsx` with `shadcn/ui` buttons as navigation links.
    - [ ] Create `app/dashboard/layout.tsx` that renders the `Sidebar` and the page `children`.

- [ ] **3. Build Order Display Component**
    - [ ] Add `shadcn/ui` `table` component.
    - [ ] Create `app/dashboard/orders/page.tsx`.
    - [ ] Create `components/orders/OrderList.tsx` as a **Server Component**.
        - It should fetch and display a list of orders for the current user's company using the Appwrite Node SDK.
        - **Props:** None.

---

## Milestone 4: AI & Hedera Integration

*The goal is to implement the platform's core differentiating features.*

- [ ] **1. Set up Hedera & AI Client Libraries**
    - [ ] Create `lib/hedera.ts`. It should export a helper function `submitToHCS(message: string)` that uses the SDK to submit a message to a pre-defined topic ID and returns the transaction ID.
    - [ ] Create `lib/ai.ts`. It should export a function `parseDocumentWithAI(fileText: string)` that calls the Requesty AI API and returns structured JSON.

- [ ] **2. Implement AI-Powered Order Creation Flow**
    - [ ] Create `app/dashboard/orders/new/page.tsx`.
    - [ ] Create `components/orders/CreateOrderForm.tsx` with a file input.
    - [ ] Create Appwrite Function `parse-po-document`:
        - Trigger: HTTP.
        - Logic: Takes a file ID, reads the file from Appwrite Storage, extracts text, calls `parseDocumentWithAI`, returns JSON.
    - [ ] Create Appwrite Function `create-order`:
        - Trigger: HTTP.
        - Logic: Takes parsed JSON, creates `Order` document, calls `submitToHCS`, updates the order with the `hedera_transaction_id`.
    - [ ] Wire the `CreateOrderForm` to call these functions sequentially.

- [ ] **3. Implement CTT Reward System**
    - [ ] Create Appwrite Function `calculate-ctt-reward`:
        - Trigger: Database Event (`databases.*.collections.Orders.documents.*.update`).
        - Logic:
            - Check if the `status` was changed to `Completed`.
            - Fetch related order details.
            - Execute the hybrid reward algorithm.
            - Use HTS to transfer CTT (for future use, for now just log it).
            - Update the `ctt_balance` on the seller's `Companies` document.