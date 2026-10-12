# 🍽️ El Misti — Sistema de Coordinación de Pedidos en Tiempo Real

Sistema web + app móvil para digitalizar el flujo de pedidos de un restaurante, coordinando en tiempo real a meseros y cocina, e imprimiendo automáticamente los tickets en una impresora térmica Bluetooth.

> **Estado:** En producción · Usado activamente en el restaurante El Misti.

---

## 📖 Descripción

El Misti es un sistema que reemplaza el flujo manual de comandas físicas por un flujo digital sincronizado en tiempo real entre todos los dispositivos del restaurante.

**El problema que resuelve:**

Antes, los meseros tomaban pedidos en papel y los llevaban físicamente a la cocina. Esto generaba:
- Demoras entre la toma del pedido y su preparación.
- Pérdida o confusión de comandas.
- Falta de visibilidad del estado de cada pedido.
- Quejas constantes de clientes por tiempos de espera.

**La solución:**

Un sistema web accesible desde cualquier dispositivo (tablet o celular) donde los meseros registran pedidos, cocina los visualiza al instante, y una app móvil conectada a una impresora térmica Bluetooth imprime los tickets automáticamente. Todo sincronizado a través de Firebase Firestore.

---

## ✨ Características

- **Sincronización en tiempo real** entre múltiples dispositivos (meseros, cocina, pantalla de clientes) usando listeners reactivos de Firestore.
- **Gestión de stock en tiempo real** con control de disponibilidad de platos por sección (entradas / segundos).
- **Flujo de estados por pedido:** mesa, llevar, reserva, alistar (cocina).
- **Cronómetro por pedido** con cambio de color según el tiempo transcurrido (verde → amarillo → rojo) para priorizar visualmente.
- **Cálculo automático de totales** según combos y platos individuales.
- **Notas personalizadas** por plato (sin arroz, sin papa, etc.).
- **Impresión automática de tickets** en impresora térmica Bluetooth vía protocolo ESC/POS.
- **Historial de pedidos atendidos** con tiempo final de cada uno.
- **Exportación a Word** de pedidos para impresión en formato batch (grid de 6 columnas).

---

## 🏗️ Arquitectura

```text
┌─────────────────┐         ┌──────────────────┐         ┌─────────────────┐
│  Web (Meseros)  │ ──────▶ │                  │ ◀────── │  App Flutter    │
│  Web (Cocina)   │         │  Firebase        │         │  (Impresora)    │
│  Web (Clientes) │ ◀────── │  Firestore       │ ──────▶ │                 │
└─────────────────┘         │  (tiempo real)   │         └────────┬────────┘
                            └──────────────────┘                  │
                                                                  │ Bluetooth
                                                                  ▼
                                                       ┌──────────────────┐
                                                       │  Impresora       │
                                                       │  Térmica ESC/POS │
                                                       └──────────────────┘
```

**Flujo completo:**

1. El mesero registra un pedido en la web.
2. El pedido se guarda en Firestore (`pedidos/diaActual`).
3. Cocina recibe la actualización al instante vía listener.
4. La web también escribe un documento en `cola_impresion` con el texto formateado.
5. La app Flutter escucha `cola_impresion` en tiempo real.
6. Al detectar un documento pendiente, la app genera los bytes ESC/POS y los envía a la impresora Bluetooth.
7. La app marca el documento como `impreso` en Firestore.

---

## 🛠️ Stack Tecnológico

### Frontend Web
- **HTML5 + CSS3 + JavaScript vanilla**
- **Firebase SDK v10** (Firestore en tiempo real)
- **docx.js** para exportación a Word

### App Móvil
- **Flutter** (Dart)
- **`firebase_core` + `cloud_firestore`** — conexión con Firestore
- **`print_bluetooth_thermal`** — comunicación Bluetooth con la impresora
- **`esc_pos_utils`** — generación de comandos ESC/POS
- **`wakelock_plus`** — mantener pantalla encendida mientras escucha pedidos

### Backend
- **Firebase Firestore** (NoSQL en tiempo real, serverless)

### Hardware
- **Impresora térmica Bluetooth** compatible con ESC/POS (80 mm)

---

## 🚀 Instalación y Configuración

### Web

1. Clona el repositorio:

   ```bash
   git clone https://github.com/Daf1807/restaurant_order_management_system.git
   ```

