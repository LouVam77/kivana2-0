# KIVANA · Tienda de cuidado personal

Aplicación web hecha con **HTML, CSS, JavaScript, React (Vite)**, **Firebase** (cuentas y pedidos) y
**MongoDB con Mongoose** (catálogo de 20 kits y 186 productos).

```
Navegador (React)  ──/api/kits──▶  Servidor Express + Mongoose  ──▶  MongoDB (kits, productos)
       │
       └──────── Firebase ──▶ Authentication (cuentas) + Firestore (usuarios, pedidos y recibos)
```
Cada página del PDF de diseño es una vista de la app:

| Vista del PDF | Ruta en la app | Archivo principal |
|---|---|---|
| 1. Inicio | `/` | `src/pages/Inicio.jsx` |
| 2. Detalle del kit | `/producto/:id` | `src/pages/DetalleProducto.jsx` |
| 3. Carrito desplegable | (se abre con el botón negro `COP/ $`) | `src/components/CartDrawer.jsx` |
| 4. Favoritos | `/favoritos` (corazón del menú) | `src/pages/Favoritos.jsx` |
| 5. Iniciar sesión / Registrarse | `/cuenta` sin sesión | `src/pages/cuenta/AccesoOpciones.jsx` |
| 6. Crear nueva cuenta | `/cuenta/registro` | `src/pages/cuenta/Registro.jsx` |
| (nuevo) Iniciar sesión | `/cuenta/ingresar` | `src/pages/cuenta/IniciarSesion.jsx` |
| 7. Mis datos y pedidos | `/cuenta` con sesión | `src/pages/cuenta/Perfil.jsx` |

---

## Paso a paso para que funcione

### 1. Instalar lo necesario (una sola vez)
1. Instala **Node.js LTS** (versión 22 o más reciente) desde https://nodejs.org
2. Instala **Visual Studio Code** desde https://code.visualstudio.com
3. Abre una terminal y comprueba la instalación:
   ```bash
   node -v     # debe mostrar v20.19 o mayor (ideal v22)
   npm -v
   ```

### 2. Abrir el proyecto e instalar dependencias
1. Descomprime `kivana.zip`.
2. En VS Code: **Archivo > Abrir carpeta...** y elige la carpeta `kivana`.
3. Abre la terminal de VS Code (**Terminal > Nueva terminal**) y ejecuta:
   ```bash
   npm install
   ```
   Esto descarga React, Vite, Firebase, React Router, EmailJS, Express y Mongoose en la carpeta `node_modules`.

> **Si ya tenías la versión anterior:** reemplaza los archivos por los nuevos pero **conserva tu archivo `.env`**,
> agrégale las líneas `MONGODB_URI=` y `PORT=4000` (están en `.env.example`) y vuelve a ejecutar `npm install`.

### 3. Primera prueba (todavía sin Firebase)
```bash
npm run dev
```
Se abre http://localhost:5173. Ya funcionan el inicio, el detalle del kit, los favoritos y el carrito.
El comando arranca dos cosas a la vez: `[web]` (la tienda) y `[api]` (el servidor de MongoDB). Hasta que hagas
el paso 7.1, `[api]` dirá que no pudo conectarse a MongoDB; es normal y la tienda usa el catálogo de respaldo.
La sección **Mis datos y pedidos** muestra un aviso amarillo porque falta conectar Firebase.
Para detener el servidor: `Ctrl + C` en la terminal.

### 4. Crear el proyecto en Firebase
1. Entra a https://console.firebase.google.com con tu cuenta de Google.
2. **Crear un proyecto** → nombre: `kivana` → puedes desactivar Google Analytics → **Crear proyecto**.
3. En la página del proyecto toca el ícono **`</>` (Web)** para registrar una app web.
   Ponle el apodo `kivana-web` (no marques Firebase Hosting por ahora) → **Registrar app**.
4. Firebase te muestra un bloque `firebaseConfig` con `apiKey`, `authDomain`, `projectId`, etc. Déjalo abierto.

### 5. Conectar la app con Firebase (archivo `.env`)
1. En la carpeta del proyecto, copia el archivo `.env.example` y renombra la copia a **`.env`**.
2. Pega cada valor de `firebaseConfig` en su línea, sin comillas:
   ```env
   VITE_FIREBASE_API_KEY=AIzaSy...
   VITE_FIREBASE_AUTH_DOMAIN=kivana-xxxxx.firebaseapp.com
   VITE_FIREBASE_PROJECT_ID=kivana-xxxxx
   VITE_FIREBASE_STORAGE_BUCKET=kivana-xxxxx.firebasestorage.app
   VITE_FIREBASE_MESSAGING_SENDER_ID=1234567890
   VITE_FIREBASE_APP_ID=1:1234567890:web:abc123
   ```
