# RestoApp · Sistema de reservas

Aplicación web educativa para administrar mesas, reservas, pedidos, cocina y despachos. Está dividida en `index.html`, `css/` y `js/`, como pide la actividad. No requiere instalar paquetes.

## Iniciar

Se necesita Node.js. Abre una terminal en esta carpeta y ejecuta:

```text
node servidor.cjs
```

Luego abre [http://127.0.0.1:5500](http://127.0.0.1:5500). El servidor solo escucha en tu propio equipo. También puedes abrir la carpeta con Visual Studio Code y usar Live Server.

## Usuarios de demostración

| Usuario | Contraseña | Rol |
| --- | --- | --- |
| `admin` | `admin123` | Administrador |
| `mesero` | `mesero123` | Mesero |
| `cocina` | `cocina123` | Cocina |
| `despacho` | `despacho123` | Despacho |

## Funciones

- **Administrador:** panel, mesas, reservas, pedidos, cocina, despachos y administración.
- **Mesero:** panel, mesas, reservas, pedidos y despachos.
- **Cocina:** panel y avance de platos pendientes a preparación y listos.
- **Despacho:** panel y seguimiento de entregas pendientes, en ruta y entregadas.
- Ocho mesas y ocho platos demo precargados.
- Los datos operativos persisten en `localStorage`. Administración permite restablecerlos.
- Un pedido libera la mesa cuando todos sus platos se han entregado y se cierra.

## Activar inicio con Google (opcional)

El botón de Google requiere una configuración propia de Google Cloud y conexión a Internet. El Client ID no viene en el ZIP, así que el inicio demo funciona desde el primer uso y Google queda pendiente de configurar.

1. Crea credenciales OAuth de tipo **Aplicación web** en Google Cloud Console.
2. En **Orígenes de JavaScript autorizados**, registra `http://127.0.0.1:5500`.
3. Copia el Client ID en `js/config.js`, reemplazando `TU_CLIENT_ID.apps.googleusercontent.com`.
4. En `GOOGLE_EMAIL_ROLES`, agrega cada correo autorizado y su rol, por ejemplo:

   ```js
   export const GOOGLE_EMAIL_ROLES = {
     "admin@ejemplo.com": "administrador",
     "cocina@ejemplo.com": "cocina",
   };
   ```

   Los roles válidos son `administrador`, `mesero`, `cocina` y `despacho`. Guarda el archivo y recarga la página.

La aplicación recibe la credencial de Google Identity Services, verifica el ID token con el endpoint público de Google y comprueba que el correo esté verificado y tenga un rol autorizado. La credencial del token no se guarda en el navegador.

## Medidas y límites de seguridad

- Las contraseñas demo se comparan con hashes PBKDF2-SHA-256 y el navegador las procesa con Web Crypto. La sesión de trabajo queda en `sessionStorage`.
- Los datos de negocio se guardan en `localStorage` como JSON y se validan al leerlos. La interfaz escapa el contenido mostrado y comprueba permisos también al ejecutar acciones.
- Como no hay servidor, una persona con acceso al navegador puede modificar los archivos, las sesiones y los datos locales, o saltarse los permisos del cliente. Por eso las cuentas y roles son solo demostrativos; no guardes información real ni uses esta aplicación para proteger un restaurante en producción.
- Los precios del menú son valores de demostración tomados del prototipo original; no se especificó una moneda.
