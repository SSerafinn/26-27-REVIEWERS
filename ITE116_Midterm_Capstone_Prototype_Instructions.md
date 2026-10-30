# ITE 116 – Integrative and Programming Technologies
## Midterm Requirement: Capstone Prototype (Early Version)

---

## Overview

For your midterm, your capstone group will build and demonstrate an **early working prototype** of your capstone project.

Your title is still going through defense, so this is **not** the finished system. The goal is to prove that the foundation works: users can log in according to their roles, and **one core feature** of your project runs from start to finish.

---

## What Your Prototype Must Have

### 1. Authentication for Different Users

Your system must support **at least two types of users** (roles) that fit your project, for example:

- Admin and Student
- Staff and Customer
- Barangay Official and Resident

Your prototype must let users:

- **Log in** with an email and password, and **log out**.
- **Register**, or log in with **pre-created accounts** (seeded) for each role.
- See pages that **match their role** after logging in. For example, an Admin lands on an admin dashboard and a Student lands on a student page.
- **Not** open pages that belong to another role. Visiting another role's page by typing its URL must be blocked.
- **Not** open any protected page while logged out. Such pages must redirect to the login page.

> **Tip:** a simple way to start is a `role` column on the `users` table, plus a check in your routes or controllers.

### 2. At Least One Core Feature

Choose **one feature that is central to your capstone**, the feature your system cannot do without. Examples:

- Submitting and approving a request
- Booking a schedule or appointment
- Recording and viewing transactions or inventory
- Posting and managing announcements or reports

The feature must:

- **Work end to end**: data is entered, saved to the database, and displayed back.
- **Validate input** on the server.
- Be **connected to the logged-in user** where it makes sense, for example "my requests" or "requests submitted to me".
- Be used by **at least one of your roles**. Showing how two roles interact through it is encouraged, such as one submits and the other approves.

> If your title or scope changes after the defense, still build and present the feature closest to your current plan. Prototypes are expected to change.

---

## Technical Requirements

- Built with **Laravel**, applying what we covered in class: routes, controllers, models, migrations, validation, Blade, and authentication.
- Database tables created through **migrations**, not by hand.
- A **seeder** that creates at least one demo account for **each role**.
- Code kept in a **Git repository** (GitHub).

---

## What to Submit

Submit the following through **[submission channel]** on or before **[November 9, 2026]**:

1. **GitHub repository link.** Make sure the repository is accessible to your instructor.
2. **A README.md** in the repository containing:
   - Project title and group members
   - A short description of the system (2–3 sentences)
   - The user roles and what each role can do in the prototype
   - The core feature you built
   - **Demo accounts** for each role (email and password)
   - **Setup steps**: the commands needed to run the project from a fresh clone (install, `.env`, migrate, seed, run)

> Do **not** upload the `vendor/` or `node_modules/` folders.

---

## Demonstration

Each group will demonstrate their prototype on **[November 10, 2026]**. Plan for about **[10] minutes** per group, followed by questions.

Follow this flow:

1. **Introduce** your project: the problem, the target users, and the roles in your system.
2. **Log in as each role** and show what each one sees.
3. **Show access protection:**
   - Try to open another role's page while logged in as a different role.
   - Try to open a protected page while logged out.
4. **Demonstrate your core feature** from start to finish, including what happens when invalid input is submitted.
5. **Walk through your code:** the routes, the part of the code that checks roles, and the controller for your core feature.
6. **Answer questions.** Any group member may be asked to explain any part of the system.

### Before Your Demo

- Run your project from a **fresh setup** at least once (`migrate:fresh --seed`) so the demo accounts work.
- Have the **demo accounts** ready on screen or on paper.
- Make sure `npm run dev` (or `npm run build`) and `php artisan serve` are running before your turn.
- Bring a backup: screenshots or a short screen recording in case of technical problems.

---

## Reminders

- This is a **prototype**. A clean, working feature matters more than a polished design.
- Every member should contribute and be able to explain the system.
- Use of AI tools must follow class guidelines. You must be able to explain every part of your code.