3. Guarda, detén el servidor (`Ctrl + C`) y vuelve a ejecutar `npm run dev` (Vite solo lee el `.env` al arrancar).

### 6. Activar el inicio de sesión (Authentication)
1. En la consola de Firebase: menú izquierdo → **Authentication** → **Comenzar**.
2. Pestaña **Método de acceso (Sign-in method)**:
   - **Correo electrónico/contraseña** → Habilitar → Guardar.
   - **Google** → Habilitar → elige tu correo de asistencia → Guardar.
3. En **Configuración > Dominios autorizados** ya aparece `localhost`. Si publicas la app en otro dominio, agrégalo ahí.

### 7. Crear la base de datos (Firestore)
1. Menú izquierdo → **Firestore Database** → **Crear base de datos**.
2. Si pregunta la edición, elige **Standard**. Ubicación: `southamerica-east1 (São Paulo)` (la más cercana a Colombia).
3. Elige **modo de producción** → **Crear**.
4. Abre la pestaña **Reglas**, borra lo que hay, pega todo el contenido del archivo `firestore.rules` del proyecto y toca **Publicar**.

Las colecciones `usuarios` y `pedidos` se crean solas cuando alguien se registra o compra.

### 7.1 Conectar MongoDB (catálogo de kits y productos)
Los kits y productos viven en MongoDB. Lo más fácil es **MongoDB Atlas**, gratis y en la nube:

1. Crea una cuenta en https://www.mongodb.com/cloud/atlas/register
2. **Create a cluster** → elige el plan **Free** → proveedor AWS, región `São Paulo (sa-east-1)` → **Create**.
3. **Database Access** → **Add New Database User** → usuario `kivana` y una contraseña solo con letras y números
   → rol **Read and write to any database** → **Add User**.
4. **Network Access** → **Add IP Address** → **Allow access from anywhere** (`0.0.0.0/0`) → **Confirm**.
   (Para practicar está bien; cuando publiques la tienda deja solo la IP de tu servidor.)
5. **Database** → botón **Connect** → **Drivers** → copia la cadena que empieza por `mongodb+srv://`.
6. Pégala en tu `.env`, cambia `<db_password>` por tu contraseña y escribe `/kivana` antes del `?`:
   ```env
   MONGODB_URI=mongodb+srv://kivana:TuClave123@cluster0.abcde.mongodb.net/kivana?retryWrites=true&w=majority
   PORT=4000
   ```
7. Carga el catálogo (20 kits y 186 productos) en la base de datos:
   ```bash
   npm run seed
   ```
   Debe decir `✔ 186 productos guardados` y `✔ 20 kits guardados`.
8. Arranca todo con `npm run dev`. En la terminal verás `[api] ✔ Conectado a MongoDB` y
   `[web] Local: http://localhost:5173`.

**¿Prefieres MongoDB en tu PC?** Instala *MongoDB Community Server* y usa `MONGODB_URI=mongodb://127.0.0.1:27017/kivana`.

**Si MongoDB no está conectado** la tienda no se cae: usa el catálogo local de respaldo
(`server/data/catalogo.js`) y, mientras programas, muestra un aviso amarillo en el inicio.

**Cambiar precios, descuentos o productos:** edita `server/data/catalogo.js` y vuelve a ejecutar `npm run seed`
(actualiza lo que existe, no duplica). También puedes cambiar `descuento` o `precioRegular` de un kit
directamente en Atlas (**Browse Collections → kits**); el precio final se calcula solo.

| Colección | Qué guarda |
|---|---|
| `productos` | nombre, marca, presentación, precio de referencia, foto, registro INVIMA |
| `kits` | nombre, público (mujer/hombre/unisex), precio regular, % de descuento, grupos "Elige tu…" con los productos |

### 8. (Opcional) Enviar el código de 4 dígitos al correo con EmailJS
Firebase no envía códigos numéricos, por eso el formulario usa **EmailJS** (plan gratis: 200 correos al mes).
Si no lo configuras, la app funciona en **modo demostración**: el código aparece en pantalla.

