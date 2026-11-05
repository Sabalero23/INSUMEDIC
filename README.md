# INSUMEDIC

**Sistema de gestión (ERP / punto de venta) para insumos médicos, ortopedia y alquiler de equipos.**
Aplicación de escritorio para Windows, desarrollada por **Cellcom Technology** ([cellcomweb.com.ar](https://www.cellcomweb.com.ar)).

Funciona 100 % de forma local sobre una base de datos Firebird: no depende de internet para vender.

> Nombre interno del proyecto y del código: `CellcomPOS`.

---

---

## Qué hace

| Módulo | Descripción |
|---|---|
| **Ventas** | Venta de mostrador, buscador y lector de código de barras, descuentos, ticket impreso, factura electrónica ARCA (opcional). |
| **Productos / Stock** | Catálogo con marca, presentación, talle/lado, registro ANMAT, lote y vencimiento, fotos, Kardex de movimientos y actualización masiva de precios. |
| **Costos y margen** | Precio de compra + costo de transporte + margen % = precio de venta calculado. Los costos se ocultan al vender y a los usuarios sin permiso. |
| **Ganancias** | Ganancia real por día, semana, mes y por producto, usando el costo que tenía cada producto al venderse. |
| **Proveedores y Compras** | Compras con actualización automática de stock y costos, cuenta corriente y pagos a proveedores. |
| **Alquileres y Equipos** | Alquiler por día/semana/mes, renovaciones, devoluciones con recargos, depósitos y comprobante. |
| **Caja** | Apertura y cierre por terminal, ingresos/retiros, desglose y arqueo. |
| **Inventario** | Conteo físico con lector, comparación contra el sistema, ajustes con Kardex e impresión del stock esperado. |
| **Etiquetas** | Etiquetas de código de barras (Code 128) con vista previa, tamaños predefinidos o personalizados, precio opcional y una etiqueta por unidad de stock. |
| **Clientes** | Alta y edición, con datos para reclamar equipos alquilados. |
| **Usuarios y Permisos** | Roles Empleado / Gerente / Superadmin y permisos por usuario. |
| **Copias de seguridad** | Copia manual y programada de la base, con carpeta secundaria y control de antigüedad. |
| **Importación** | Carga masiva de productos desde un archivo CSV. |
| **Sync Web** | Sincronización con el sitio web / tienda online (módulo opcional). |
| **Actualizaciones** | Al abrirse, consulta si hay una versión nueva publicada en este repositorio y ofrece instalarla. |

El manual de uso (`INSUMEDIC_Manual_de_Usuario.docx`) explica cada pantalla paso a paso.

---

## Tecnología

- **.NET 8** · WPF (`net8.0-windows`)
- **Firebird 5.0** (servidor en la PC principal, el resto de las PCs se conectan por red local)
- `FirebirdSql.Data.FirebirdClient`, `BCrypt.Net-Next`, `System.*` 10.0.14
- Instalador con **Inno Setup 6**

## Requisitos

- Windows 10 / 11 (x64)
- Runtime de escritorio de .NET 8 (si la publicación no es autocontenida)
- Firebird 5.0 en la PC que hace de servidor

## Estructura del repositorio

```
CellcomPOS/                  Proyecto WPF (código fuente)
  Models/ Repositories/ Services/ Views/ Assets/
CellcomPOS.sln
INSUMEDIC_Setup.iss          Script del instalador (Inno Setup)
migracion_faseXX_*.sql       Scripts SQL por versión (se ejecutan en orden, una sola vez)
base_de_datos_nueva_INSUMEDIC_FB5.sql   Base completa para una instalación nueva
LicenseGenerator/            Generador de licencias
sync-web-php-ejemplo/        Ejemplo del lado servidor para Sync Web
README.md
```

---

## Instalación en una PC nueva

1. Instalar Firebird 5.0 (solo en la PC servidor) y crear la base con `base_de_datos_nueva_INSUMEDIC_FB5.sql`.
2. Ejecutar `INSUMEDIC_Setup_X.Y.Z.exe` (se instala en `C:\INSUMEDIC`).
3. Abrir INSUMEDIC. En la primera apertura pide crear la cuenta de Superadmin y configurar el servidor.

## Base de datos: migraciones

Cada versión que cambia la base trae un script `migracion_faseXX_*.sql`. **Se ejecuta una sola vez, antes de abrir la versión nueva**, con `isql` contra la base existente. Hacer siempre una copia de seguridad antes.

---

## Actualizaciones automáticas

INSUMEDIC consulta, cada vez que se abre, el **último Release** de este repositorio:

1. Pide a la API de GitHub (`/releases/latest`) la etiqueta y los archivos adjuntos.
2. Compara la etiqueta (por ejemplo `v0.27.0`) con la versión instalada (la que figura en el título de la ventana).
3. Si es más nueva, **a Gerente y Superadmin** les muestra una ventana con las novedades y el botón *Descargar e instalar*. Los Empleados no ven nada.
4. Descarga el instalador, verifica su firma SHA-256 (si GitHub la informa), cierra el programa y ejecuta el instalador.

Si no hay internet, o el repositorio no responde, el sistema abre normalmente y no muestra ningún error.

**Configuración:** en `CellcomPOS/AppBrand.cs`, cambiar

```csharp
public const string RepositorioActualizaciones = "TU_USUARIO/insumedic";
```

por `usuario/repositorio`. El repositorio (o uno aparte solo para releases) tiene que ser **público**: la consulta es anónima. Si el código debe ser privado, crear un segundo repositorio público (por ejemplo `insumedic-releases`) y publicar allí solo los Releases.

### Cómo publicar una versión nueva

1. Subir la versión en `CellcomPOS/CellcomPOS.csproj` (`<Version>0.27.0</Version>`) y en `#define VersionApp` de `INSUMEDIC_Setup.iss`. Las dos tienen que coincidir.
2. En Visual Studio: *Publicar* el proyecto en la carpeta `Publish`.
3. Compilar `INSUMEDIC_Setup.iss` en Inno Setup (F9): genera `Instalador\INSUMEDIC_Setup_0.27.0.exe`.
4. En GitHub: *Releases → Draft a new release*.
   - **Tag:** `v0.27.0` (siempre `v` + la misma versión del csproj).
   - **Título:** `INSUMEDIC 0.27.0`.
   - **Descripción:** las novedades (es lo que ve el usuario en la ventana de actualización). Si la versión trae una migración de base de datos, **decirlo acá** y adjuntar el `.sql`.
   - **Archivos adjuntos:** `INSUMEDIC_Setup_0.27.0.exe` (el nombre tiene que empezar con `INSUMEDIC_Setup` y terminar en `.exe`) y, si corresponde, `migracion_faseXX_*.sql`.
   - Marcar *Set as the latest release* y publicar. Un borrador (*draft*) o *pre-release* no se ofrece a los usuarios.

### Plantilla de notas para un Release

```markdown
## Novedades
- ...

## Correcciones
- ...

## ⚠️ Antes de actualizar
- Requiere ejecutar `migracion_faseXX_nombre.sql` en la base (una sola vez, antes de abrir la versión nueva).
- (o) No requiere cambios en la base de datos.
```

> **Importante:** la actualización instala el programa en esa PC, pero **no toca la base de datos**. Si una versión trae migración, hay que correrla en el servidor antes de abrir la versión nueva en cualquier PC, y actualizar todas las PCs del local.

---

## Versionado

`MAYOR.MENOR.PARCHE`, tomado de `<Version>` en el csproj. Cada entrega suma el MENOR (0.26.0 → 0.27.0); las correcciones chicas suman el PARCHE (0.26.1).

## Licencia

Software propietario de **Cellcom Technology**. Todos los derechos reservados.
