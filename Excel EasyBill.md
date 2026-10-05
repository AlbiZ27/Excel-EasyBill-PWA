**Excel EasyBill**

Perfecto. Yo congelaría el MVP alrededor de una sola promesa:

> **“Sube el Excel que ya usa tu negocio, revisa los datos y genera las facturas electrónicas en Factus sin aprender a usar un sistema contable.”**

Factus ya nos da el núcleo técnico: autenticación OAuth2, creación/validación de facturas, consulta de rangos y gestión/descarga de documentos.

## **1\. Qué debe hacer el MVP**

El recorrido completo sería:

INICIO  
  ↓  
Configurar Factus  
  ↓  
Subir Excel  
  ↓  
Detectar columnas  
  ↓  
Validar datos  
  ↓  
Vista previa  
  ↓  
Confirmar  
  ↓  
Generar facturas  
  ↓  
Factus API  
  ↓  
Resultados  
  ↓  
Descargar PDF

Y **nada más** en la primera versión.

No metería todavía notas crédito, nómina, documentos soporte, inventario, contabilidad, etc.

---

# **2\. La experiencia del usuario**

Queremos que una persona con casi cero conocimientos técnicos pueda usarlo.

### **Pantalla 1 — Inicio**

┌──────────────────────────────────┐  
│          FACTUS EASY             │  
│                                  │  
│ Convierte tus ventas de Excel    │  
│ en facturas electrónicas.        │  
│                                  │  
│       \[ Comenzar \]               │  
│                                  │  
│ ¿Necesitas ayuda?                │  
└──────────────────────────────────┘

Nada de términos como:

> OAuth  
>  API  
>  JSON  
>  Endpoint

Eso es nuestro problema, no el del usuario.

---

# **3\. Configuración de Factus**

La primera vez:

Conectar Factus

Correo / credenciales  
Token / configuración necesaria

\[Conectar\]

Nuestro backend se encarga de autenticación y renovación de tokens.

Esto es importante porque Factus usa OAuth2 y los access tokens tienen una duración limitada; la documentación indica que el token de acceso caduca cada hora.

### **MUY IMPORTANTE**

Las credenciales/tokens de Factus **no deberían quedar expuestos en el frontend**.

❌ PWA → Factus directamente

✅ PWA → Backend → Factus  
---

# **4\. Pantalla de Excel**

Esta será probablemente la pantalla más importante.

┌───────────────────────────────────────┐  
│        📊 Cargar ventas               │  
│                                       │  
│     Arrastra tu archivo aquí         │  
│                                       │  
│       \[ Seleccionar Excel \]           │  
│                                       │  
│ Formatos: .xlsx                      │  
└───────────────────────────────────────┘

Al subirlo:

ventas\_octubre.xlsx

✓ Archivo leído correctamente  
✓ 152 filas encontradas

\[Continuar\]

Yo empezaría **solo con `.xlsx`**.

CSV lo dejaría para después.

---

# **5\. Definir nuestro Excel**

Aquí tenemos que tomar una decisión importante.

No intentaría soportar desde el día uno cualquier Excel imaginable.

Primero tendremos una **plantilla oficial descargable**:

\[ Descargar plantilla Excel \]

Con columnas como:

| factura\_id | fecha | tipo\_doc | documento | cliente | email | producto | cantidad | precio | iva | forma\_pago |
| ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| FAC001 | 2026-10-04 | CC | 123456789 | Juan Pérez | juan@email.com | Camiseta | 2 | 50000 | 19 | Contado |

Esto nos permite controlar el MVP.

Pero aquí hay una característica que sí dejaría preparada:

### **Mapeo de columnas**

Si el negocio tiene:

Cliente  
C.C.  
Artículo  
Unidades  
Precio venta

podemos ofrecer:

Columna Excel       Campo del sistema

Cliente        →    Nombre cliente  
C.C.           →    Documento  
Artículo       →    Producto  
Unidades       →    Cantidad  
Precio venta   →    Precio

Más adelante ahí metemos IA.

---

# **6\. Agrupación de productos**

Este punto es **MUY importante técnicamente**.

No necesariamente:

1 fila \= 1 factura

Podemos tener:

