# SIDREP Google Doc Generator

Next.js App Router application that uploads a Chilean SIDREP PDF, extracts its text, uses the OpenAI API to produce structured data, lets the user edit the extracted fields and residue rows, and creates a formatted Google Doc with the Google Docs API.

## Stack

- Next.js App Router
- TypeScript
- Tailwind CSS
- OpenAI Node SDK
- pdf-parse
- Zod
- googleapis

## Project Structure

```text
app/
  api/
    extract/route.ts
    generate-doc/route.ts
  globals.css
  layout.tsx
  page.tsx
components/
  SidrepForm.tsx
lib/
  google-docs.ts
  sidrep-fields.ts
  sidrep-schema.ts
  sidrep-text.ts
.env.example
next.config.mjs
package.json
postcss.config.mjs
tailwind.config.ts
tsconfig.json
vercel.json
```

## Setup

1. Install dependencies:

```bash
npm install
```

2. Create `.env.local`:

```bash
cp .env.example .env.local
```

3. Add the required environment variables:

```bash
OPENAI_API_KEY=sk-...
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...
GOOGLE_REFRESH_TOKEN=...
```

`OPENAI_MODEL` is optional and defaults to `gpt-4.1-mini`.

## Google Credentials

1. Create or select a Google Cloud project.
2. Enable the Google Docs API.
3. Create an OAuth client.
4. Generate a refresh token for the Google account that should own the created documents.
5. Use a token with the `https://www.googleapis.com/auth/documents` scope.

The app uses the refresh token only on the server.

## OpenAI Extraction

The extraction route reads the uploaded PDF with `pdf-parse`, sends the extracted text to OpenAI using Structured Outputs, and validates the result with Zod before returning it to the frontend.

The extractor is tuned for real SIDREP text such as `Estatus: CERRADO, Nº Folio:2163231`, the `GENERADOR`, `TRANSPORTISTA`, and `DESTINATARIO` sections, and the flattened `Detalle de Declaración` table. It also sends deterministic hints to OpenAI to handle line breaks, irregular spacing, accents, uppercase/lowercase differences, and table rows split across multiple lines.

The schema contains:

- `numero_sidrep`
- `estatus`
- `fecha_generador_firma`
- `fecha_transportista_firma`
- `fecha_destinatario_firma`
- `generador_nombre`
- `generador_rut`
- `generador_identificacion`
- `generador_autoridad_sanitaria`
- `generador_direccion`
- `generador_comuna`
- `generador_responsable`
- `generador_email`
- `transportista_nombre`
- `transportista_rut`
- `transportista_identificacion`
- `transportista_autoridad_sanitaria`
- `transportista_direccion`
- `transportista_comuna`
- `transportista_responsable`
- `transportista_email`
- `destinatario_nombre`
- `destinatario_rut`
- `destinatario_identificacion`
- `destinatario_autoridad_sanitaria`
- `destinatario_direccion`
- `destinatario_comuna`
- `destinatario_responsable`
- `destinatario_email`
- `identificacion_transporte`
- `identificacion_acoplado`
- `cantidad_recibida_kg`
- `residuos[]`
- `cantidad_total_kg`
- `observaciones_generador`
- `observaciones_transportista`
- `observaciones_destinatario`
- `discrepancias`

Each `residuos[]` row contains:

- `descripcion_residuo`
- `codigo_principal`
- `codigo_secundario`
- `lista_a`
- `peligrosidad`
- `estado_fisico`
- `contenedor`
- `estado_residuo`
- `cantidad_kg`

## Run Locally

```bash
npm run dev
```

Open `http://localhost:3000`.

## Deploy to Vercel

1. Push this project to a Git repository.
2. Import it in Vercel.
3. Add the same environment variables in Vercel Project Settings.
4. Deploy.

The included `vercel.json` gives both API routes a 60-second maximum duration.

## API Routes

- `POST /api/extract`
  - Accepts multipart form data with a PDF in the `file` field.
  - Returns extracted SIDREP data, page count, and a text preview.

- `POST /api/generate-doc`
  - Accepts `{ "data": { ...sidrepFields } }`.
  - Creates a Google Doc titled `Resumen SIDREP Nº [folio]`.
  - Includes summary, Generador, Transportista, Destinatario, residue table, observations, discrepancies, and total kg transported.
