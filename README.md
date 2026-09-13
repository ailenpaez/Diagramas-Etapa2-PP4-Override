# Diagramas Etapa 2: Sistema Gluten Free Life

Este documento acompaña los diagramas UML desarrollados para el **Sistema Integral de Gestión de Inventario, Ventas y Trazabilidad Comercial de Gluten Free Life**. Los modelos se construyeron a partir de la Especificación de Requerimientos de Software (SRS) y del diagrama de clases elaborado por el equipo, tratando mantener la trazabilidad entre los requerimientos y cada representación.

Los diagramas incluidos son:

- `diagrama-actividades`
- `diagrama-estados`
- `diagrama-componentes`
- `diagrama-despliegue`

---

## Criterio utilizado

Los diagramas se realizaron a partir de los requerimientos funcionales y no funcionales definidos en el SRS y de las entidades presentes en el diagrama de clases.
Se buscó mantener coherencia entre los distintos modelos y representar únicamente los procesos y elementos necesarios para describir el funcionamiento del sistema.

---

## 1. Diagrama de Actividades: Registro de Venta

### Descripción

El diagrama representa el flujo principal correspondiente al **registro de una venta en el mostrador**. El proceso comienza con la selección del producto y el ingreso de la cantidad o peso, continúa con la validación de stock y el cálculo del total, y luego permite seleccionar el medio de pago.
Una vez registrada la venta, el sistema descuenta el stock del producto, evalúa si se alcanzó el nivel crítico y, cuando corresponde, genera una alerta. También contempla el funcionamiento ante caídas de conectividad: si no hay internet, la venta se guarda localmente y queda pendiente de sincronización hasta que la conexión se restablezca.

### Puntos de interés

- **Validación de stock antes de continuar la venta:** si la cantidad solicitada no está disponible, el usuario debe modificar la cantidad o peso y el sistema vuelve a verificar el stock antes de avanzar.
- **Venta por cantidad o peso:** se contempla el soporte para productos vendidos por unidad o de forma fraccionada.
- **Medios de pago:** el flujo distingue entre pago en efectivo y cobro electrónico mediante Mercado Pago/tarjeta.
- **Actualización del inventario:** al registrarse la venta se descuenta la cantidad correspondiente del producto.
- **Stock crítico:** luego de actualizar el inventario se verifica si el producto alcanzó el umbral crítico para generar la alerta correspondiente.
- **Operación sin conexión:** las ventas pueden almacenarse localmente cuando no hay internet y sincronizarse posteriormente.

### Requerimientos relacionados

- **RF-01:** administración de productos por unidad o peso fraccionado.
- **RF-03:** actualización del inventario con cada venta.
- **RF-04:** alertas de stock crítico.
- **RF-06:** registro de ventas y medios de cobro.
- **RNF-02:** funcionamiento offline y sincronización posterior.

### Modificaciones posteriores

El flujo de **stock insuficiente** vuelve a la modificación de cantidad o peso y realiza una nueva validación. De esta manera, la venta no continúa hasta contar con una cantidad válida.

---
## 2. Diagrama de Estados: PedidoCompra

### Descripción

El diagrama representa el **ciclo de vida propuesto de un `PedidoCompra`**, tomando como base la clase definida en el modelo y el requerimiento de compras del SRS.
El proceso comienza con la consolidación de productos que se encuentran en stock crítico. A partir de esa información se genera el pedido, se exporta en formato PDF/listado y se envía al intermediario logístico. Luego se representa la gestión de la compra, el retiro y la entrega de la mercadería hasta finalizar el pedido.

### Estados representados

**En preparación → Generado → Exportado → Enviado → En gestión → Entregado → Finalizado**

### Puntos de interés

- **En preparación:** se consolidan los productos que requieren reposición por stock crítico.
- **Generado:** representa la creación del pedido a partir de los faltantes detectados.
- **Exportado:** se relaciona con el método `generarPDF()` y con el atributo `archivoExportado` de `PedidoCompra`.
- **Enviado:** representa el envío del pedido al `IntermediarioLogistico`.
- **En gestión:** refleja la gestión de compra y retiro mencionada en el requerimiento.
- **Entregado:** corresponde a la entrega de la mercadería al comercio.
- **Finalizado:** indica el cierre del ciclo del pedido una vez recibida la mercadería.

### Requerimiento relacionado

- **RF-08:** consolidar artículos con stock crítico y exportar un pedido de compra para el intermediario logístico, encargado de gestionar la compra, el retiro y la entrega.

### Relación con el diagrama de clases

El modelo utiliza elementos ya presentes en el diagrama de clases:

- `PedidoCompra`
- `DetallePedido`
- `IntermediarioLogistico`
- `Producto`

También se mantiene la relación con los métodos `consolidarFaltantes()` y `generarPDF()` definidos en `PedidoCompra`.

### Modificaciones posteriores

Se eliminó el estado **Cancelado**, ya que esa posibilidad no está especificada en el SRS. De esta forma, el diagrama conserva únicamente el flujo respaldado por la documentación actual.

---
## 3. Diagrama de Componentes

### Descripción

El diagrama representa la **organización funcional del sistema** y las dependencias entre sus componentes. Cada módulo se relaciona con uno o más requerimientos definidos en el SRS.
La interfaz de usuario funciona como punto de acceso a las principales funcionalidades. Los módulos de negocio se comunican entre sí y utilizan mecanismos de persistencia local y remota. El sistema también contempla integraciones externas con Mercado Pago y la importación de información desde una planilla Excel.

