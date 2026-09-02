# SafeTrace

SafeTrace is an open-source, full-stack Geographic Information System (GIS) platform for plantation supply-chain traceability. It consists of two independent repositories - a Django-based REST API backend and a Next.js frontend dashboard - that together allow organizations to manage farmer data, plantation plots, STDB (Surat Tanda Daftar Budidaya) documents, GAP (Good Agricultural Practice) compliance, and interactive map layers.

The project includes a case-study implementation for a farming cooperative in Sekadau Regency, West Kalimantan, Indonesia. 

This system was produced with the financial support of the European Union, the German Federal Ministry for Economic Cooperation and Development (BMZ) and the Ministry of Foreign Affairs of the Netherlands, as part of the SAFE Project, implemented by GIZ. Its contents are the sole responsibility of PT VBL and do not necessarily reflect the views of GIZ, the European Union, the German Federal Ministry for Economic Cooperation and Development (BMZ) and the Ministry of Foreign Affairs of the Netherlands

This repository provides a single reference for setting up and using both applications.

## Repositories

- [SafeTrace Backend](https://github.com/it-vbl/safetrace-backend) - Django REST API, geospatial processing, data storage, and business logic.
- [SafeTrace Frontend](https://github.com/it-vbl/safetrace-frontend) - Next.js dashboard, maps, tables, and role-based workflows.

## Project Overview

SafeTrace gives agricultural cooperatives, government agencies, and processing partners a shared, transparent view of where agricultural products come from and how they were produced. The backend exposes a REST API that stores and processes geospatial and compliance data, while the frontend renders that data as an interactive dashboard with maps, tables, and role-based workflows.

## Backend (Django)

The backend powers all data storage, geospatial processing, and business logic for the traceability system, including deforestation alerts, spatial data handling, and periodic synchronization with external data sources such as Google Earth Engine (GEE).

### Backend Dependencies

- Python 3.12
- GDAL library for shapefile processing
- Poppler for `pdf2image`
- PostgreSQL with the PostGIS extension
- Redis as the task queue backend
- Ubuntu 24.04 as the reference operating system

### System Dependencies (Ubuntu 24.04)

```bash
sudo apt update
sudo apt install -y python3-dev python3.12-dev
sudo apt install -y gdal-bin libgdal-dev
sudo apt install -y poppler-utils
sudo apt install -y postgresql postgresql-contrib postgis
sudo apt install -y redis-server
```

### Backend Setup

1. Clone the repository:

   ```bash
   git clone https://github.com/it-vbl/safetrace-backend.git
   cd safetrace-backend
   ```

2. Create and activate a virtual environment:

   ```bash
   python3.12 -m venv venv
   source venv/bin/activate
   ```

3. Install the GDAL Python bindings matching the installed system library:

   ```bash
   gdal-config --version
   pip install gdal==$(gdal-config --version)
   ```

4. Install the remaining Python dependencies:

   ```bash
   pip install -r requirements.txt
   ```

5. Import Indonesian administrative base data. Because the project depends on `django-wilayah-indonesia`, the administrative reference data for provinces, regencies, districts, and villages must be imported before first use:

   ```bash
   ./manage.py import_base_csv
   ```

6. Run the database migrations:

   ```bash
   python manage.py migrate
   ```

7. Start the development server:

   ```bash
   python manage.py runserver
   ```

8. Open [http://localhost:8000](http://localhost:8000) in your browser.

### Production Environment

To run the backend with production settings, export the following environment variable before starting the application:

```bash
export DJANGO_ENV=prod
```

This tells Django to load its configuration from `settings/prod.py` instead of the default development settings. Ensure that a valid production `.env` file is present in the project root.

### Production Setup

1. Clone the project onto the server. The recommended location is `/var/www/html`:

   ```bash
   cd /var/www/html
   sudo git clone https://github.com/it-vbl/safetrace-backend.git safetrace-backend
   cd safetrace-backend
   ```

2. Ensure that the production `.env` file is present in the project root.

3. Run the initial deployment script:

   ```bash
   sudo bash scripts/deploy_init.sh
   ```

4. Validate that the services started correctly:

   ```bash
   sudo systemctl status safetrace
   sudo systemctl status nginx
   ```

5. Configure a cron job for Google Earth Engine deforestation-data synchronization:

   ```bash
   sudo crontab -e
   ```

   Add a job such as the following to run the synchronization daily at 02:00:

   ```cron
   0 2 * * * cd /var/www/html/safetrace-backend && DJANGO_ENV=prod /var/www/html/safetrace-backend/venv/bin/python manage.py get_data_gee_deforestation >> /var/log/safetrace_gee_deforestation.log 2>&1
   ```

6. For subsequent releases and updates, run:

   ```bash
   sudo bash scripts/deploy_update.sh
   ```

### Backend Troubleshooting

#### GDAL version mismatch

If you see an error similar to:

```text
Python bindings of GDAL X.X.X require at least libgdal X.X.X
```

the Python bindings do not match the installed GDAL system library. Check the installed version and install matching bindings:

```bash
gdal-config --version
pip install gdal==$(gdal-config --version)
```

Ubuntu 24.04 servers typically use GDAL 3.8.4 or newer.

#### Missing Python headers

If you see `Python.h: No such file or directory`, install the Python development headers:

```bash
sudo apt install python3-dev python3.12-dev
```

#### Deployment scripts

The deployment helpers automatically install the correct GDAL version detected from the system library:

```bash
# Initial deployment
sudo bash scripts/deploy_init.sh

# Update an existing deployment
sudo bash scripts/deploy_update.sh
```

## Frontend (Next.js)

The frontend is an open-source GIS dashboard that consumes the backend REST API. It can be connected to any compatible backend to manage farmer data, plantation data, STDB documents, GAP compliance records, and interactive map layers.

### Key Features

- **Interactive GIS map dashboard** - visualizes plantation plot polygons, deforestation alerts, spatial filters, and custom map layers using Leaflet.
- **Traceability management** - handles farmer records, plantation plots, sales data, and GAP compliance covering production practices, pesticide use, fertilizer use, and hazardous (B3) waste handling.
- **Settings and role-based access control** - supports user management, user profiles, permission-based roles (Admin, Farmer Group Leader, Government Agency, and Factory Partner), and map-layer configuration.

### Frontend Tech Stack

- **Framework:** Next.js 15 with App Router and Turbopack
- **UI:** React 19 and Tailwind CSS
- **State management:** Redux Toolkit
- **Mapping:** Leaflet, React Leaflet, and Turf.js
- **Tables and charts:** AG Grid, Chart.js, and D3
- **Forms:** Formik and Yup
- **API client:** Axios
- **UI components:** Atomic Design principles with Radix UI

### Frontend Prerequisites

- Node.js 18 or newer; version 20 or later is recommended
- npm; pnpm and yarn are also supported

### Frontend Setup

1. Clone the repository:

   ```bash
   git clone https://github.com/it-vbl/safetrace-frontend.git
   cd safetrace-frontend
   ```

2. Install dependencies:

   ```bash
   npm install
   ```

3. Copy the example environment file:

   ```bash
   cp .env.local.example .env
   ```

   Open `.env` and set `NEXT_PUBLIC_BASE_URL` to the base URL of the backend REST API. This value is required.

4. Run the application:

   ```bash
   npm run dev
   ```

The application will be available at [http://localhost:3000](http://localhost:3000).

### Available Scripts

- `npm run dev` - starts the development server using Turbopack.
- `npm run build` - creates an optimized production build.
- `npm run start` - serves the production build.
- `npm run lint` - runs ESLint to check the codebase for potential errors.

### Frontend Project Structure

The frontend uses an Atomic Design architecture to keep its component library scalable and maintainable.

```text
src/
├── app/             # Main Next.js App Router directory (pages and layouts)
├── assets/          # Project-specific SVG icons and images
├── components/      # Reusable UI components (Atomic Design)
│   ├── atoms/       # Smallest building blocks (Button, Input, Icon)
│   ├── molecules/   # Combinations of atoms (Form Field, Card Header)
│   ├── organisms/   # Groups of molecules (Modal, Complex Form, Header)
│   ├── providers/   # Context providers (Redux, Toast, Theme)
│   └── ui/          # Components sourced from Radix UI or Shadcn
├── config/          # Theme configuration and base assets
├── constants/       # Global static variables and enums
├── hooks/           # Custom React hooks
├── i18n/            # Localization configuration and translation data
├── libs/            # Utility libraries and permission helpers
├── services/        # REST API integrations implemented with Axios
├── store/           # Redux state management configuration and slices
├── styles/          # Global CSS and Tailwind base styles
├── types/           # TypeScript types and interfaces
└── utils/           # Formatters, environment helpers, and general utilities

public/              # Publicly served logos, backgrounds, and icons
```

### Theme and Logo Customization

1. **Brand colors:** Update theme colors in `src/config/brand.ts`. The corresponding Tailwind classes are generated automatically from this file.
2. **Logo and assets:** Update institution-logo, login-background, and other asset paths in `src/config/assets.ts`, then place the physical files in the `/public` directory.

## Documentation

- [SafeTrace README (PDF)](./Safetrace%20-%20Readme.pdf)
- [SafeTrace User Guide (PDF)](./Safetrace%20-%20User%20Guide.pdf)
- [WHISP Risk Assessment - English (PDF)](./Whisp_Risk_Assessment_English.pdf)
- [Forest Baseline and Loss Alerts - English (PDF)](./CUKK_Forest_Baseline_and_Loss_Alert_English.pdf)

## License

The frontend and backend are released under the MIT License. See the `LICENSE` file in each repository for full details.