2. Abre el archivo HTML en cualquier navegador, o súbelo a un hosting estático (Firebase Hosting, Netlify, Vercel).
3. La contraseña de acceso por defecto está definida en el propio archivo JS (`CLAVE_PRINCIPAL`).

### App Flutter

1. Entra a la carpeta de la app:

   ```bash
   cd app_flutter
   ```

2. Instala dependencias:

   ```bash
   flutter pub get
   ```

3. **Configura Firebase:**
   - Descarga `google-services.json` desde la consola de Firebase del proyecto `el-misti-pedidos`.
   - Colócalo en `android/app/google-services.json`.

4. **Conecta la impresora térmica:**
   - Empareja la impresora desde los ajustes Bluetooth del dispositivo Android.
   - Abre la app y selecciona la impresora de la lista.

5. Ejecuta:

   ```bash
   flutter run
   ```

---

## 📱 Uso

### Vista Meseros
- Selecciona tipo de pedido (mesa / llevar / reserva).
- Ingresa el número o nombre de mesa.
- Agrega platos con los botones `+` / `-` (con bloqueo automático si no hay stock disponible).
- Añade notas personalizadas con el botón 📝.
- Presiona **Registrar pedido**.

### Vista Cocina
- Muestra los pedidos activos con cronómetro.
- El color del pedido cambia según el tiempo (verde < 5 min, amarillo < 10 min, rojo > 10 min).
- Permite marcar pedidos como atendidos.
- Muestra el acumulado de platos para preparar (salón y llevar).

### Vista Clientes
- Pantalla pública que muestra la disponibilidad de platos en tiempo real.
- Ideal para proyectar en un monitor en la entrada del restaurante.

### App Flutter (Impresora)
- Se conecta a la impresora térmica Bluetooth.
- Escucha la cola de impresión en tiempo real.
- Imprime los tickets automáticamente al recibir un pedido nuevo.

---

## 📈 Resultados en Producción

El sistema está desplegado y en uso activo en el restaurante El Misti.

- ✅ **Reducción del 70 %** en quejas por demora en la atención.
- ✅ Coordinación en tiempo real entre meseros y cocina sin traslado físico de comandas.
- ✅ Adopción real por parte del personal operativo.

---

## 🧠 Lecciones Aprendidas

Este proyecto no solo fue un ejercicio técnico, sino una lección sobre adopción tecnológica en el mundo real:

1. **El éxito de una herramienta se mide por su adopción, no por sus funcionalidades.** Diseñé el sistema con múltiples vistas (meseros, cocina, clientes), pero en la práctica solo la vista de meseros-cocina se adoptó porque era la que resolvía el dolor real.
2. **En la transición de procesos manuales a digitales, el operador final decide.** Quien diseña la herramienta puede explotarla al 100 %, pero terceros suelen aprovechar solo un ~20 % de su potencial. La decisión final la toma el usuario según su comodidad y facilidad de uso.
3. **Priorizar UX simple > cantidad de features.** Incluso si eso significa sacrificar parte del potencial técnico del sistema. Es mejor una herramienta que se usa al 100 % que una que se usa al 20 %.
4. **Medir el impacto real, no solo entregar código.** El éxito no fue «terminé el sistema», fue «las quejas bajaron 70 %».

---

## 🗺️ Roadmap / Mejoras Futuras

- [ ] Sincronización offline con reintentos automáticos.
- [ ] Panel de administración con reportes de ventas diarias.
- [ ] Autenticación por roles (mesero / cocinero / admin).
- [ ] Notificaciones push a meseros cuando un pedido está listo.
- [ ] Integración con sistema de facturación electrónica (SUNAT).
- [ ] Historial de pedidos por día/semana/mes con gráficos.

---

## 📸 Capturas

> _Agrega aquí capturas de pantalla de la vista meseros, cocina, clientes y la app Flutter._

```text
/docs/screenshots/meseros.png
/docs/screenshots/cocina.png
/docs/screenshots/clientes.png
/docs/screenshots/app_flutter.png
```

---

## 👤 Autor

**Diego (Daf1807)**

- GitHub: [@Daf1807](https://github.com/Daf1807)
- Proyecto: [restaurant_order_management_system](https://github.com/Daf1807/restaurant_order_management_system)


