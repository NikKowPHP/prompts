# **Phase [Letter]: [Name of Phase]**

**Goal:** [State the primary, measurable objective for this phase. What is the "definition of done"? What will be true about the codebase after this phase is complete?]

---

### 1. [First Logical Group of Tasks, e.g., Configuration, Cleanup, Scaffolding]

- `[ ]` **Task 1.1: [Action-Oriented Task Name, e.g., Define the Core Database Schema]**
  - **File:** `path/to/the/relevant/file.ext`
  - **Action:** [Be specific: "Replace the entire file content", "Update the function `xyz`", "Add the following properties"].
  - **Content:**

    ```[language]
    // Provide the exact code to be used.
    // Use comments to explain the purpose of key sections, especially in a template.
    // Example for a Prisma schema:

    model User {
      id             String    @id @default(uuid())
      email          String    @unique
      // ... other user fields
    }

    // A core content model for this application
    model Post {
      id        String   @id @default(cuid())
      title     String
      content   String   @db.Text
      authorId  String
      author    User     @relation(fields: [authorId], references: [id])
    }
    ```

- `[ ]` **Task 1.2: [Another Task in this Group]**
  - **File:** `path/to/another/file.ext`
  - **Action:** ...
  - **Content:** ...

---

### 2. [Second Logical Group of Tasks, e.g., Dependency Management, Component Creation]

- `[ ]` **Task 2.1: [Example: Install Project Dependencies]**
  - **Action:** Run the following shell command to install the required libraries for this phase.
  - **Command:**

    ```bash
    # List dependencies by category for clarity
    # UI & Styling
    npm install lucide-react clsx tailwind-merge

    # State Management & Data Fetching
    npm install zustand @tanstack/react-query

    # Forms & Validation
    npm install react-hook-form zod
    ```

- `[ ]` **Task 2.2: [Example: Initialize a UI Library]**
  - **Action:** Run the CLI tool and provide the following answers when prompted.
  - **Command:**
    ```bash
    npx shadcn-ui@latest init
    ```
  - **Prompts:**
    - Use TypeScript? **yes**
    - Style: **[e.g., Default]**
    - Base color: **[e.g., Slate]**
    - Components alias: **`@/components`**
    - Utils alias: **`@/lib/utils`**
    - ... and so on.

---

### 3. [Third Logical Group of Tasks, e.g., Creating New Files/Pages]

- `[ ]` **Task 3.1: [Example: Create a New Reusable Component]**
  - **File:** `src/components/ui/[new-component-name].tsx`
  - **Action:** Create a new file with the following content.
  - **Content:**

    ```tsx
    // A well-structured, reusable component.
    // Props should be clearly defined.

    import React from 'react';

    interface [ComponentName]Props {
      title: string;
      children: React.ReactNode;
    }

    export const [ComponentName] = ({ title, children }: [ComponentName]Props) => {
      return (
        <div className="card-styles">
          <h3 className="card-title-styles">{title}</h3>
          <div>{children}</div>
        </div>
      );
    };
    ```

- `[ ]` **Task 3.2: [Example: Scaffold a New Application Page]**
  - **File:** `src/app/[page-name]/page.tsx`
  - **Action:** Create this new file to establish the route and provide a placeholder.
  - **Content:**

    ```tsx
    import React from 'react';

    export default function [PageName]Page() {
      return (
        <div className="container mx-auto p-4">
          <h1 className="text-2xl font-bold">[Page Name]</h1>
          <p>
            Placeholder content for the [Page Name] page. The main components for this feature will be added here in a later phase.
          </p>
        </div>
      );
    }
    ```

---

_Continue with as many sections and tasks as are needed to fulfill the phase's goal._
