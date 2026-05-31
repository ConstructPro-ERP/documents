# UI/UX & Accessibility

The ConstructPro ERP interface is designed for the construction industry's operational staff — professionals who need to act quickly, navigate intuitively, and trust the data they see. The design system prioritizes **clarity, efficiency, visual consistency, and inclusivity** across all user-facing screens.

---

## 1. Design Philosophy

The interface follows an **enterprise dashboard paradigm** with the following core principles:

| Principle          | Implementation                                                                                                                                    |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Clarity**        | Consistent typography hierarchy, whitespace, and visual grouping so information is never cluttered                                                |
| **Efficiency**     | Every core action is reachable within a maximum of **3 clicks** from the main dashboard (NFR-06)                                                  |
| **Trust**          | Real data surfaced through accurate KPI cards, status badges, and progress indicators — no ambiguous states                                       |
| **Responsiveness** | The interface adapts fully across desktop (1440px+), laptop (1024px–1440px), tablet (768px–1024px), and mobile (320px–768px) breakpoints (NFR-05) |
| **Accessibility**  | WCAG 2.1 Level AA compliance for all user-facing components                                                                                       |

---

## 2. Design System

### 2.1 Typography

| Element                     | Font           | Weight         | Size      |
| --------------------------- | -------------- | -------------- | --------- |
| Page Titles (H1)            | Inter          | 700 (Bold)     | 28px      |
| Section Headings (H2)       | Inter          | 600 (SemiBold) | 22px      |
| Subsection Headings (H3)    | Inter          | 600 (SemiBold) | 18px      |
| Body Text                   | Inter          | 400 (Regular)  | 14px–16px |
| Labels & Captions           | Inter          | 500 (Medium)   | 12px      |
| Monospaced Data (IDs, refs) | JetBrains Mono | 400            | 13px      |

All fonts are loaded from Google Fonts with `font-display: swap` to prevent Flash of Invisible Text (FOIT).

### 2.2 Color Palette

| Token              | Hex Value | Usage                                              |
| ------------------ | --------- | -------------------------------------------------- |
| `--primary`        | `#2563EB` | Primary buttons, active nav links, links           |
| `--primary-hover`  | `#1D4ED8` | Hover state for primary interactive elements       |
| `--secondary`      | `#64748B` | Secondary actions, muted text                      |
| `--success`        | `#16A34A` | Paid status, completed milestones                  |
| `--warning`        | `#D97706` | Partially paid, in-progress, at-risk status        |
| `--danger`         | `#DC2626` | Overdue invoices, cancelled projects, error states |
| `--surface`        | `#F8FAFC` | Page background                                    |
| `--card`           | `#FFFFFF` | Card and panel backgrounds                         |
| `--border`         | `#E2E8F0` | Subtle borders, dividers                           |
| `--text-primary`   | `#0F172A` | Primary text — contrast ratio ≥ 7:1                |
| `--text-secondary` | `#475569` | Secondary / helper text — contrast ratio ≥ 4.5:1   |

### 2.3 Spacing & Grid

- **Base unit**: 4px
- **Spacing scale**: 4, 8, 12, 16, 24, 32, 48, 64, 96px
- **Layout**: 12-column CSS Grid with 24px gutters on desktop; 4-column on mobile
- **Content max-width**: 1280px (centered with auto margins)

### 2.4 Component Library

All UI components are built with **Tailwind CSS + shadcn/ui** as the base, customized to the ConstructPro design token system. Key components include:

| Component          | Description                                                                  |
| ------------------ | ---------------------------------------------------------------------------- |
| `<KPICard />`      | Displays a metric label, current value, trend indicator (↑↓), and sparkline  |
| `<StatusBadge />`  | Color-coded pill badge with both color and text label (no color-only status) |
| `<DataTable />`    | Sortable, filterable, paginated table with column visibility controls        |
| `<ProgressBar />`  | Labeled percentage bar for milestone/project completion tracking             |
| `<Modal />`        | Accessible dialog (focus trap, Escape to close, aria-modal)                  |
| `<Sidebar />`      | Persistent collapsible navigation sidebar                                    |
| `<Breadcrumb />`   | Context path for deep navigation                                             |
| `<FileUploader />` | Drag-and-drop zone with file type/size validation and upload progress        |
| `<DatePicker />`   | Keyboard-accessible calendar selector                                        |

---

## 3. Core UI Screens

### 3.1 Dashboard (Admin / Analyst View)

The Dashboard is the system's landing page after login. It provides a real-time summary of business health.

**Key Components:**

