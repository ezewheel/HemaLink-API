# HemaLink

HemaLink es una plataforma web que busca facilitar la conexión entre donantes de sangre y hospitales.

Permite publicar y buscar campañas de donación, reservar turnos y gestionar solicitudes según el rol de cada usuario: donante, solicitante, moderador o administrador.

El proyecto está dividido en dos repositorios:

- **HemaLink-API:** backend desarrollado con ASP.NET Core 8, C# y PostgreSQL.
- **HemaLink-Web:** frontend desarrollado con React 19 y Vite.

## Requisitos

- [.NET SDK 8](https://dotnet.microsoft.com/download/dotnet/8.0)
- PostgreSQL o Docker
- Node.js y npm
- `dotnet-ef` para inicializar la base de datos

Instalar `dotnet-ef`:

```bash
dotnet tool install --global dotnet-ef --version 8.0.19
```

## Instalación y ejecución

### Backend

#### Base de datos

La cadena de conexión se encuentra en `HemaLink-API/WebAPI/appsettings.json`:

```text
Host=localhost;Port=5432;Database=DonationsDb;Username=donations_user;Password=password
```

Para levantar PostgreSQL con Docker:

```bash
docker run --name hemalink-db \
  -e POSTGRES_DB=DonationsDb \
  -e POSTGRES_USER=donations_user \
  -e POSTGRES_PASSWORD=password \
  -p 5432:5432 -d postgres:16
```

Si ya tenés PostgreSQL instalado, podés crear la base de datos y el usuario manualmente:

```sql
CREATE USER donations_user WITH PASSWORD 'password';
CREATE DATABASE DonationsDb OWNER donations_user;
```

#### Configuración

El archivo `appsettings.json` contiene valores de ejemplo para JWT y el correo remitente.

Para habilitar el envío de correos mediante SendGrid, configurar la API key desde la carpeta `HemaLink-API`:

```bash
dotnet user-secrets set "SendGrid:ApiKey" "TU_API_KEY_DE_SENDGRID" --project WebAPI
```

Sin esta configuración, la API inicia normalmente, pero no se pueden enviar correos de confirmación, cancelación o actualización de solicitudes.

#### Migraciones y arranque

Desde la carpeta `HemaLink-API`, ejecutar:

```bash
dotnet restore WebAPI.sln
dotnet ef database update --project Infrastructure --startup-project WebAPI
dotnet run --project WebAPI --launch-profile http
```

La API queda disponible en `http://localhost:5165`.

Otros endpoints útiles:

- Swagger: http://localhost:5165/swagger
- Health check: http://localhost:5165/api/health

Swagger está disponible en el entorno Development.

#### Usuarios de prueba

Las migraciones cargan datos iniciales para probar el sistema, incluyendo usuarios, hospitales, donantes, campañas y turnos.

Algunas cuentas disponibles:

| Email | Rol |
| --- | --- |
| `admin@example.com` | Administrador |
| `mod@example.com` | Moderador |
| `heca@example.com` | Solicitante aprobado |

La contraseña de las cuentas de prueba es `string`.

#### Autenticación

El login devuelve un token JWT que debe enviarse en las solicitudes a endpoints protegidos:

```http
Authorization: Bearer <token>
```

Desde Swagger se puede ingresar el token utilizando el esquema de autorización `DonationsAPI`.

#### CORS

La configuración actual permite únicamente el dominio de producción.

Para ejecutar el frontend localmente, agregar `http://localhost:5173` a la lista de orígenes permitidos en `HemaLink-API/WebAPI/Program.cs`, dentro de `builder.WithOrigins(...)`.

### Frontend

Desde la carpeta `HemaLink-Web`, instalar las dependencias:

```bash
npm install
```

Crear un archivo `.env.local` en la raíz del proyecto:

```env
VITE_API_URL=http://localhost:5165
```

Iniciar el servidor de desarrollo:

```bash
npm run dev
```

La aplicación queda disponible en `http://localhost:5173`.

Otros comandos:

```bash
npm run build
npm run preview
npm run lint
```

- `build`: genera la versión de producción en `dist/`.
- `preview`: permite probar localmente el build.
- `lint`: ejecuta ESLint.

## Despliegue

El proyecto utiliza GitHub Actions para automatizar el despliegue.

- **Backend:** se despliega en Azure Web App (`hemalink-api`) al realizar un push a `master`. Workflow: `.github/workflows/master_hemalink-api.yml`.
- **Frontend:** se despliega en Azure Static Web Apps al realizar un push a `main`. Workflow: `.github/workflows/azure-static-web-apps-lively-stone-0a3327e0f.yml`.

La variable `VITE_API_URL` se configura en el workflow del frontend.

## Tecnologías

- C#
- .NET 8
- ASP.NET Core Web API
- PostgreSQL
- Entity Framework Core
- JWT y BCrypt
- SendGrid
- Polly
- Swagger
- React 19
- Vite
- React Router
- Context API
- CoreUI y Bootstrap
- GitHub Actions
- Azure Web App
- Azure Static Web Apps