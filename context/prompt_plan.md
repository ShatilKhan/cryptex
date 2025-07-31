
### **Developer Blueprint: Cryptex MVP**

#### **Milestone 0: Project Setup & Foundation**
The goal here is to establish the entire development environment, from local services to the cloud hosting pipeline.

*   **Step 0.1: Set up Appwrite Locally.** Use Docker to run a local Appwrite instance for development. This provides a clean, isolated environment.
*   **Step 0.2: Initialize Next.js Project.** Create the Next.js application using `pnpm`, the designated package manager.
*   **Step 0.3: Initialize UI Framework.** Integrate `shadcn/ui` into the project and configure it for light/dark modes.
*   **Step 0.4: Connect App to Appwrite.** Create a centralized Appwrite client library to manage the connection and SDK instances.
*   **Step 0.5: Set up Version Control & Deployment.** Initialize a Git repository and connect it to a new Vercel project for continuous deployment.

#### **Milestone 1: Authentication & Core UI**
The goal is to allow users to sign up, log in, and see a protected dashboard area. This completes the basic user session loop.

*   **Step 1.1: Create the Login UI.** Build the component for the login/signup page.
*   **Step 1.2: Implement Authentication Logic.** Wire the UI to the Appwrite authentication service (Google & Magic Link).
*   **Step 1.3: Create a Protected Dashboard Page.** Build a simple dashboard page that is only accessible to logged-in users.
*   **Step 1.4: Implement Logout Functionality.** Add a button to clear the user's session and redirect them to the login page.

#### **Milestone 2: Company Profile & Onboarding**
The goal is to ensure every new user is associated with a company, fulfilling the mandatory onboarding flow.

*   **Step 2.1: Create Company Database Schema.** Manually create the `Companies` collection and its attributes in the Appwrite console.
*   **Step 2.2: Implement Onboarding Middleware.** Create logic in Next.js that checks if a logged-in user has an associated company. If not, it forces a redirect to a company setup page.
*   **Step 2.3: Build the Company Setup UI.** Create the form for a user to input their company details.
*   **Step 2.4: Create the Company Setup Backend Function.** Write an Appwrite Function that takes the form data, creates a new `Company` document, and links it to the user.

#### **Milestone 3: Core Data & Feature Foundation**
The goal is to lay the groundwork for the main features by setting up the remaining data models and creating the UI to display them.

*   **Step 3.1: Create Remaining Database Schemas.** Manually create the `Products`, `Orders`, `DPPs`, `Lifecycle_Events`, and `Invoices` collections in Appwrite.
*   **Step 3.2: Build the Main Dashboard Layout.** Create the primary UI shell with navigation sidebar and a main content area.
*   **Step 3.3: Build the Order Display Component.** Create a server component that fetches and displays a list of orders for the logged-in user's company.

#### **Milestone 4: AI & Hedera Integration**
The goal is to integrate the external services that make Cryptex unique.

*   **Step 4.1: Set up Hedera & AI Clients.** Create centralized library files for the Hedera and Requesty AI SDKs, managing API keys via environment variables.
*   **Step 4.2: Implement AI-Powered Order Creation.**
    *   Build the UI component with a file uploader.
    *   Create the `parse-po-document` Appwrite Function to handle the AI parsing.
    *   Create the `create-order` Appwrite Function that takes the parsed data, creates the `Order` document, and submits the creation event to Hedera.
    *   Connect the frontend UI to trigger this flow.
*   **Step 4.3: Implement the CTT Reward System.**
    *   Create the `calculate-ctt-reward` Appwrite Function containing the hybrid reward algorithm.
    *   Configure a database trigger in the Appwrite console to execute this function whenever an `Order`'s status is updated to "Completed".

***

### **LLM Prompts for Implementation**

Here are the detailed, sequential prompts to generate the code for the Cryptex MVP.

---

#### **Prompt 1: Project Initialization**

```text
We are starting a new project called "Cryptex".

1.  Initialize a new Next.js project in the current directory using `pnpm`. Use TypeScript, Tailwind CSS, and the App Router.
2.  Once initialized, use `pnpm` to install the following dependencies: `appwrite`, `lucide-react`, and `@hashgraph/sdk`.
3.  Initialize `shadcn/ui` in the project. When prompted, select "Default" for the style, "Slate" for the base color, and confirm using CSS variables.
4.  Create a `.env.local` file and add the following placeholder variables. We will populate these later.

    ```
    # Appwrite
    NEXT_PUBLIC_APPWRITE_ENDPOINT=
    NEXT_PUBLIC_APPWRITE_PROJECT_ID=

    # Hedera
    HEDERA_ACCOUNT_ID=
    HEDERA_PRIVATE_KEY=

    # AI Service
    REQUESTY_AI_API_KEY=
    ```
```