1. Crea una cuenta en https://www.emailjs.com
2. **Email Services → Add New Service → Gmail** → conecta tu Gmail → copia el **Service ID**.
3. **Email Templates → Create New Template**:
   - En **To Email** escribe: `{{to_email}}`
   - **Subject**: `Tu código de KIVANA`
   - **Content**: `Hola {{to_name}}, tu código de confirmación es {{codigo}}. Vence en {{minutos}} minutos.`
   - Guarda y copia el **Template ID**.
4. **Account → General** → copia la **Public Key**.
5. Pega los tres datos en el `.env`:
   ```env
   VITE_EMAILJS_SERVICE_ID=service_xxx
   VITE_EMAILJS_TEMPLATE_ID=template_xxx
   VITE_EMAILJS_PUBLIC_KEY=xxxxxxxx
   ```
6. Reinicia `npm run dev`.

### 9. Probar todo el recorrido
1. **Inicio** → toca el ♡ de un kit → toca el ♡ del menú: aparece en **Favoritos**.
2. **Agregar** → elige un producto en cada grupo → **Agregar** → se abre el carrito.
3. Agrega 2 kits o más y la barra del carrito marca **ENVÍO GRATIS** (1 kit paga $12.000, se cambia en `src/config/tienda.js`).
4. **Ir a pagar** sin sesión → sale el aviso *"Antes de pagar, inicia sesión"* → **Registrarme** → llena el
   formulario (la dirección es opcional aquí) → **Finalizar**. El carrito se vuelve a abrir solo.
5. **Ir a pagar** → revisa o escribe la dirección de envío (departamento, ciudad, barrio o vereda y especificaciones)
   y el teléfono → **Pagar**.
6. Sale *"¡Gracias por tu compra! Tu pedido se despachará pronto"*. El pedido queda guardado en Firestore y el
   recibo aparece en **Mis datos y pedidos → Pedidos pendientes** (en verde).

### 10. Manejar los pedidos como administrador
Los estados van en este orden: **Pedido recibido → Preparando → Despachado → Entregado** (o Cancelado).
El cliente los ve en su recibo en tiempo real, con la fecha de cada paso.

**Opción A: con el panel `/admin` (recomendada)**
1. Vuelve a publicar las reglas: copia todo `firestore.rules` en Firebase → Firestore Database → **Reglas** → **Publicar**.
2. Entra a la tienda con tu cuenta y abre `http://localhost:5173/admin`. Te va a decir que no eres
   administrador y te muestra tu **UID** (también está en Firebase → Authentication, columna "UID de usuario").
3. En Firebase → Firestore Database → **Iniciar colección** → ID de colección `admins` → **ID del documento**:
   pega tu UID → agrega un campo `rol` (string) = `admin` → Guardar.
4. Recarga `/admin`. Ya ves todos los pedidos. En "Mis datos y pedidos" también aparece el botón
   **Panel de pedidos (administrador)**.
5. Por cada pedido:
   - **Preparando**: cuando lo pidas al proveedor (Dropi).
   - Pega la **guía** y toca **Guardar guía**: el pedido pasa solo a **Despachado**.
   - **Entregado**: cuando Interrapidísimo lo entregue. El recibo se mueve a "Pedidos realizados".
   - **Avisar por WhatsApp** abre un mensaje listo con el estado y la guía.

**Opción B: a mano en la consola de Firebase**
Firestore Database → colección **`pedidos`** → abre el pedido:

| Quieres... | Haz esto en el documento |
|---|---|
| Mostrar la guía y el botón de rastreo | Escribe el número en el campo **`guia`** (string) y cambia **`estado`** a `despachado` |
| Pasarlo a **Pedidos realizados** | Cambia **`estado`** a `entregado` y agrega **`entregadoEn`** con tipo **timestamp** |

Valores válidos de `estado`: `pendiente`, `preparando`, `despachado`, `entregado`, `cancelado`.

### 11. Poner las fotos de los kits y productos
Los espacios para las fotos ya están listos. Solo guarda cada imagen con el nombre exacto:
- Kits: `public/img/kits/<slug-del-kit>.jpg` (ej: `public/img/kits/kit-rostro.jpg`)
- Productos: `public/img/productos/<slug-del-producto>.jpg` (ej: `public/img/productos/cerave-contorno.jpg`)