| factura\_id | cliente | producto | cantidad | precio |
| ----- | ----- | ----- | ----- | ----- |
| FAC001 | Juan | Camiseta | 2 | 50000 |
| FAC001 | Juan | Pantalón | 1 | 80000 |
| FAC002 | María | Zapatos | 1 | 120000 |

Nuestro sistema genera:

FAC001  
 ├── Camiseta × 2  
 └── Pantalón × 1

FAC002  
 └── Zapatos × 1

Esto encaja directamente con el modelo de Factus, donde una factura contiene un arreglo `items` y cada producto/servicio tiene campos como código, nombre, cantidad, precio, unidad de medida e impuestos.

---

# **7\. Validador**

Antes de enviar **una sola factura**, mostramos:

RESULTADO DE VALIDACIÓN

152 ventas encontradas

✅ 145 listas  
⚠️ 7 con errores

Ejemplo:

Fila 32  
❌ Falta documento del cliente

Fila 58  
❌ Precio inválido

Fila 71  
❌ Falta producto

Y, muy importante:

### **El usuario puede corregirlos.**

\[ Corregir Excel \]

o

\[ Continuar solo con 145 \]

Para el MVP yo permitiría la segunda opción.

---

# **8\. Validaciones que nosotros hacemos**

No debemos dejar que todos los errores lleguen a Factus.

Por ejemplo:

✓ Fecha válida  
✓ Documento presente  
✓ Cliente presente  
✓ Producto presente  
✓ Cantidad \> 0  
✓ Precio \>= 0  
✓ IVA válido  
✓ Forma de pago válida  
✓ Factura tiene al menos un producto

Además, podemos consultar los rangos de numeración disponibles mediante Factus y determinar cuál usar. La API expone precisamente un endpoint para listar esos rangos y saber cuáles están activos.

---

# **9\. Generar facturas**

Después:

145 facturas listas

        \[ GENERAR FACTURAS \]

El backend transforma:

Excel

en:

{  
  "reference\_code": "FE-2026-000001",  
  "customer": {...},  
  "payment\_details": \[...\],  
  "items": \[...\]  
}

y llama:

POST /v2/bills/validate

en Factus.

---

# **10\. Reference Code**

Aquí tenemos que implementar algo importante desde el principio.

Cada factura tendrá un identificador interno único:

FE-2026-000001  
FE-2026-000002  
FE-2026-000003

Factus utiliza `reference_code` precisamente para identificar de forma única cada factura y evitar duplicados.

Esto además nos protege contra el usuario que accidentalmente haga:

\[Generar facturas\]

... espera ...

\[Generar facturas\] ← segunda vez

No deberíamos generar duplicados.

---

# **11\. Resultado del procesamiento**

La pantalla final:

╔═══════════════════════════════╗  
║      PROCESO COMPLETADO       ║  
╠═══════════════════════════════╣  
║                               ║  
║  145 facturas procesadas      ║  
║                               ║  
║  ✅ 140 aceptadas              ║  
║  ❌   5 rechazadas             ║  
║                               ║  
╚═══════════════════════════════╝

Y debajo:

| Factura | Cliente | Estado | Acción |
| ----- | ----- | ----- | ----- |
| FE-001 | Juan Pérez | ✅ Validada | PDF |
| FE-002 | María Gómez | ✅ Validada | PDF |
| FE-003 | Carlos Ruiz | ❌ Error | Ver |
| FE-004 | Ana López | ✅ Validada | PDF |

---

# **12\. PDF**

Para cada factura válida:

\[ 📄 Descargar PDF \]

La documentación de Factus indica que las facturas pueden consultarse y descargarse en PDF y XML.

Para el MVP solamente necesitamos:

**PDF.**

XML lo dejamos para una siguiente iteración.

---

# **13\. Arquitectura**

Yo dejaría exactamente esto:

               ┌─────────────────┐  
                │      PWA        │  
                │                 │  
                │ React           │  
                │ TypeScript      │  
                │ Vite            │  
                └────────┬────────┘  
                         │  
                   HTTPS / JSON  
                         │  
                         ▼  
                ┌─────────────────┐  
                │    BACKEND      │  
                │                 │  
                │ Node.js         │  
                │ Express         │  
                │ TypeScript      │  
                └───────┬─────────┘  
                        │  
             ┌──────────┼───────────┐  
             │          │           │  
             ▼          ▼           ▼  
          PostgreSQL  Validador   Factus API  