---

#### **Prompt 2: Appwrite Client Configuration**

```text
We need to create a centralized client to interact with the Appwrite backend.

Create a new file at `lib/appwrite.ts`. Inside this file:
1.  Import `Client`, `Account`, and `Databases` from the `appwrite` package.
2.  Create a new `Client` instance.
3.  Configure the client using the `setEndpoint` and `setProject` methods, reading the values from the public environment variables (`NEXT_PUBLIC_APPWRITE_ENDPOINT` and `NEXT_PUBLIC_APPWRITE_PROJECT_ID`).
4.  Export singleton instances of the `Client`, `Account`, and `Databases` services so they can be reused throughout the application.
```

---

#### **Prompt 3: Basic UI Layout and Theme Toggle**

```text
Let's create the basic layout shell for our application.

1.  Create a `components/ThemeProvider.tsx` component that uses `next-themes` to manage light and dark modes.
2.  Create a `components/ThemeToggle.tsx` component that renders a `shadcn/ui` `Button` with a Sun and Moon icon (`lucide-react`) to toggle the theme.
3.  In the root `app/layout.tsx`, import and use the `ThemeProvider`. In the `body`, render the `ThemeToggle` component in the top-right corner for now, followed by the `children`. This ensures all pages will have the theme provider and toggle.
```

---

#### **Prompt 4: Login UI Component**

```text
We will now build the UI for user authentication.

1.  Use the `shadcn/ui` CLI (`pnpm dlx shadcn-ui@latest add ...`) to add the following components: `card`, `input`, `button`, `label`, `separator`.
2.  Create a new component file at `components/auth/AuthForm.tsx`.
3.  Inside `AuthForm`, build a form using the `Card` components (`Card`, `CardHeader`, `CardTitle`, `CardContent`, `CardFooter`).
4.  The form should contain:
    *   A `Button` for "Sign in with Google".
    *   A `Separator` with text "OR CONTINUE WITH".
    *   An `Input` field for an email address, associated with a `Label`.
    *   A `Button` with the text "Sign in with Email".
```

---

#### **Prompt 5: Implement Authentication Logic**

```text
Now, let's wire the `AuthForm` to our Appwrite client.

1.  Modify `components/auth/AuthForm.tsx`.
2.  Import the `account` service from `lib/appwrite.ts` and `useState` from React.
3.  Create state to hold the email input value.
4.  Implement the `handleGoogleAuth` function. When the Google button is clicked, it should call `account.createOAuth2Session('google', 'http://localhost:3000/dashboard')`.
5.  Implement the `handleEmailAuth` function. When the email button is clicked, it should call `account.createMagicURLSession('unique()', email, 'http://localhost:3000/dashboard')`. It should show a success message upon submission.
6.  Wrap the entire component logic in `try...catch` blocks and log any errors to the console.
```

---

#### **Prompt 6: Create Protected Dashboard and Root Page**

```text
We need to create a home page for authentication and a dashboard page for authenticated users.

1.  Create a new route directory `app/dashboard`. Inside it, create a `page.tsx` file. This will be our protected dashboard. For now, it should just display an `h1` tag with "Dashboard" and a `p` tag with "Welcome to your dashboard."
2.  Modify the root `app/page.tsx`. This will be our login page.
3.  Clear the existing boilerplate. Import and render the `AuthForm` component in the center of the page.
4.  Create a new file `hooks/useAuth.ts`. This hook will check for a current user session using `account.get()`. It should return the `user` object and a `loading` state.```

---

#### **Prompt 7: Implement Protected Routes and Logout**

```text
Let's secure the dashboard and provide a way for users to log out.

1.  Create a new component `components/auth/LogoutButton.tsx`. This component should render a `Button` that, when clicked, calls `account.deleteSession('current')` and then uses the Next.js `useRouter` to redirect the user to the home page (`/`).
2.  In `app/dashboard/page.tsx`, use the `useAuth` hook. Add a `useEffect` that checks if `loading` is false and `user` is null. If so, redirect the user to the home page (`/`). At the top of the page, display the user's email and render the `LogoutButton` component.
```

---

#### **Prompt 8: Company Profile Onboarding Flow**

```text
We need to ensure every new user sets up a company profile.

1.  First, go to your local Appwrite Console UI. Manually create a new Database called "CryptexDB". Inside it, create a new Collection named `Companies` with the following attributes: `company_name` (string, required), `role` (string, required), `description` (string, optional), `country_of_operation` (string, required), and `ctt_balance` (number, default: 0). Also add a relationship attribute to the `Users` collection.
2.  Create a new route `app/onboarding/page.tsx`.
3.  Create a new component `components/onboarding/CompanyForm.tsx`. This form should use `shadcn/ui` components (`Input`, `Select`, `Textarea`, `Button`) to capture all the fields for the `Companies` collection.
4.  The `app/onboarding/page.tsx` should render this form.
```