- **KPI Cards Row**: Total Revenue (Month), Pending Invoices (Value), Active Projects, Overdue Tasks
- **Project Status Breakdown**: Doughnut chart showing projects by status (Planning, In Progress, On Hold, Completed)
- **Revenue Trend Chart**: Monthly bar chart (12 months) of invoiced vs. collected revenue
- **Recent Activity Feed**: Chronological list of latest system events (new leads, invoice paid, project status changed)
- **AI Risk Alerts Panel**: Highlighted projects flagged by the AI module for delay risk or payment risk
- **Quick Actions**: Buttons for "New Lead", "New Quotation", "New Invoice"

---

### 3.2 Sales Management Screens

**Lead Management (`/sales/leads`)**

- Data table of all leads with columns: Company, Contact, Status, Created Date, Assigned To
- Filters: Status, Date Range, Assigned Sales Rep
- Inline status transition buttons (e.g., "Mark as Qualified", "Convert to Client")
- "New Lead" modal with form validation

**Quotation Builder (`/sales/quotations/new`)**

- Multi-section form: Client Selection → Line Items → Summary → Notes
- **Line Item Builder**: Dynamic rows where the user enters description, quantity, and unit price; `line_total` is auto-calculated
- Live running **Grand Total** displayed in a sticky summary panel
- "Save as Draft" and "Send to Client" actions
- Generated PDF preview before sending

---

### 3.3 Project Management Screens

**Project List (`/projects`)**

- Cards/table toggle view; each card shows project name, client, PM, status badge, budget, and completion %
- Filter by: Status, Project Manager, Date Range

**Project Detail (`/projects/[id]`)**

- **Header**: Project name, status badge, budget vs. spent progress bar
- **Tabs**: Overview | Milestones | Tasks | Documents | Finance | Activity Log
- **Milestones Tab**: Vertical timeline of milestones with due date, status, and completion percentage. "Add Milestone" inline form.
- **Tasks Tab**: Kanban-style board: `TODO | IN PROGRESS | DONE` columns with drag-and-drop reordering
- **Calendar View**: Milestone and task deadlines synced and displayed on a monthly calendar grid

---

### 3.4 Financial Management Screens

**Invoice List (`/finance/invoices`)**

- Table with: Invoice #, Project, Client, Amount Due, Amount Paid, Status, Due Date
- Status color-coded: PENDING (blue), PARTIALLY PAID (amber), PAID (green), OVERDUE (red)
- Filter by: Status, Date Range, Client, Project
- "Generate Invoice" button (for Accountant role)

**Invoice Detail (`/finance/invoices/[id]`)**

- Invoice summary: Total due, total paid, outstanding balance
- Payment history table with amount, date, method, and reference number
- "Record Payment" modal with validation
- "Download PDF" button to retrieve the Cloudinary-hosted invoice PDF

---

### 3.5 Document Management Screen (`/documents`)

- File browser interface with folder structure by project and document category
- Columns: Filename, Category, Uploaded By, Date, File Size, Actions (Download, Delete)
- **Drag-and-drop file upload zone** with real-time upload progress bar
- Category filter sidebar (Contract, Drawing, Approval, Report, Other)
- File type icons (PDF, image, Word, spreadsheet) for quick visual identification

---

### 3.6 Analytics & Reporting Screen (`/analytics`)

- **KPI Summary Row**: Total Revenue YTD, Collection Rate %, Average Project Duration, On-Time Completion Rate
- **Revenue by Month**: Bar chart (12-month rolling)
- **Project Completion Distribution**: Pie chart
- **Top Clients by Revenue**: Horizontal bar chart
- **Payment Delay Heatmap**: Calendar heatmap showing payment receipt vs. due date patterns
- **AI Predictions Panel**: Tabular list of risk predictions with confidence scores

---

### 3.7 Client Portal (`/client-portal`)

Accessible to **Client Portal User** role only. Presents a simplified, read-only view:

- Active project status card with milestone progress
- Invoice summary (outstanding balance, payment history)
- Document list (downloadable files for their own project)
- Quotation view (approved quotation details)

---

## 4. Accessibility – WCAG 2.1 Level AA Compliance

ConstructPro adheres to the **POUR** framework across all interface components.

### 4.1 Perceivable

| Requirement             | Implementation                                                                                                                                                     |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Text Contrast Ratio** | Minimum 4.5:1 for body text; 3:1 for large text (18px+ or bold 14px+). All color tokens verified against WCAG AA. `--text-primary` achieves 14.5:1 on `--surface`. |
| **Non-Text Contrast**   | UI components (buttons, inputs, focus rings) maintain ≥ 3:1 contrast against adjacent colors                                                                       |
| **Color Independence**  | Status badges always include both a color indicator **and** a text label (e.g., "● PAID" not just a green dot). Charts include labeled legends.                    |
| **Alt Text**            | All KPI chart images and AI prediction graphs include descriptive `alt` attributes summarizing the displayed data                                                  |
| **Text Resize**         | The layout supports up to 200% browser text zoom without loss of content or horizontal scrolling                                                                   |
| **Responsive Reflow**   | Content reflows to a single column at 320px viewport width (WCAG 1.4.10)                                                                                           |

