# GeDocs - Document Management System with PQRS

GeDocs is a web platform to manage institutional documents and to handle PQRS (petitions, complaints, claims and suggestions) in a centralized and traceable way. It was developed as a training project for SENA (Servicio Nacional de Aprendizaje, Colombia).

The backend is built with Laravel 12 and the interface with React rendered through Inertia.js. The user interface is in Spanish.

Full documentation is in the project wiki: <https://github.com/Joalregra/GeDocs/wiki>

## Table of contents

- [Main features](#main-features)
- [Technology stack](#technology-stack)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Environment configuration](#environment-configuration)
- [Running the project](#running-the-project)
- [Roles](#roles)
- [Project structure](#project-structure)
- [Scripts and useful commands](#scripts-and-useful-commands)
- [Testing](#testing)
- [Common issues](#common-issues)
- [Documentation](#documentation)
- [License](#license)

## Main features

- Registration with approval, login and password recovery.
- Four roles: `Admin`, `Instructor`, `Aprendiz` and `Dependencia`.
- Administration of training groups (sheets), their users and their dependencies.
- File explorer with a default archival folder tree per year, upload with standardized file names and SHA256 hash, trash and restore, move, and download as single file or ZIP.
- PQRS: creation with attachments, assignment to a dependency, response deadlines of 10, 15 or 30 days, response letters in PDF merged with supporting files and sent by email, automatic archiving of expired requests.
- PDF builder for letters, circulars, acts, reports and certificates.
- Excel register of received and sent PQRS.
- Notifications for new user registrations.

Planned but not implemented: conversion to PDF/A, duplicate detection by hash, automatic acts, deadline blocking, audit history and deadline notifications. See the wiki page "Modules" for the complete list.

## Technology stack

### Backend

| Package | Version |
|---|---|
| PHP | ^8.2 |
| Laravel | ^12.0 |
| Inertia Laravel | ^2.0 |
| Laravel Sanctum | API authentication |
| Spatie Laravel Permission | ^6.23 (roles) |
| Laravel DomPDF | ^3.1 (PDF rendering) |
| Webklex Laravel PDFMerger | ^1.3 (PDF merging) |
| Simple QrCode | ^4.2 |
| Maatwebsite Excel | ^3.1 |
| Tightenco Ziggy | ^2.0 (routes in the frontend) |
| Laraveles Spanish | ^1.5 |

### Frontend

| Package | Version |
|---|---|
| React, React DOM | ^18.2 |
| @inertiajs/react | ^2.2.15 |
| Vite | ^7.0.7 |
| Tailwind CSS | ^4.1.16 (through PostCSS) |
| daisyUI | ^5.3.10 |
| Headless UI | ^2.0 |
| Heroicons | ^2.2 |
| Axios | ^1.13.2 |
| Sonner | ^2.0.7 (toasts) |
| pdfjs-dist | ^5.5.207 (PDF thumbnails) |
| @szhsin/react-menu | ^4.5.1 |

### Database

MySQL 8.

### Development and testing tools

Laravel Breeze, Laravel Pint, Laravel Pail, Laravel Sail (declared, not configured), Pest 3.8, Vitest 4.1, Testing Library, jsdom, Concurrently.

## Prerequisites

- PHP 8.2 or newer with the usual Laravel extensions (mbstring, openssl, pdo_mysql, zip, gd, fileinfo)
- Composer 2.x
- Node.js 18 or newer and npm 9 or newer
- MySQL 8.x
- Git

## Installation

```bash
git clone https://github.com/Joalregra/GeDocs.git
cd GeDocs
composer install
npm install
```

All required PHP packages are already declared in `composer.json`. Do not run `composer require` or `vendor:publish` for them.

## Environment configuration

```bash
cp .env.example .env
php artisan key:generate
```

On Windows PowerShell use `Copy-Item .env.example .env`.

Create the MySQL database and adjust these values in `.env`:

| Variable | Default in `.env.example` | Notes |
|---|---|---|
| `APP_URL` | `http://localhost:8000` | |
| `DB_CONNECTION` | `mysql` | |
| `DB_DATABASE` | `gedocs` | Create it before migrating |
| `DB_USERNAME`, `DB_PASSWORD` | `root`, empty | Change for your server |
| `MAIL_MAILER` | `log` | Emails are written to the log. Configure SMTP to deliver them |
| `SANCTUM_STATEFUL_DOMAINS` | `localhost:8000,localhost,127.0.0.1` | Must contain the host used in the browser |

Create the tables, seed the roles and link the storage:

```bash
php artisan migrate
php artisan db:seed
php artisan storage:link
```

`db:seed` creates the roles and one administrator account (`obuitragon@sena.edu.co`). The initial password is defined in `database/seeders/DatabaseSeeder.php`. Change it immediately after the first login.

## Running the project

```bash
composer run dev
```

This starts the backend (`php artisan serve`), the queue listener and Vite. The application is available at <http://localhost:8000>.

The scheduled task that archives expired PQRS needs the Laravel scheduler. In development run `php artisan schedule:work` in a separate terminal. In production add the cron entry `* * * * * php /path/to/artisan schedule:run`.

## Roles

| Role | Description |
|---|---|
| `Admin` | Full administration. Seeded |
| `Instructor` | Manages the sheets assigned to them and approves apprentice registrations |
| `Aprendiz` | Learner linked to a sheet |
| `Dependencia` | Staff of a dependency that answers PQRS. Created by an Admin |

New accounts registered through the interface start as `pending` until an Admin (for instructors) or an Instructor (for apprentices) approves them.

## Project structure

```
app/
  Exports/                 Excel report
  Http/Controllers/
    Api/                   PQRController, CommunicationController
    Auth/                  Breeze authentication
    DependencyController, ExcelEspreadSheetsReports, ExplorerController,
    FolderController, NotificationController, PdfController, ProfileController,
    SheetController, SheetUserController, UserController
  Http/Middleware/         HandleInertiaRequests, RedirectIfNotPermission
  Mail/                    PQRResponseMail
  Models/                  User, Sheet_number, Dependency, Folder, File, PQR,
                           comunication, AttachedSupport
  Notifications/           NewUserRegistered
  Services/                FolderStructureService, PqrAutoArchiveService
config/                    Standard Laravel and package configuration
database/                  migrations, seeders, factories
lang/, resources/lang/     Laravel language files (not used by the React interface)
public/                    Logos and images, build output, storage link
resources/
  css/app.css              Tailwind CSS and the daisyUI theme
  js/                      Pages, Components, context, Hooks, Layouts, lib, tests
  views/                   Inertia root view, PDF template, email template
routes/                    web.php, api.php, auth.php, console.php
tests/                     Pest tests
```

## Scripts and useful commands

| Command | Purpose |
|---|---|
| `npm run dev` | Vite in development mode |
| `npm run build` | Build the assets for production |
| `composer setup` | Install, copy `.env`, generate the key, migrate and build (does not seed, link storage or create the database) |
| `composer run dev` | Backend, queue listener and Vite in parallel |
| `composer test` | Backend tests (Pest) |
| `npm run test:unit` | Frontend tests (Vitest), single run |
| `php artisan optimize:clear` | Clear caches |
| `php artisan migrate:fresh --seed` | Rebuild the database (destroys all data) |
| `php artisan pqrs:auto-archive-expired` | Run the archiving of expired PQRS manually |

## Testing

Backend tests use Pest with an in memory SQLite database. Frontend tests use Vitest with jsdom. See the wiki page "Testing" for the inventory and history.

## Common issues

- Blank screen: `npm run dev` must be running, or run `npm run build`.
- API calls answer `401` in the browser: `SANCTUM_STATEFUL_DOMAINS` and `APP_URL` must match the host and port used in the browser.
- Permission errors on upload or PDF generation: check write permission on `storage` and `bootstrap/cache` and run `php artisan storage:link`.
- Emails do not arrive: `MAIL_MAILER=log` writes them to `storage/logs/laravel.log`.
- Response letter or Excel export fails: use MySQL. Some queries are not compatible with SQLite.
- Vite errors after switching branches: remove `node_modules` and run `npm install`.

## Documentation

The wiki contains the architecture, data model, roles, modules, complete API reference, test inventory, user story status and the list of known issues: <https://github.com/Joalregra/GeDocs/wiki>

## License

`composer.json` declares the MIT license. The repository does not include a `LICENSE` file yet.