### Puntos de interés

- **Interfaz de Usuario:** punto de acceso para el dueño y su pareja.
- **Autenticación y Usuarios:** administra los accesos individuales al sistema.
- **Gestión de Productos e Inventario:** centraliza catálogo, categorías y actualización del stock.
- **Gestión de Vencimientos y Alertas:** controla vencimientos y advertencias de stock crítico.
- **Punto de Venta (POS):** registra las operaciones de venta y se comunica con inventario, caja y cobro electrónico.
- **Gestión de Caja:** concentra la información necesaria para el cierre diario.
- **Gestión de Compras:** utiliza la información del inventario para la reposición de productos.
- **Costos y Precios:** relaciona el costo de compra con el margen comercial y el precio de venta.
- **Reportes:** utiliza información de inventario, ventas y caja para las estadísticas definidas en el SRS.
- **Persistencia Local + Sincronización:** permiten mantener la operación del sistema cuando se interrumpe la conexión.
- **Persistencia Remota:** recibe la información sincronizada y representa el almacenamiento remoto previsto por los requerimientos.
- **Mercado Pago / Cobro Electrónico:** se representa como sistema externo.
- **Planilla Excel:** representa la fuente utilizada para la migración inicial del catálogo, descripciones y precios.

### Trazabilidad con los requerimientos

- **RF-01 / RF-02 / RF-03:** Gestión de Productos e Inventario.
- **RF-04 / RF-05:** Gestión de Vencimientos y Alertas.
- **RF-06:** Punto de Venta y cobro electrónico.
- **RF-07:** Gestión de Caja.
- **RF-08:** Gestión de Compras.
- **RF-09:** Costos y Precios.
- **RF-10:** Reportes.
- **RF-11:** Autenticación y Usuarios.
- **RF-12:** migración desde Excel.
- **RNF-02 / RNF-03:** persistencia, sincronización y almacenamiento remoto.

---
## 4. Diagrama de Despliegue
### Descripción

El diagrama representa la **distribución física propuesta del sistema** y la comunicación entre dispositivos utilizados en Gluten Free Life, la infraestructura remota y el servicio externo de cobro electrónico.
La **PC de Mostrador** constituye el equipo principal de operación. Se ejecuta la aplicación, se mantiene almacenamiento local y se utiliza el servicio de sincronización. Esto permite continuar trabajando durante interrupciones de internet y enviar posteriormente la información a la infraestructura remota.
La **Notebook del Dueño** utiliza la aplicación o interfaz de gestión y accede a la información mediante la infraestructura en la nube. La integración con **Mercado Pago** se representa como un servicio externo utilizado para los cobros electrónicos.

### Puntos de interés

- **PC de Mostrador:** dispositivo principal para registrar las operaciones del comercio.
- **Almacenamiento Local:** permite conservar las operaciones durante los cortes de conectividad.
- **Servicio de Sincronización:** comunica los datos locales con la infraestructura remota cuando existe conexión.
- **Notebook del Dueño:** representa el segundo dispositivo de acceso definido para la operación y gestión.
- **Servidor de Aplicación / Backend API:** representa de forma tecnológica neutral el servicio que recibe las comunicaciones de los dispositivos.
- **Base de Datos / Réplica Remota Segura:** representa el almacenamiento remoto requerido para la información sincronizada y los respaldos.
- **Mercado Pago:** se mantiene fuera de la infraestructura propia del sistema por tratarse de un servicio externo.

### Requerimientos relacionados

- **RF-06:** utilización de cobro electrónico mediante Mercado Pago / tarjetas.
- **RNF-02:** operación offline, almacenamiento local y sincronización al recuperar internet.
- **RNF-03:** respaldo de la información con almacenamiento local y réplica remota segura.

### Modificaciones posteriores

Se evitó representar un segundo nodo de respaldo remoto independiente. El diagrama utiliza el nodo **Base de Datos/Réplica Remota Segura** para mantenerse dentro de lo especificado por RNF-03, sin asumir una infraestructura adicional que no se contempló en la documentación.

---
## Comentarios adicionales

Los cuatro diagramas representan diferentes caras de un mismo sistema:

- El **Diagrama de Actividades** muestra cómo se ejecuta el proceso de venta.
- El **Diagrama de Estados** muestra cómo evoluciona un `PedidoCompra` durante el ciclo de vida.
- El **Diagrama de Componentes** muestra qué partes funcionales del software permiten ejecutar los requerimientos.
- El **Diagrama de Despliegue** muestra dónde se ejecutan esos componentes y cómo se comunican los dispositivos y servicios externos.


Estas vistas complementan el **Diagrama de Clases**, que define las principales entidades del dominio, entre ellas `Usuario`, `Venta`, `DetalleVenta`, `Producto`, `VencimientoProducto`, `CierreCaja`, `PedidoCompra`, `DetallePedido` e `IntermediarioLogistico`.

---

📍 Basado en las actividades realizadas en el [Drive - Grupo Override](https://drive.google.com/drive/folders/1RWc6eRiwrnX08MYskbyjL1upUMOJ-Lql?usp=drive_link).

| Integrante grupo Override  |
| ------------- |
| Marcela Herrera      |
| Sebastián Puche      | 
| Neyel Vilaseco      |
| Ailén Páez     |