### 4.2 Operable

| Requirement               | Implementation                                                                                                                                 |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **Keyboard Navigation**   | All interactive elements (buttons, links, form inputs, modals, dropdowns) are fully operable via keyboard alone                                |
| **Tab Order**             | Logical tab order matches the visual reading order of each page                                                                                |
| **Focus Visible**         | All focusable elements display a visible focus ring (2px solid `--primary` with 2px offset). Focus rings are never suppressed globally.        |
| **No Keyboard Traps**     | Modals and dropdowns implement a **focus trap** that cycles within the component while open, and returns focus to the trigger element on close |
| **Skip Navigation**       | A "Skip to main content" link is the first focusable element on every page, allowing keyboard users to bypass the navigation sidebar           |
| **No Timing Constraints** | No timed sessions or auto-expiring content that users cannot extend                                                                            |

### 4.3 Understandable

| Requirement                   | Implementation                                                                                                     |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| **Required Field Indicators** | All mandatory form inputs have `aria-required="true"` and a visual asterisk (\*) with a legend                     |
| **Error Identification**      | Validation errors are displayed adjacent to the relevant field, linked via `aria-describedby` to the input element |
| **Error Description**         | Error messages clearly state what is wrong and how to fix it (e.g., "Contact email must be a valid email address") |
| **Field Instructions**        | Helper text is rendered immediately before or beneath each input (not as placeholder text alone)                   |
| **Language**                  | `<html lang="en">` declared on all pages                                                                           |
| **Consistent Navigation**     | The sidebar navigation structure and order is identical across all pages                                           |
| **Labels**                    | Every form input has an associated `<label>` element — no input is labeled by placeholder text alone               |

### 4.4 Robust

| Requirement              | Implementation                                                                                                                                                                 |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Semantic HTML**        | Pages use semantic elements: `<nav>`, `<main>`, `<header>`, `<footer>`, `<section>`, `<article>`, `<aside>`                                                                    |
| **ARIA Landmarks**       | `role="navigation"`, `role="main"`, `role="complementary"` applied to structural regions                                                                                       |
| **ARIA Live Regions**    | Dynamic updates (e.g., "Invoice saved successfully" toast notifications, dashboard KPI refreshes) are announced via `aria-live="polite"` regions without interrupting the user |
| **ARIA Labels**          | Icon-only buttons include `aria-label` attributes (e.g., `<button aria-label="Delete document">`)                                                                              |
| **Valid HTML**           | All rendered HTML is validated against W3C standards; no deprecated or invalid element usage                                                                                   |
| **Screen Reader Tested** | Key flows tested with NVDA (Windows) and VoiceOver (macOS) to verify announcement quality                                                                                      |

---

## 5. Responsive Breakpoints

| Breakpoint  | Viewport Width  | Layout Behavior                                                 |
| ----------- | --------------- | --------------------------------------------------------------- |
| **Mobile**  | 320px – 767px   | Single column, sidebar collapses to a hamburger menu overlay    |
| **Tablet**  | 768px – 1023px  | Two-column layout, sidebar is icon-only by default (expandable) |
| **Laptop**  | 1024px – 1439px | Full sidebar + content area; tables show 5–6 columns            |
| **Desktop** | 1440px+         | Full sidebar + wide content area; multi-panel layouts enabled   |

---

## 6. Interaction Design & Micro-Animations

| Interaction               | Animation                      | Duration          |
| ------------------------- | ------------------------------ | ----------------- |
| Page transitions          | Fade-in slide-up               | 200ms ease-out    |
| Sidebar expand/collapse   | Smooth width transition        | 250ms ease-in-out |
| Modal open/close          | Scale + fade                   | 150ms ease-out    |
| Toast notification appear | Slide-in from bottom-right     | 200ms ease-out    |
| Table row hover           | Background color shift         | 100ms             |
| Button click feedback     | Scale-down (0.97) + color      | 100ms             |
| KPI card load             | Count-up animation for numbers | 800ms ease-out    |
| Progress bars             | Width animation on mount       | 600ms ease-out    |

All animations respect the user's **`prefers-reduced-motion`** OS setting — when enabled, animations are replaced with instant state changes.