---

#### **Prompt 9: Onboarding Middleware and Backend Logic**

```text
Let's create the logic to enforce the onboarding flow.

1.  First, create an Appwrite Function named `create-company-profile`. This Node.js function will be triggered by an HTTP request. It should:
    *   Accept `company_name`, `role`, `description`, etc., from the request body.
    *   Get the current user's ID from the `APPWRITE_FUNCTION_USER_ID` environment variable.
    *   Use the Node SDK to create a new document in the `Companies` collection with the provided data.
    *   Link this new company document to the user who triggered the function.
    *   Return a success or error response.
2.  In `components/onboarding/CompanyForm.tsx`, implement the `onSubmit` handler. It should call the `create-company-profile` Appwrite Function with the form data. On success, it should redirect the user to `/dashboard`.
3.  Modify the `useAuth.ts` hook. In addition to the user, also check if the user has a related company document. Return a boolean `hasCompany`.
4.  In `app/dashboard/page.tsx`, use the `hasCompany` value from the hook. If the user is logged in but `hasCompany` is false, redirect them to `/onboarding`.
```

---

#### **Prompt 10: Finalize Schemas and Dashboard Layout**

```text
Now we set up the rest of our data structure and the main application layout.

1.  In your Appwrite Console, manually create the remaining collections as defined in the blueprint: `Products`, `Orders`, `Digital_Product_Passports`, `Lifecycle_Events`, and `Invoices`. Define all their attributes and relationships precisely.
2.  Create a new component `components/layout/Sidebar.tsx`. This component should display navigation links for "Dashboard", "Orders", "Products", etc.
3.  Create a new layout file `app/dashboard/layout.tsx`. This layout should render the `Sidebar` on the left and a main content area on the right where the page `children` will be displayed. This ensures all dashboard pages have a consistent look and feel.
```

---

#### **Prompt 11: Displaying Core Data (Orders)**

```text
Let's fetch and display the user's orders on the dashboard.

1.  Use the `shadcn/ui` CLI to add the `table` component.
2.  Create a new route `app/dashboard/orders/page.tsx`.
3.  Create a new component `components/orders/OrderList.tsx`. This component will be a **Server Component**.
4.  Inside `OrderList`, use the Appwrite Node SDK to fetch all documents from the `Orders` collection that are related to the user's company. (You will need to manage user sessions for server-side rendering).
5.  Render the fetched orders in a table using the `shadcn/ui` `Table` components. Display columns for Order ID, Status, Total Amount, and Expected Delivery Date.
6.  In `app/dashboard/orders/page.tsx`, import and render the `OrderList` component.
```

---

#### **Prompt 12: Hedera & AI Client Setup**

```text
Let's encapsulate the logic for our external AI and Hedera services.

1.  Create a new file `lib/hedera.ts`. This file should:
    *   Import the Hedera SDK.
    *   Create a `Client` for the Hedera Testnet.
    *   Use the `HEDERA_ACCOUNT_ID` and `HEDERA_PRIVATE_KEY` from the environment variables to configure the client operator.
    *   Export a helper function `submitToHCS(message)` that takes a string, creates a `TopicMessageSubmitTransaction`, executes it, and returns the transaction receipt.
2.  Create a new file `lib/ai.ts`. This file should:
    *   Contain a function `parseDocumentWithAI(fileText)` that takes the text extracted from a PDF.
    *   This function will use `fetch` to make a POST request to the Requesty AI API endpoint.
    *   It should send the `REQUESTY_AI_API_KEY` in the authorization header and the file text in the body.
    *   It should return the parsed JSON data from the AI service.
```

---

#### **Prompt 13: AI-Powered Order Creation Flow**

```text
This is the core feature. Let's build the complete flow for creating an order via AI.

1.  Create a new route and page at `app/dashboard/orders/new/page.tsx`.
2.  On this page, create a component `components/orders/CreateOrderForm.tsx`. This component should contain a file input field and a "Create Order" button.
3.  Create an Appwrite Function named `parse-po-document`. This function takes a file, extracts the text, and calls the `parseDocumentWithAI` logic from `lib/ai.ts`. It should return the structured JSON.
4.  Create a second Appwrite Function named `create-order`. This function takes the parsed JSON data. It should:
    *   Create a new document in the `Orders` collection.
    *   Call the `submitToHCS` logic from `lib/hedera.ts` with the order details.
    *   Update the newly created order document with the returned Hedera transaction ID.
5.  In the `CreateOrderForm` component, when a file is uploaded and the button is clicked, it should first call the `parse-po-document` function. Upon receiving the parsed data, it should then call the `create-order` function. Show notifications for each step. On final success, redirect to the main orders page.
```