La lista completa de los 206 nombres está en `public/img/LISTA_DE_FOTOS.txt`. Mientras falte una foto, la tienda
muestra el recuadro vacío o "FOTO", igual que en el diseño.

### 12. (Opcional) Publicar la tienda en internet
Ahora la tienda tiene un servidor (Express + MongoDB), así que necesita un hosting que ejecute Node.js,
por ejemplo **Render** (tiene plan gratis):
1. Sube la carpeta a un repositorio de GitHub (sin el archivo `.env`).
2. En https://render.com → **New Web Service** → elige el repositorio.
3. Build command: `npm install && npm run build` · Start command: `npm start`.
4. En **Environment** agrega todas las variables de tu `.env` (las `VITE_...` y `MONGODB_URI`).
5. Render te da una dirección como `https://kivana.onrender.com`. Agrégala en Firebase →
   Authentication → Configuración → **Dominios autorizados**.

`npm start` sirve la tienda compilada (`dist`) y la API en el mismo puerto.

---

## Problemas comunes

| Mensaje / síntoma | Solución |
|---|---|
| `'vite' no se reconoce como un comando` | No corriste `npm install` dentro de la carpeta `kivana`. |
| Sigue saliendo "Falta conectar Firebase" | El archivo debe llamarse exactamente `.env` (no `.env.txt`) y hay que reiniciar `npm run dev`. |
| "Este método de inicio de sesión no está activado" | Activa Correo/contraseña y Google en Authentication (paso 6). |
| "No tienes permiso para esta acción" | Publica las reglas de `firestore.rules` (paso 7). |
| En `/admin` sale "no es administradora" | Falta el documento `admins/<tu UID>` en Firestore, o el UID quedó con un espacio (paso 10). |
| Aviso amarillo "Catálogo local de respaldo" | La API no se conectó a MongoDB: revisa `MONGODB_URI`, la contraseña y Network Access en Atlas. |
| `npm run seed` dice `bad auth` | Usuario o contraseña de MongoDB mal escritos en `MONGODB_URI`. |
| `npm run seed` se queda esperando y falla | Tu IP no está permitida en Atlas → Network Access. |
| "Este dominio no está autorizado" | Agrega el dominio en Authentication > Configuración > Dominios autorizados. |
| La ventana de Google se cierra sola | El navegador bloqueó la ventana emergente; permítela o vuelve a intentar. |
| No llega el código al correo | Revisa spam y los 3 datos de EmailJS en `.env`. Sin ellos funciona el modo demostración. |

## Estructura de carpetas

```
kivana/
├── index.html               ← página base y fuentes de Google
├── package.json             ← dependencias y comandos (npm run dev / seed / build / start)
├── server/                  ← servidor de MongoDB
│   ├── index.js             ← API Express: /api/kits, /api/productos
│   ├── db.js                ← conexión con Mongoose
│   ├── seed.js              ← carga el catálogo en MongoDB (npm run seed)
│   ├── models/Kit.js        ← modelo de kit (precio, descuento, grupos)
│   ├── models/Producto.js   ← modelo de producto (nombre, marca, INVIMA)
│   └── data/catalogo.js     ← los 20 kits y 186 productos
├── .env.example             ← plantilla de las claves de Firebase y EmailJS
├── firebase.json            ← configuración de Hosting
├── firestore.rules          ← reglas de seguridad de la base de datos
├── public/img/kits/         ← fotos de los kits
├── public/img/productos/    ← fotos de los productos
└── src/
    ├── main.jsx             ← punto de entrada (proveedores de contexto)
    ├── App.jsx              ← rutas de cada vista
    ├── config/tienda.js     ← costo de envío, envío gratis, transportadora
    ├── firebase/config.js   ← conexión con Firebase
    ├── context/             ← sesión, catálogo, carrito, favoritos, avisos
    ├── services/            ← usuarios, pedidos y código por correo
    ├── components/          ← menú, tarjeta, carrito, recibo, dirección...
    ├── pages/               ← vistas (inicio, detalle, favoritos, cuenta)
    ├── utils/               ← precios, fechas, dirección, departamentos, errores
    └── styles/global.css    ← colores y medidas del diseño
```

**Sobre las medidas:** el diseño mide 819 px de ancho. En `global.css`, `1rem` equivale a 10 px del diseño y se
reduce automáticamente en pantallas más pequeñas, así la tienda se ve igual al PDF en cualquier celular.
# kivana2-0
