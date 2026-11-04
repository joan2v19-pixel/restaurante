# 🍽️ Sistema de Reservas de Restaurante

Aplicación web para gestionar un restaurante de principio a fin: **mesas, reservas, pedidos, cocina y despachos**, con inicio de sesión por roles. Está construida con HTML, CSS y JavaScript puro, sin dependencias, sin *build* y sin backend: todo corre en el navegador y los datos se guardan **cifrados** en `LocalStorage`.

> ⚠️ **Proyecto de demostración.** Al no tener servidor, las contraseñas, sesiones y permisos se validan en el navegador, por lo que **no ofrecen seguridad real**. Consulta la sección [Seguridad y limitaciones](#-seguridad-y-limitaciones) antes de usarlo en un entorno real.

---

## 📑 Tabla de contenidos

- [Características](#-características)
- [Tecnologías](#-tecnologías)
- [Estructura del proyecto](#-estructura-del-proyecto)
- [Puesta en marcha](#-puesta-en-marcha)
- [Usuarios de demostración](#-usuarios-de-demostración)
- [Roles y permisos](#-roles-y-permisos)
- [Flujo de trabajo](#-flujo-de-trabajo)
- [Reglas de negocio](#-reglas-de-negocio)
- [Almacenamiento de datos](#-almacenamiento-de-datos)
- [Inicio de sesión con Google (opcional)](#-inicio-de-sesión-con-google-opcional)
- [Personalización](#-personalización)
- [Seguridad y limitaciones](#-seguridad-y-limitaciones)
- [Mejoras sugeridas](#-mejoras-sugeridas)

---

## ✨ Características

- **Panel principal** con cuatro indicadores: reservas de hoy, platos pendientes, despachos activos y mesas ocupadas, además de las últimas reservas y pedidos.
- **Mesas**: 8 mesas precargadas con distinta capacidad; el administrador puede editar capacidad y estado.
- **Reservas**: creación y cancelación con validación de fecha, capacidad y conflictos de horario.
- **Pedidos**: carrito de platos por mesa, con edición mientras nada haya entrado a cocina.
- **Cocina**: tablero por estados (*Pendiente → En preparación → Listo*).
- **Despachos**: agrupa platos listos y sigue su entrega (*Pendiente → En ruta → Entregado*).
- **Roles y permisos** (Administrador, Mesero, Cocina, Despacho) verificados en **cada operación**, no solo al ocultar botones.
- **Datos cifrados** con AES-GCM de 256 bits (Web Crypto API) y **sesión con expiración** de 120 minutos.
- **Protección contra XSS**: todo texto del usuario se escapa antes de insertarse con `innerHTML`.
- **Diseño responsive** con menú lateral y puntos de corte en 768 px y 480 px.
- **Datos de demostración** restablecibles con un clic.
- **Inicio de sesión con Google** opcional (pendiente de configurar).

## 🛠️ Tecnologías

| Área | Tecnología |
| --- | --- |
| Marcado | HTML5 |
| Estilos | CSS3 (variables CSS, Flexbox/Grid, *media queries*) |
| Lógica | JavaScript (ES6+) sin frameworks ni librerías |
| Cifrado | Web Crypto API (`AES-GCM`) |
| Persistencia | `LocalStorage` (datos) y `SessionStorage` (sesión) |
| Autenticación externa | Google Identity Services (opcional) |

## 📁 Estructura del proyecto

```
sistema-reservas-restaurante/
├── index.html          # Pantalla de login, layout principal, modal y carga de scripts
├── css/
│   └── styles.css      # Variables de color, login, menú, tarjetas, tablas, formularios,
│                       # etiquetas de estado, modales, mensajes y estilos móviles
└── js/
    ├── storage.js      # Cifrado, lectura/escritura, validación de estructura,
    │                   # datos de demostración, hash de contraseñas y contadores
    ├── auth.js         # Login, sesión con expiración, roles/permisos y Google Sign-In
    ├── modules.js      # Lógica de negocio: mesas, reservas, pedidos, cocina,
    │                   # despachos, estadísticas y reinicio de datos
    └── app.js          # Arranque, navegación, renderizado de vistas y modales
```

**El orden de los scripts importa** y está definido en `index.html`:

```
storage.js  →  auth.js  →  modules.js  →  app.js
```

Cada archivo depende de las funciones globales del anterior (no se usan módulos ES ni bundler).

## 🚀 Puesta en marcha

No requiere instalación ni dependencias.

1. Descomprime el proyecto.
2. Sirve la carpeta con un servidor local (recomendado) y abre la URL en el navegador:

   ```bash
   cd sistema-reservas-restaurante

   # Opción A: Python
   python3 -m http.server 8000

   # Opción B: Node.js
   npx serve .
   ```

   Luego entra a `http://localhost:8000`.

3. Inicia sesión con alguno de los [usuarios de demostración](#-usuarios-de-demostración).

> **Nota:** el cifrado usa la Web Crypto API, que exige un navegador actualizado y que la página se abra por **HTTPS o `localhost`**. Si el navegador no lo permite, la aplicación muestra un aviso en la pantalla de login.

## 👤 Usuarios de demostración

| Usuario | Contraseña | Rol |
| --- | --- | --- |
| `admin` | `admin123` | Administrador |
| `mesero` | `mesero123` | Mesero |
| `cocina` | `cocina123` | Cocina |
| `despacho` | `despacho123` | Despacho |

## 🔐 Roles y permisos

Cada rol ve únicamente las vistas para las que tiene permiso y solo puede ejecutar sus operaciones.

| Permiso | Administrador | Mesero | Cocina | Despacho |
| --- | :---: | :---: | :---: | :---: |
| Ver panel principal | ✅ | | | |
| Ver mesas | ✅ | ✅ | | |
| Gestionar mesas | ✅ | | | |
| Ver reservas | ✅ | ✅ | | |
| Crear reservas | ✅ | ✅ | | |
| Cancelar reservas | ✅ | ✅ | | |
| Ver pedidos | ✅ | ✅ | ✅ | |
| Crear / modificar pedidos | ✅ | ✅ | | |
| Ver cocina | ✅ | | ✅ | |
| Preparar platos | ✅ | | ✅ | |
| Ver despachos | ✅ | ✅ | | ✅ |
| Gestionar despachos | ✅ | | | ✅ |
| Ver usuarios | ✅ | | | |
| Restablecer datos | ✅ | | | |

## 🔄 Flujo de trabajo

```
 Reserva ──► Pedido ──► Cocina ──► Despacho ──► Entrega
(Mesero)    (Mesero)   (Cocina)   (Despacho)   (Despacho)
```

1. **Reserva**: el mesero reserva una mesa para una fecha y hora. La mesa pasa a *Reservada* cuando la reserva es para hoy.
2. **Pedido**: cuando los clientes llegan, se crea un pedido para la mesa. La reserva de hoy se marca como *Finalizada* y la mesa pasa a *Ocupada*.
3. **Cocina**: cada plato avanza de *Pendiente* a *En preparación* y luego a *Listo*.
4. **Despacho**: el personal de despacho agrupa platos *Listos* en un despacho y lo marca *En ruta*.
5. **Entrega**: al marcar el despacho como *Entregado*, sus platos pasan a *Entregado*. Cuando todos los platos del pedido están entregados, el pedido queda *Completado* y la mesa se libera.

## 📏 Reglas de negocio

### Mesas
- Su estado se recalcula automáticamente: **Ocupada** si tiene un pedido activo, **Reservada** si tiene una reserva confirmada para hoy, **Disponible** en cualquier otro caso.
- El administrador puede editar la capacidad (entero entre 1 y 30) y el estado.

### Reservas
- Estados: `Confirmada`, `Cancelada`, `Finalizada`. Solo se pueden cancelar las confirmadas.
- Todos los campos son obligatorios; el nombre del cliente debe tener entre 3 y 60 caracteres.
- Personas: entero entre 1 y 50, y no puede superar la capacidad de la mesa.
- No se permiten fechas pasadas ni fechas inexistentes (por ejemplo, 31 de febrero).
- No se puede reservar una mesa ocupada.
- Cada reserva dura **120 minutos** para detectar conflictos: no puede solaparse con otra reserva confirmada de la misma mesa y fecha.

### Pedidos
- Una mesa ocupada (con pedido activo) no puede recibir otro pedido.
- Máximo 30 líneas por pedido y entre 1 y 50 unidades por plato.
- Precio y subtotal se toman siempre del menú, no de lo que envíe la interfaz.
- Un pedido solo puede modificarse mientras **todos** sus platos estén en `Pendiente`.
- Estados del pedido: `Activo` y `Completado`.

### Cocina
- Transiciones permitidas de cada plato: `Pendiente → En preparación → Listo`. No se puede retroceder ni saltar estados.

### Despachos
- Solo se despachan platos en estado `Listo` que aún no pertenezcan a otro despacho (máximo 30 líneas por despacho).
- Si una sola línea seleccionada no es válida, no se guarda nada.
- Transiciones: `Pendiente → En ruta → Entregado`.

### Menú precargado

| Plato | Precio |
| --- | ---: |
| Ensalada César | $6.50 |
| Sopa del Día | $5.00 |
| Lomo al Horno | $12.90 |
| Pollo a la Parrilla | $10.50 |
| Pasta Alfredo | $9.75 |
| Pescado al Ajillo | $13.20 |
| Postre de Chocolate | $4.50 |
| Jugo Natural | $3.00 |

## 💾 Almacenamiento de datos

Toda la información vive en el navegador, cifrada con **AES-GCM (256 bits)** y un vector de inicio aleatorio en cada guardado. GCM además detecta cualquier alteración del contenido.

| Clave | Almacén | Contenido |
| --- | --- | --- |
| `sistemaReservasRestaurante` | `LocalStorage` | Usuarios, mesas, menú, reservas, pedidos, despachos y contadores |
| `sistemaReservasSesion` | `SessionStorage` | Sesión actual (usuario, rol, expiración) |

Detalles de funcionamiento:

- Al abrir la página se descifran los datos y se mantienen en memoria; cada cambio se vuelve a cifrar y se guarda.
- Las escrituras se encadenan en una **cola** para conservar el orden, ya que el cifrado es asíncrono.
- Si los datos no existen, están corruptos o no se pueden descifrar, se regeneran los datos de demostración.
- Los pedidos y despachos usan **números consecutivos** (`contadores`), y el resto de entidades usan identificadores únicos (`RES-…`, `PED-…`, `DES-…`, etc.).

Estructura general de los datos:

```jsonc
{
  "version": 1,
  "contadores": { "pedidos": 0, "despachos": 0 },
  "usuarios": [],      // nombreUsuario, contrasenaHash, nombreCompleto, rol
  "mesas": [],         // numero, capacidad, estado
  "platosDelMenu": [], // nombre, precio, descripcion
  "reservas": [],      // nombreCliente, idMesa, fecha, hora, cantidadPersonas, estado
  "pedidos": [],       // numero, idMesa, platos[], total, estado
  "despachos": []      // numero, idPedido, platos[], estado
}
```

Para empezar de cero, entra como `admin`, ve a **Usuarios** y pulsa **Restablecer datos de demostración** (se eliminan reservas, pedidos y despachos). También puedes borrar manualmente la clave `sistemaReservasRestaurante` desde las herramientas del navegador.

## 🔑 Inicio de sesión con Google (opcional)

Por defecto el botón de Google **no se dibuja** y la pantalla de login muestra que está pendiente de configuración. Para activarlo, edita `js/auth.js`:

1. Crea un **Client ID de OAuth 2.0** en [Google Cloud Console](https://console.cloud.google.com/) y autoriza el origen desde el que sirves la app (por ejemplo `http://localhost:8000`).
2. Colócalo en la configuración:

   ```js
   const CONFIGURACION_GOOGLE = {
       clientId: 'TU_CLIENT_ID.apps.googleusercontent.com'
   };
   ```

3. Autoriza correos y asígnales un rol de forma explícita:

   ```js
   const CORREOS_AUTORIZADOS_GOOGLE = [
       { correo: 'nombre@ejemplo.com', rol: 'Mesero' },
       { correo: 'otro@ejemplo.com',  rol: 'Cocina' }
   ];
   ```

Cómo se valida el acceso:

1. Se revisan localmente el emisor (`iss`), la audiencia (`aud`) y la expiración (`exp`) del token.
2. Se verifica el token contra el servicio de Google (`oauth2.googleapis.com/tokeninfo`).
3. Solo ingresan los correos de la lista, con el rol que se les asignó. **El acceso con Google nunca otorga roles de forma automática.**

> Decodificar un JWT en el cliente no equivale a verificar su firma. En producción esta verificación debe hacerse en un servidor.

## 🎛️ Personalización

| Qué | Dónde |
| --- | --- |
| Duración de una reserva (conflictos de horario) | `DURACION_DE_LA_RESERVA_EN_MINUTOS` en `js/modules.js` |
| Duración de la sesión (por defecto 120 min) | `DURACION_SESION_EN_MINUTOS` en `js/auth.js` |
| Permisos de cada rol | `PERMISOS_POR_ROL` en `js/auth.js` |
| Usuarios, mesas y menú iniciales | `crearUsuariosDeDemostracion`, `crearMesasDeDemostracion` y `crearPlatosDeDemostracion` en `js/storage.js` |
| Símbolo de moneda | `formatearMoneda` en `js/app.js` (actualmente `$`) |
| Colores y tema | Variables CSS en `:root` dentro de `css/styles.css` |
| Vistas y menú lateral | Objeto `VISTAS` en `js/app.js` |

> Si modificas los datos de demostración en `storage.js`, restablece los datos (o borra la clave de `LocalStorage`) para que los cambios se apliquen, ya que los datos existentes se conservan entre sesiones.

## 🛡️ Seguridad y limitaciones

Este proyecto **no tiene backend**, y eso condiciona su seguridad:

- **La clave de cifrado está en el código fuente** (`storage.js`). El cifrado evita que los datos queden en texto plano y que se alteren de forma casual desde las herramientas del navegador, pero cualquiera con acceso al código puede descifrarlos.
- **El hash de contraseñas es solo demostrativo**: se calcula en el navegador, no usa un algoritmo lento ni una clave secreta y puede reproducirse fuera de la aplicación.
- **Sesión y permisos se controlan en el cliente**, por lo que un usuario con conocimientos técnicos podría manipularlos.
- Los datos viven en un solo navegador: **no se comparten entre dispositivos ni usuarios**.

Medidas que sí incluye:

- Cifrado AES-GCM de datos y sesión con detección de alteraciones.
- Expiración automática de sesión.
- Comprobación de permisos dentro de cada función de negocio (no solo en la interfaz).
- Validación de estructura al leer los datos guardados.
- Escape de HTML para prevenir XSS.
- Mensajes de login genéricos (no revelan si el usuario existe).

**Para producción** se necesita un servidor que valide credenciales (con `bcrypt`, `scrypt` o `argon2`), gestione sesiones, verifique permisos y guarde los datos en una base de datos.

## 🗺️ Mejoras sugeridas

- Backend con API REST y base de datos (autenticación, autorización y persistencia reales).
- Gestión del menú (alta, edición y baja de platos) desde la interfaz.
- Gestión de usuarios (crear, editar y desactivar).
- Reportes e historial (ventas por día, platos más pedidos).
- Impresión de comandas y cuentas.
- Pruebas automatizadas de la lógica de `modules.js`.

## 📄 Licencia

Por definir.