---

# **14\. ¿Dónde procesamos el Excel?**

Yo haría:

Excel  
  ↓  
PWA  
  ↓  
SheetJS  
  ↓  
JSON

La lectura inicial puede hacerse **en el navegador**.

Después enviamos al backend los datos necesarios.

Eso significa que no tenemos que subir el archivo Excel completo al servidor necesariamente.

---

# **15\. Base de datos mínima**

Solo necesitamos unas pocas tablas.

### **`users`**

id  
email  
password\_hash  
created\_at

### **`businesses`**

id  
user\_id  
nit  
name

### **`factus_config`**

id  
business\_id  
...

Aquí hay que definir con cuidado cómo almacenamos los secretos de Factus.

### **`invoices`**

id  
business\_id  
reference\_code  
factus\_number  
customer\_name  
total  
status  
error\_message  
created\_at

### **`invoice_items`**

id  
invoice\_id  
product  
quantity  
price  
tax\_rate

Con eso tenemos suficiente para el MVP.

---

# **16\. Stack definitivo**

Yo lo dejaría así:

FRONTEND  
React  
TypeScript  
Vite  
PWA  
SheetJS

BACKEND  
Node.js  
Express  
TypeScript

DATABASE  
PostgreSQL

FACTURACIÓN  
Factus API

DEPLOY  
Frontend → Vercel  
Backend → Render / Railway  
DB → PostgreSQL / Supabase

Y **no metería IA todavía**.

La IA sería:

### **MVP+**

Excel  
 ↓  
IA detecta columnas  
 ↓  
"Creo que estas columnas corresponden a..."  
 ↓  
Usuario confirma

Eso puede ser nuestro **elemento sorpresa** después de que el flujo básico ya funcione.

---

# **17\. Lo que NO vamos a desarrollar**

Para evitar que el proyecto se vuelva inmanejable:

❌ Inventario  
❌ Contabilidad  
❌ POS  
❌ Nómina  
❌ Documentos soporte  
❌ Notas débito  
❌ Notas crédito  
❌ Aplicación móvil  
❌ Complemento Excel  
❌ Google Sheets  
❌ IA conversacional completa

Más adelante podemos agregar algunas.

---

# **18\. Las 6 funcionalidades que realmente debemos terminar**

Nuestro MVP queda reducido a:

### **🟢 1\. Conectar Factus**

Autenticación y configuración.

### **🟢 2\. Cargar Excel**

Leer `.xlsx`.

### **🟢 3\. Validar**

Detectar errores antes de enviar.

### **🟢 4\. Convertir**

Excel → estructura JSON de Factus.

### **🟢 5\. Facturar en lote**

Enviar varias facturas a Factus.

### **🟢 6\. Resultados**

Ver estado y descargar PDF.

Eso ya constituye **un producto funcional**, no solo una demo.

---

## **Y la demo ideal sería brutalmente sencilla**

Tenemos una tienda ficticia.

↓  
Abrimos Excel

150 ventas.

↓

Abrimos nuestra PWA.

↓

"Subir Excel"

↓

150 registros.

↓

"145 listos / 5 errores"

↓

Corregimos o descartamos los 5\.

↓

"Generar facturas"

↓

Factus

↓

"145 facturas procesadas"

↓

Abrimos una.

↓

PDF de factura electrónica.

Y podemos cerrar la presentación con:

> **“El negocio no tuvo que cambiar su Excel, comprar un ERP ni aprender a usar una API. Simplemente utilizó los datos que ya tenía.”**

Ese debería ser el corazón del proyecto.

### **Orden de construcción**

Yo programaría en este orden:

**Fase 1:** backend conectado al sandbox de Factus.  
 **Fase 2:** Excel → JSON interno.  
 **Fase 3:** validador.  
 **Fase 4:** JSON interno → Factus.  
 **Fase 5:** PWA.  
 **Fase 6:** procesamiento múltiple \+ resultados.  
 **Fase 7:** PDF.  
 **Fase 8:** IA para mapeo automático como diferenciador.

Así, incluso si nos quedamos sin tiempo, **ya tenemos funcionando el núcleo del reto**.

