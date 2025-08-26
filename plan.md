Below is the detailed plan for implementing the Wahana Ilmu Spentwonine library management system along with all the dependent files/changes. This plan assumes a Next.js app using local storage for persistence on a single-computer system and includes modern, clean UI elements with red, yellow, and green as the primary colors.

---

### Overall System Structure

- The system consists of two main flows:  
 – The front-end library transaction module (for book borrowing/return)  
 – An admin mode for backup & statistics protected by login

- Data (transactions and admin state) is stored in local storage.  
- Date-related utilities will compute the automatic return date (3 working days) and calculate lateness (max 5 working days) by skipping Saturdays and Sundays.  
- CSV export for backup (monthly, yearly, all) and PDF download for diagrams are generated client-side.

---

### Detailed File-by-File Changes

#### 1. Global Styles & Configuration

- **File:** `src/app/globals.css`  
  **Changes:**  
  - Add new CSS classes for a modern UI using red, yellow, and green accents (e.g., header background, button styles, card borders).  
  - Ensure responsive design with proper spacing and typography.  
  - Provide fallback styles for error messages and alerts.

---

#### 2. Utility Functions

- **File:** `src/lib/utils.ts`  
  **Changes:**  
  - Add a function `getReturnDate(borrowDate: Date, workingDays: number): Date` that iterates day-by-day (ignoring weekends) to calculate the correct return date.  
  - Add a `getLateDays(currentDate: Date, returnDate: Date): number` function to compute lateness (capped at 5 working days).  
  - Implement helper functions for CSV export:  
  - `exportToCSV(data: any[], filename: string): void` which creates a CSV string and triggers a Blob download.  
  - Optionally, add a function `exportDiagramToPDF(diagramElement: HTMLElement, filename: string): void` which uses a library like jsPDF, and update `package.json` with that dependency if needed.  
  - Wrap local storage reads/writes in try-catch blocks for error handling.

---

#### 3. Library Transaction Module

- **File:** `src/app/library/page.tsx`  
  **Changes:**  
  - Create a Next.js page that renders the main library system.  
  - Import and include the BorrowForm and TransactionList components.

- **File:** `src/components/library/BorrowForm.tsx` *(New File)*  
  **Features & Changes:**  
  - Build a form that collects:  
  - Name  
  - Class (dropdown with options for Kelas 7A-7I, 8A-8I, 9A-9I, plus teacher/karyawan choices)  
  - Judul Buku (Book Title)  
  - Tanggal Peminjaman – default to current date (read-only)  
  - Nomor HP  
  - On submit, auto-calculate the return date using `getReturnDate()` for 3 working days ahead, and store the transaction in local storage.  
  - Validate inputs (e.g., non-empty, valid phone number) and use alert components (from `src/components/ui/alert.tsx`) for error messages.

- **File:** `src/components/library/TransactionList.tsx` *(New File)*  
  **Features & Changes:**  
  - Display a table (or list using `table.tsx` component) showing each transaction with columns:  
  - Nama, Kelas, Judul Buku, Tanggal Peminjaman, Tanggal Pengembalian, Nomor HP, Keterlambatan  
  - For “keterlambatan”, use `getLateDays()` to compute the difference between the current date and the return date.  
  - Include error handling for rendering if no transactions are found.

---

#### 4. Admin Backup & Statistics Module

- **File:** `src/app/admin/page.tsx`  
  **Changes:**  
  - Create an admin page.  
  - Check for the authenticated state (stored in local storage) and render either the AdminLogin component or the BackupStatistics dashboard.  
  - Ensure that if the session expires or is idle, the system auto-locks (using the custom idle hook described later).

- **File:** `src/components/admin/AdminLogin.tsx` *(New File)*  
  **Features & Changes:**  
  - Render a login form with fields for username and password.  
  - Validate against hardcoded values (username: "admin", password: "perpuspentwonine29").  
  - On successful login, set an authenticated flag in local storage and proceed to load the backup & statistics features.  
  - Provide proper error messages (using alert dialog components) for invalid login attempts.

- **File:** `src/components/admin/BackupStatistics.tsx` *(New File)*  
  **Features & Changes:**  
  - Display buttons to trigger CSV export for monthly, yearly, and all data backups.  
  - Use the utility’s CSV export function to handle downloads.  
  - Render a simple diagram (e.g., using HTML/CSS div bars representing transaction counts per month) that can be exported as a PDF.  
  - Include a "Keluar" (logout) button that clears the admin authentication flag.  
  - Implement an auto lock feature (via the useIdleTimer hook) so the backup/statistics panel locks after inactivity, requiring re-login.

---

#### 5. Auto-Lock / Idle Timeout

- **File:** `src/hooks/useIdleTimer.ts` *(New File)*  
  **Changes:**  
  - Create a custom React hook that listens for user activity events (mouse move, key press).  
  - If no activity is detected for a predetermined period (e.g., 5 minutes), trigger an auto logout function to clear the admin authentication state and redirect to login.  
  - Ensure proper cleanup of event listeners on component unmount.

---

#### 6. Package Dependencies & Configurations

- **File:** `package.json`  
  **Changes (if needed):**  
  - Optionally add libraries for PDF generation (e.g., jsPDF) if a robust PDF export is required for the diagram.  
  - Update any script configurations if additional build steps or assets are required.

---

### UI/UX Considerations

- Use simple, modern input elements with clear labels and a consistent red-yellow-green theme (red for headers/borders, yellow for buttons, green for success messages).  
- The BorrowForm and TransactionList should be clearly separated in a card layout with ample spacing and clear typography.  
- The admin backup section should have a secure login view and a dashboard-style layout for exporting data and viewing statistics.  
- All images that might be used (if any future need arises) will follow the proper HTML `<img>` guidelines using descriptive placeholder URLs (not needed on dashboards/forms unless explicitly required).

---

### Error Handling & Best Practices

- Validate all form inputs and wrap local storage operations in try-catch blocks.  
- Display user-friendly error messages using prebuilt UI alert components.  
- Ensure that the idle timer is robust so that unauthorized access is minimized.  
- Use TypeScript types consistently to enhance reliability and maintainability.

---

### Summary

- The plan creates two Next.js pages: one for library transactions and one for admin backup/statistics.  
- New components (BorrowForm, TransactionList, AdminLogin, BackupStatistics) and a custom idle timer hook are introduced.  
- Utility functions in utils.ts handle date calculations and CSV/PDF export.  
- UI elements are styled with a red, yellow, and green theme to ensure clarity and modern design.  
- Security measures include a hardcoded admin login and auto-lock functionality based on inactivity.  
- All local storage interactions and potential errors are properly managed with try-catch blocks.  
- The system is built for single-computer use using a simple, local storage solution.  
- This plan ensures a comprehensive, user-friendly library management system tailored for UPT SMPN 29 GRESIK.
