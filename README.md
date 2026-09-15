# Creando Tendencias — sistema operativo interno

Página de acceso y panel por divisiones. Frontend estático (HTML + JS), base de datos y autenticación en Supabase, despliegue en Netlify.

## Divisiones y módulos

| División | Módulos |
|---|---|
| Creando Tendencias | Clientes · Stock y almacén |
| Creando Hogares | Alquileres |

## Roles

| Rol | Alcance |
|---|---|
| `super_admin` | Acceso total a las dos divisiones y a todos los módulos. Ignora la tabla de permisos. |
| `admin` | Acceso a todo, menos eliminar. Los límites se ajustan en la tabla `permisos`, sin tocar código. |
| `staff` | Ver, crear y editar. |
| `usuario` | Solo lectura. |
| `cliente` | Solo su propia información. |

## Estructura del repositorio

```
index.html    página de acceso y panel
schema.sql    esquema completo de Supabase (se ejecuta una sola vez)
README.md     este archivo
```

## Puesta en marcha

### 1. Supabase

1. En el proyecto, abre **SQL Editor → New query**, pega el contenido de `schema.sql` y dale **Run**.
2. Ve a **Authentication → Users → Add user** y crea las dos cuentas (correo y contraseña). Marca *Auto Confirm User*.
3. Vuelve al SQL Editor y ejecuta, con los correos reales:

```sql
update perfiles set rol = 'super_admin'
where email in ('correo1@ejemplo.com', 'correo2@ejemplo.com');

insert into perfil_divisiones (perfil_id, division_id)
select p.id, d.id from perfiles p, divisiones d where p.rol = 'super_admin'
on conflict do nothing;
```

4. En **Project Settings → API** copia `Project URL` y `anon public key`.
5. Pégalos en `index.html`, arriba del todo del `<script>`:

```js
const SUPABASE_URL  = "https://xxxxx.supabase.co";
const SUPABASE_ANON = "eyJhbGciOi...";
```

> La `anon key` es pública por diseño: lo que protege los datos es RLS, que ya queda activado en `schema.sql`. La `service_role key` nunca va en el repositorio.

### 2. GitHub

Repositorio nuevo, privado, con los tres archivos en la raíz. Desde la web: **Add file → Upload files**. Desde la terminal:

```bash
git init
git add index.html schema.sql README.md
git commit -m "Estructura inicial"
git branch -M main
git remote add origin https://github.com/USUARIO/creando-tendencias.git
git push -u origin main
```

### 3. Netlify

Conecta GitHub directamente: **Add new site → Import an existing project → GitHub**, elige el repositorio y deja la configuración vacía (sin build command, publish directory `/` o `.`). Cada `push` a `main` publica solo.

Arrastrar el `index.html` al panel de Netlify también funciona, pero crea un sitio suelto sin historial: cada cambio hay que volver a arrastrarlo. Solo vale la pena si quieres ver la página funcionando en dos minutos antes de armar el repositorio.

Después, en **Site configuration → Domain management**, puedes cambiar el subdominio a algo como `creandotendencias.netlify.app`.

### 4. Redirección de URL en Supabase

En **Authentication → URL Configuration**, agrega la URL de Netlify en *Site URL* y en *Redirect URLs*. Sin esto, la recuperación de contraseña no vuelve a la página.

## Cómo limitar a las admins más adelante

Todo está en la tabla `permisos` (`rol`, `modulo_id`, `accion`, `permitido`). Por ejemplo, quitarle a las admins la edición de alquileres:

```sql
update permisos set permitido = false
where rol = 'admin' and modulo_id = 'alquileres' and accion = 'editar';
```

El cambio aplica al instante, sin desplegar nada.
