# Laravel API — Attendance System Project Plan

A beginner-friendly build plan for a Laravel API-only attendance system with client and admin features.

Environment

- PHP: 8.2.12 (cli) (built: Oct 24 2023) — Zend Engine v4.2.12
- Notes: This document is aligned to PHP 8.2.12 (your machine). When creating the Laravel project, make sure the Laravel/composer versions you choose are compatible with PHP 8.2.12 (Laravel 10+ generally supports PHP 8.1+, and works on PHP 8.2).

---

## Phase 1 — Project Setup

- [ ] Install PHP 8.2.12 (or PHP 8.2+) and Composer on your machine
- [ ] Create a new Laravel project: `composer create-project laravel/laravel attendance-api`
- [ ] Configure `.env` file (database name, username, password)
- [ ] Create the database in MySQL (e.g. `attendance_db`)
- [ ] Test DB connection: `php artisan migrate`
- [ ] Install Laravel Sanctum: `composer require laravel/sanctum`
- [ ] Publish Sanctum config: `php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"`
- [ ] Run migrations: `php artisan migrate`

---

## Phase 2 — Database & Models

- [ ] Add `role` column to users table (migration: `add_role_to_users_table`)
  - `enum('role', ['user', 'admin'])->default('user')`
- [ ] Create `attendances` migration with fields:
  - `user_id` (foreign key → users)
  - `time_in` (datetime)
  - `time_out` (datetime, nullable)
  - `date` (date)
  - `notes` (text, nullable)
- [ ] Run `php artisan migrate`
- [ ] Update `User` model
  - Add `HasApiTokens` trait
  - Add `role` to `$fillable`
  - Define `hasMany(Attendance::class)` relationship
- [ ] Create `Attendance` model
  - Add all fields to `$fillable`
  - Define `belongsTo(User::class)` relationship

---

## Phase 3 — Authentication

- [ ] Generate `AuthController`: `php artisan make:controller AuthController`
- [ ] Implement `register()` method
  - Validate name, email, password
  - Hash password with `bcrypt()`
  - Return token on success
- [ ] Implement `login()` method
  - Validate credentials with `Auth::attempt()`
  - Return Sanctum token on success
- [ ] Implement `logout()` method
  - Delete current token: `$request->user()->currentAccessToken()->delete()`
- [ ] Implement `forgotPassword()` method
  - Use `Password::sendResetLink()`
- [ ] Implement `resetPassword()` method
  - Use `Password::reset()`
- [ ] Configure mail driver in `.env` (use Mailtrap for local testing)

---

## Phase 4 — User Profile

- [ ] Generate `ProfileController`: `php artisan make:controller ProfileController`
- [ ] Implement `show()` — return `auth()->user()`
- [ ] Implement `update()` — validate and update name/email
- [ ] Protect routes with `auth:sanctum` middleware

---

## Phase 5 — Attendance (Client Side)

- [ ] Generate `AttendanceController`: `php artisan make:controller AttendanceController`
- [ ] Implement `timeIn()` method
  - Check if user has already clocked in today
  - Create attendance record with `time_in = now()`
- [ ] Implement `timeOut()` method
  - Find today's open record (no `time_out`)
  - Update `time_out = now()`
- [ ] Implement `myRecords()` — return authenticated user's attendance list
- [ ] Protect all routes with `auth:sanctum`

---

## Phase 6 — Admin Middleware & Controllers

- [ ] Generate admin middleware: `php artisan make:middleware IsAdmin`
- [ ] Implement check: `auth()->user()->role !== 'admin'` → return 403
- [ ] Register middleware in `bootstrap/app.php` (Laravel 11) or `Kernel.php` (Laravel 10)
- [ ] Generate `AdminUserController`: `php artisan make:controller AdminUserController --api`
  - [ ] `index()` — list all users (with pagination)
  - [ ] `store()` — create a user
  - [ ] `show()` — view single user
  - [ ] `update()` — edit user details / role
  - [ ] `destroy()` — delete a user
- [ ] Generate `AdminAttendanceController`: `php artisan make:controller AdminAttendanceController --api`
  - [ ] `index()` — list all attendance records (filterable by user/date)
  - [ ] `store()` — manually create a record
  - [ ] `update()` — edit a record
  - [ ] `destroy()` — delete a record

---

## Phase 7 — API Routes

- [ ] Open `routes/api.php` and define all routes
- [ ] Public routes (no auth):
  - `POST /auth/register`
  - `POST /auth/login`
  - `POST /auth/forgot-password`
  - `POST /auth/reset-password`
- [ ] Authenticated routes (`auth:sanctum` middleware):
  - `POST /auth/logout`
  - `GET /profile`
  - `PUT /profile`
  - `POST /attendance/time-in`
  - `POST /attendance/time-out`
  - `GET /attendance`
- [ ] Admin routes (`auth:sanctum` + `is_admin` middleware):
  - `apiResource /admin/users`
  - `apiResource /admin/attendance`
- [ ] Verify routes list: `php artisan route:list`

---

## Phase 8 — Validation & Error Handling

- [ ] Add `Form Request` classes for cleaner validation (optional but recommended)
  - `php artisan make:request RegisterRequest`
  - `php artisan make:request AttendanceRequest`
- [ ] Return consistent JSON error responses
- [ ] Handle `ModelNotFoundException` globally in `bootstrap/app.php`
  - Return a clean 404 JSON message instead of HTML error page
- [ ] Make sure all responses use proper HTTP status codes:
  - `200` OK, `201` Created, `422` Validation Error, `401` Unauthorized, `403` Forbidden, `404` Not Found

---

## Phase 9 — Testing

- [ ] Download and set up Postman or Insomnia
- [ ] Test **Registration** — check token is returned
- [ ] Test **Login** — copy the bearer token
- [ ] Test **Profile** — use `Authorization: Bearer {token}` header
- [ ] Test **Time In** — verify record is created in DB
- [ ] Test **Time Out** — verify `time_out` is filled
- [ ] Test **Forgot Password** — check email is sent (Mailtrap inbox)
- [ ] Test **Reset Password** — verify password is changed
- [ ] Test **Admin login** — use a user with `role = admin`
- [ ] Test **Admin users CRUD** — create, view, update, delete users
- [ ] Test **Admin attendance CRUD** — create, view, update, delete records
- [ ] Test **403 guard** — call admin route with a regular user token
- [ ] Test **duplicate time-in** — clock in twice on the same day, expect 422

---

## Phase 10 — Final Cleanup

- [ ] Remove unused routes and controllers
- [ ] Add `api_only` response format (no HTML responses anywhere)
- [ ] Set `APP_DEBUG=false` in `.env` for production
- [ ] Review all `$fillable` arrays — make sure no sensitive fields are exposed
- [ ] Add pagination to admin list endpoints (`->paginate(15)`)
- [ ] Write a basic `README.md` documenting your endpoints

---

## Quick Reference

| Command | Purpose |
|---|---|
| `php artisan serve` | Start local dev server |
| `php artisan migrate` | Run all pending migrations |
| `php artisan migrate:fresh` | Drop all tables and re-migrate |
| `php artisan route:list` | See all registered routes |
| `php artisan make:controller X` | Create a controller |
| `php artisan make:model X -m` | Create model + migration together |
| `php artisan make:middleware X` | Create middleware |
| `php artisan tinker` | Interactive PHP shell for testing |

---

> **Tip for beginners:** Build and test one phase at a time. Don't move to the next phase until the current one is working. Use `php artisan tinker` to quickly test your models and relationships.
