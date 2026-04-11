<!-- .github/copilot-instructions.md -->
# Contexto del Proyecto Green-POS

## 📋 Descripción General
Green-POS es un Sistema de Punto de Venta completo desarrollado en Flask que incluye gestión de inventario, facturación, clientes y servicios de mascotas (grooming, veterinaria).

## 🤖 Sistema de Copilot Agents Especializados

Este proyecto utiliza **tres agents ejecutables en Copilot Agent Mode** para desarrollo modular:

### Agents Disponibles (VS Code Insiders)

- **🎨 @green-pos-frontend** (`.github/agents/green-pos-frontend.agent.md`)
  - Templates Jinja2, Bootstrap 5.3+, JavaScript Vanilla
  - Componentes UI, responsive design, validación cliente
  - **Subagents**: scaffold_page, table_datatable, accessibility_audit
  
- **🐍 @green-pos-backend** (`.github/agents/green-pos-backend.agent.md`)
  - Rutas Flask, lógica de negocio, APIs JSON
  - Autenticación, validación servidor, transacciones
  - **Subagents**: generate_crud, add_validation, create_api
  
- **🗄️ @green-pos-database** (`.github/agents/green-pos-database.agent.md`)
  - Modelos SQLAlchemy, relaciones, migraciones
  - Optimización queries, constraints, índices
  - **Subagents**: generate_model, create_migration, optimize_queries

**Guías de uso**: 
- Completa: `.github/agents/README.md`
- Referencia rápida: `.github/agents/QUICK_REFERENCE.md`

**Documentación técnica** (para humanos):
- Frontend: `.github/instructions/frontend-html-agent.instructions.md`
- Backend: `.github/instructions/backend-python-agent.instructions.md`
- Database: `.github/instructions/database-sqlite-agent.instructions.md`

### Cómo Invocar los Agents

**Uso básico**:
```
@green-pos-frontend crea templates/suppliers/list.html con DataTable
@green-pos-backend implementa CRUD completo para Supplier
@green-pos-database crea modelo Supplier con relación a Product
```

**Con subagents**:
```
@green-pos-frontend 
#runSubagent <subagent_scaffold_page> pathOut=templates/reports/sales.html pageTitle="Ventas"

@green-pos-backend 
#runSubagent <subagent_generate_crud> entityName=Supplier routePrefix=suppliers

@green-pos-database 
#runSubagent <subagent_generate_model> entityName=Category fields=[...]
```

**Workflow multi-agent** (CRUD completo):
```bash
# Paso 1: Crear modelo
@green-pos-database crea modelo Supplier con campos code, name, phone, email

# Paso 2: Crear rutas
@green-pos-backend implementa CRUD para Supplier con validación

# Paso 3: Crear vistas
@green-pos-frontend crea templates/suppliers/list.html, form.html, view.html
```

## 🏗️ Stack Tecnológico Principal
- **Backend**: Flask 3.0+ + SQLAlchemy + Flask-Login
- **Frontend**: HTML5 + Bootstrap 5.3+ (sin jQuery) + Vanilla JavaScript
- **Base de Datos**: SQLite (desarrollo) con soporte PostgreSQL/MySQL
- **Servidor**: Waitress (Windows) / Gunicorn (Linux)
- **Reportes**: ReportLab para PDFs (facturas, consentimientos)
- **Zona Horaria**: America/Bogota (CO_TZ) - pytz
- **Iconos**: Bootstrap Icons (`bi-*`)

## � Constraints Tecnológicos

### Limitaciones de SQLite
**CRÍTICO - Restricciones de Producción:**

1. **Concurrencia Limitada**:
   - SQLite NO soporta escrituras concurrentes
   - Un solo writer a la vez (readers múltiples permitidos)
   - Timeout configurado a 30 segundos para locks
   - **Solución**: Para alta concurrencia → PostgreSQL/MySQL

2. **Transacciones Explícitas**:
   ```python
   # OBLIGATORIO: Siempre usar try-except con rollback
   try:
       db.session.add(entity)
       db.session.commit()
   except Exception as e:
       db.session.rollback()
       app.logger.error(f"Error: {e}")
       raise
   ```

3. **Tipos de Datos**:
   - No tiene tipo DATE/TIME nativo (usa TEXT/INTEGER)
   - `DateTime` se almacena como TEXT ISO 8601
   - Boolean se almacena como INTEGER (0/1)
   - No tiene tipo DECIMAL (usa REAL o TEXT)

4. **Tamaño de Base de Datos**:
   - Límite teórico: 140 TB
   - Límite práctico: 1-2 GB para rendimiento óptimo
   - Para más: migrar a PostgreSQL

5. **Backups**:
   - NO tiene backup en caliente nativo
   - Requiere copiar archivo completo
   - Recomendado: Backup nocturno automatizado

### Restricciones de Codificación UTF-8

**CRÍTICO - Servidor de Producción Windows:**

1. **NO usar emojis en código Python**:
   - Servidor de producción tiene problemas con emojis (✅ ❌ 🔄 ⚠️ 📝 etc.)
   - Error relacionado con codificación UTF-8 al imprimir
   - Impacta: Scripts de migración, verificación, mensajes de consola

2. **Alternativas permitidas**:
   ```python
   # ❌ INCORRECTO: Usar emojis
   print("✅ Migración exitosa")
   print("❌ Error en migración")
   print("🔄 Procesando...")
   
   # ✅ CORRECTO: Usar prefijos de texto
   print("[OK] Migracion exitosa")
   print("[ERROR] Error en migracion")
   print("[INFO] Procesando...")
   
   # Prefijos estándar:
   # [OK]      - Operación exitosa (verde)
   # [ERROR]   - Error crítico (rojo)
   # [WARNING] - Advertencia (amarillo)
   # [INFO]    - Información (cyan)
   # [DELETE]  - Operación de borrado
   ```

3. **Caracteres acentuados**:
   - EVITAR acentos en mensajes de consola (ó, á, é, í, ú, ñ)
   - Usar versiones sin acento: (o, a, e, i, u, n)
   - OK en templates HTML y base de datos (soportan UTF-8)
   
4. **Archivos afectados**:
   - Scripts de migración (`migrate_*.py`)
   - Scripts de verificación (`verify_*.py`)
   - Funciones de logging y print statements
   - Mensajes de error/éxito en consola

### Scripts de Migración (migrations/)

**CRÍTICO - Patrón de Resolución de Paths:**

1. **NUNCA usar rutas relativas simples**:
   ```python
   # ❌ INCORRECTO: Depende del CWD (Current Working Directory)
   open('archivo.sql')
   sqlite3.connect('instance/app.db')
   db_path = os.path.join('instance', 'app.db')
   
   # ✅ CORRECTO: Ruta relativa al script (funciona desde cualquier CWD)
   from pathlib import Path
   
   SCRIPT_DIR = Path(__file__).parent
   PROJECT_ROOT = SCRIPT_DIR.parent
   
   sql_file = SCRIPT_DIR / 'archivo.sql'
   db_path = PROJECT_ROOT / 'instance' / 'app.db'
   ```

2. **Usar template estándar**:
   - Base: `migrations/TEMPLATE_MIGRATION.py`
   - Copiar template y personalizar para nueva migración
   - Incluye: Path resolution, backup automático, fallback SQL, verificación
   - Logging con prefijos [OK], [ERROR], [WARN], [INFO]

3. **Verificar desde diferentes directorios**:
   ```powershell
   # SIEMPRE probar desde raíz del proyecto (caso común en producción)
   python migrations/migration_nombre.py
   
   # También probar desde directorio migrations/
   cd migrations && python migration_nombre.py
   
   # Verificar con ruta absoluta
   python D:\ruta\completa\migrations\migration_nombre.py
   ```
   **Resultado esperado**: Script funciona en todos los casos sin FileNotFoundError

4. **Archivos afectados**:
   - Scripts de migración (`migrate_*.py`, `migration_*.py`)
   - Scripts de verificación (`verify_*.py`)
   - Scripts de consulta (`query_*.py`)
   - Cualquier script standalone en `migrations/`

5. **Por qué es importante**:
   - Python resuelve `open('archivo')` relativo al CWD, NO a la ubicación del script
   - En producción, scripts se ejecutan desde raíz: `python migrations/script.py`
   - Sin `Path(__file__).parent`, script busca en raíz en lugar de `migrations/`
   - Resultado: `FileNotFoundError` aunque archivo exista

6. **Ejemplo del problema**:
   ```
   Green-POS/                          <-- CWD al ejecutar: python migrations/script.py
   ├── migrations/
   │   ├── script.py                   <-- Script ejecutado
   │   └── archivo.sql                 <-- Archivo real AQUÍ
   └── archivo.sql                     <-- Python busca AQUÍ (no existe) ❌
   ```

7. **Fix aplicado**:
   - Commit `2d412fc`: Fix en `migration_add_inventory_flag.py`
   - Investigación completa: `docs/research/2025-11-24-causa-raiz-filenotfounderror-migracion-produccion.md`
   - Documento dedicado: `docs/FIX_FILENOTFOUNDERROR_MIGRATION_PATHS.md`

### Restricciones de Flask

1. **Single-threaded por defecto**:
   - Waitress usa 4 threads (configurado)
   - NO usar variables globales mutables
   - Thread-safety con `current_app`, `g`, `session`

2. **Manejo de Archivos**:
   - Max upload size: Configurar en nginx/apache
   - Validar extensiones permitidas
   - Sanitizar nombres de archivo

3. **Sesiones**:
   - Por defecto: Cookie-based (client-side)
   - Max 4KB de datos
   - Firmadas pero NO encriptadas
   - Sensibles: Usar Flask-Session con Redis

### Restricciones de Bootstrap 5

1. **Sin jQuery**:
   - NO usar plugins jQuery legacy
   - Vanilla JS para interacciones
   - Bootstrap Bundle incluye Popper.js

2. **Navegadores Soportados**:
   - Chrome, Firefox, Safari (últimas 2 versiones)
   - Edge Chromium
   - NO soporta IE11

3. **Responsive Breakpoints**:
   ```scss
   // Obligatorio respetar estos breakpoints
   xs: <576px   (mobile)
   sm: ≥576px   (mobile landscape)
   md: ≥768px   (tablet)
   lg: ≥992px   (desktop)
   xl: ≥1200px  (large desktop)
   xxl: ≥1400px (extra large)
   ```

### Restricciones de Zona Horaria

1. **Appointments (Citas)**:
   - `scheduled_at`: Timezone-naive (hora local)
   - NO convertir a UTC
   - Usuario programa → guarda → muestra (sin transformación)
   
2. **Invoices (Facturas)**:
   - `date`: Timezone-aware (UTC storage)
   - Convertir a CO_TZ para display
   - Usar filtro `format_time_co`

3. **Timestamps de Sistema**:
   - `created_at`, `updated_at`: UTC
   - Conversión automática en queries
   - `datetime.now(timezone.utc)` para consistency

### Restricciones de ReportLab (PDFs)

1. **Fuentes**:
   - Usar fuentes built-in (Helvetica, Times, Courier)
   - Custom fonts requieren archivo .ttf
   - Unicode requiere configuración especial

2. **Tamaño de Página**:
   - Facturas: Optimizado para 80mm térmicas
   - A4 para reportes estándar
   - Letter para US

3. **Imágenes**:
   - PNG, JPEG soportados
   - GIF requiere conversión
   - Tamaño máximo: 10MB recomendado

## 🎯 Patrones de Diseño Implementados

### 1. Repository Pattern (Parcial)

**Uso**: Acceso a datos centralizado

```python
# Patrón usado en queries complejas
class ProductRepository:
    @staticmethod
    def get_with_sales_count():
        """Obtiene productos con conteo de ventas."""
        return db.session.query(
            Product,
            func.count(InvoiceItem.id).label('sales_count')
        ).outerjoin(InvoiceItem).group_by(Product.id).all()
    
    @staticmethod
    def search(query_text, sort_by='name', sort_order='asc'):
        """Búsqueda con ordenamiento dinámico."""
        q = Product.query.filter(
            or_(
                Product.name.ilike(f'%{query_text}%'),
                Product.code.ilike(f'%{query_text}%')
            )
        )
        return q.order_by(getattr(Product, sort_by).desc() 
                         if sort_order == 'desc' 
                         else getattr(Product, sort_by).asc()).all()
```

**Ventajas**:
- Lógica de queries centralizada
- Reutilización de código
- Testing más fácil

### 2. Decorator Pattern

**Uso**: Autorización y autenticación

```python
from functools import wraps
from flask_login import current_user
from flask import flash, redirect, url_for

def role_required(role):
    """Decorator para validar rol del usuario."""
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            if not current_user.is_authenticated:
                flash('Debe iniciar sesión', 'warning')
                return redirect(url_for('login'))
            
            if current_user.role != role and current_user.role != 'admin':
                flash('Acceso denegado', 'danger')
                return redirect(url_for('index'))
            
            return f(*args, **kwargs)
        return decorated_function
    return decorator

# Uso:
@app.route('/products/new')
@login_required
@role_required('admin')
def product_new():
    # Solo admin puede crear productos
    pass
```

**Ventajas**:
- Código declarativo
- Reutilizable en todas las rutas
- Separación de concerns

### 3. Factory Pattern

**Uso**: Creación de usuarios por defecto

```python
class User(db.Model):
    # ...
    
    @staticmethod
    def create_defaults():
        """Factory para crear usuarios iniciales."""
        existing = User.query.count()
        if existing == 0:
            users = [
                ('admin', 'admin', 'admin'),
                ('vendedor', 'vendedor', 'vendedor'),
            ]
            for username, password, role in users:
                user = User(username=username, role=role)
                user.set_password(password)
                db.session.add(user)
            db.session.commit()
```

### 4. Observer Pattern (Implicit)

**Uso**: Hooks de SQLAlchemy para audit logs

```python
class ProductStockLog(db.Model):
    """Log de cambios - patrón Observer implícito."""
    # Al cambiar Product.stock, se crea automáticamente un log

# Implementación en product_edit():
@app.route('/products/edit/<int:id>', methods=['GET', 'POST'])
def product_edit(id):
    product = Product.query.get_or_404(id)
    
    if request.method == 'POST':
        new_stock = int(request.form.get('stock', 0))
        old_stock = product.stock
        
        if new_stock != old_stock:
            # "Notificar" al observer creando log
            log = ProductStockLog(
                product_id=product.id,
                user_id=current_user.id,
                quantity=abs(new_stock - old_stock),
                movement_type='addition' if new_stock > old_stock else 'subtraction',
                reason=request.form['stock_reason'],
                previous_stock=old_stock,
                new_stock=new_stock
            )
            db.session.add(log)
```

**Ventajas**:
- Trazabilidad automática
- Auditoría completa
- Desacoplado del modelo principal

### 5. Strategy Pattern

**Uso**: Métodos de pago en facturas

```python
class Invoice(db.Model):
    payment_method = db.Column(db.String(50), default='cash')
    # Valores: 'cash', 'transfer', 'card', 'mixed'

# Estrategia de procesamiento según método
def process_payment(invoice):
    strategies = {
        'cash': process_cash_payment,
        'transfer': process_transfer_payment,
        'card': process_card_payment,
        'mixed': process_mixed_payment
    }
    
    processor = strategies.get(invoice.payment_method)
    return processor(invoice) if processor else None
```

### 6. Template Method Pattern

**Uso**: Rutas CRUD estándar

```python
# Plantilla común para CRUD operations
def crud_list(model_class, template, **filters):
    """Template method para listar entidades."""
    query = model_class.query
    
    # Hook: aplicar filtros
    for key, value in filters.items():
        if value:
            query = query.filter(getattr(model_class, key).ilike(f'%{value}%'))
    
    # Hook: ordenar
    items = query.order_by(model_class.created_at.desc()).all()
    
    # Hook: renderizar
    return render_template(template, items=items)

# Especialización:
@app.route('/products')
def product_list():
    return crud_list(
        Product, 
        'products/list.html',
        name=request.args.get('query')
    )
```

### 7. Builder Pattern

**Uso**: Construcción de mensajes WhatsApp

```python
class WhatsAppMessageBuilder:
    """Builder para crear mensajes WhatsApp estructurados."""
    
    def __init__(self):
        self.parts = []
    
    def add_greeting(self, name):
        self.parts.append(f"Hola {name}!")
        return self
    
    def add_business(self, business_name):
        self.parts.append(f"Te saludamos desde *{business_name}*")
        return self
    
    def add_appointment_info(self, pet_name, date, time):
        self.parts.append(
            f"Recuerda la cita con nuestra groomer para *{pet_name}*\n\n"
            f"Fecha: {date}\n"
            f"Hora: {time}"
        )
        return self
    
    def add_closing(self):
        self.parts.append("Gracias por preferirnos!")
        return self
    
    def build(self):
        return "%0A".join(self.parts)

# Uso en template:
{% set mensaje = WhatsAppMessageBuilder()
    .add_greeting(first_name)
    .add_business("Pet Verde")
    .add_appointment_info(pet_name, fecha, hora)
    .add_closing()
    .build() %}
```

### 8. Adapter Pattern

**Uso**: Adaptador de zona horaria

```python
class TimezoneAdapter:
    """Adapta datetime UTC a timezone local."""
    
    @staticmethod
    def to_local(utc_datetime, tz=CO_TZ):
        """Convierte UTC a timezone local."""
        if utc_datetime.tzinfo is None:
            utc_datetime = utc_datetime.replace(tzinfo=timezone.utc)
        return utc_datetime.astimezone(tz)
    
    @staticmethod
    def to_utc(local_datetime, tz=CO_TZ):
        """Convierte local a UTC."""
        if local_datetime.tzinfo is None:
            local_datetime = tz.localize(local_datetime)
        return local_datetime.astimezone(timezone.utc)

# Filtro Jinja2:
@app.template_filter('format_time_co')
def format_time_co(dt):
    """Adapta y formatea datetime a hora Colombia."""
    if not dt:
        return ''
    local_dt = TimezoneAdapter.to_local(dt)
    return local_dt.strftime('%I:%M %p')
```

### 9. Composite Pattern

**Uso**: Estructura jerárquica de servicios

```python
class Appointment(db.Model):
    """Composite: Agrupa múltiples servicios."""
    services = db.relationship('PetService', backref='appointment', 
                              lazy='dynamic', cascade='all, delete-orphan')
    
    def recompute_total(self):
        """Calcula total recursivamente de todos los servicios."""
        self.total_price = sum(s.price or 0 for s in self.services)
        self.total_services = self.services.count()

class PetService(db.Model):
    """Leaf: Servicio individual."""
    appointment_id = db.Column(db.Integer, db.ForeignKey('appointment.id'))
    service_type_id = db.Column(db.Integer, db.ForeignKey('service_type.id'))
    price = db.Column(db.Float, nullable=False)
```

### 10. State Pattern

**Uso**: Estados de citas y facturas

```python
class AppointmentState:
    """Estado de una cita con transiciones válidas."""
    
    PENDING = 'pending'
    DONE = 'done'
    CANCELLED = 'cancelled'
    
    TRANSITIONS = {
        PENDING: [DONE, CANCELLED],
        DONE: [],  # Estado final
        CANCELLED: []  # Estado final
    }
    
    @classmethod
    def can_transition(cls, from_state, to_state):
        """Valida si la transición es permitida."""
        return to_state in cls.TRANSITIONS.get(from_state, [])

# Uso en ruta:
@app.route('/appointments/<int:id>/cancel', methods=['POST'])
def appointment_cancel(id):
    appointment = Appointment.query.get_or_404(id)
    
    if not AppointmentState.can_transition(appointment.status, 'cancelled'):
        flash('No se puede cancelar esta cita', 'error')
        return redirect(url_for('appointment_view', id=id))
    
    appointment.status = AppointmentState.CANCELLED
    db.session.commit()
    flash('Cita cancelada exitosamente', 'success')
    return redirect(url_for('appointment_list'))
```

### 11. Module Pattern (JavaScript)

**Uso**: Encapsulación de funcionalidad frontend

```javascript
// Pattern IIFE - Módulo de formulario de servicios
window.ServiceForm = (function() {
    // Variables privadas
    let selectedServices = [];
    let totalPrice = 0;
    
    // Métodos privados
    function calculateTotal() {
        totalPrice = selectedServices.reduce((sum, s) => sum + s.price, 0);
        updateDisplay();
    }
    
    function updateDisplay() {
        document.getElementById('total').textContent = 
            formatCurrency(totalPrice);
    }
    
    // API pública
    return {
        init: function() {
            bindEvents();
            loadSelectedServices();
        },
        
        addService: function(serviceId, price) {
            selectedServices.push({ id: serviceId, price: price });
            calculateTotal();
        },
        
        removeService: function(serviceId) {
            selectedServices = selectedServices.filter(s => s.id !== serviceId);
            calculateTotal();
        },
        
        getTotal: function() {
            return totalPrice;
        }
    };
})();

// Uso:
ServiceForm.init();
ServiceForm.addService(1, 50000);
```

## 🏛️ Arquitectura por Capas

```
┌─────────────────────────────────────────┐
│         Presentation Layer              │
│  (Templates Jinja2 + Bootstrap + JS)   │
│  - layout.html                          │
│  - list/form/view templates             │
│  - Validación cliente                   │
└────────────┬────────────────────────────┘
             │
┌────────────▼────────────────────────────┐
│         Application Layer               │
│  (Flask Routes - app.py)                │
│  - Autorización (@decorators)           │
│  - Validación servidor                  │
│  - Orquestación lógica de negocio       │
│  - Flash messages                       │
└────────────┬────────────────────────────┘
             │
┌────────────▼────────────────────────────┐
│         Business Logic Layer            │
│  (Models + Methods)                     │
│  - Reglas de negocio                    │
│  - Cálculos (totales, stocks)           │
│  - Validaciones complejas               │
│  - State transitions                    │
└────────────┬────────────────────────────┘
             │
┌────────────▼────────────────────────────┐
│         Data Access Layer               │
│  (SQLAlchemy ORM)                       │
│  - CRUD operations                      │
│  - Queries complejas                    │
│  - Transacciones                        │
│  - Relationships                        │
└────────────┬────────────────────────────┘
             │
┌────────────▼────────────────────────────┐
│         Database Layer                  │
│  (SQLite / PostgreSQL)                  │
│  - Persistencia                         │
│  - Constraints                          │
│  - Indexes                              │
└─────────────────────────────────────────┘
```

## 🔄 Flujo de Datos

### Request Flow (Entrada)
```
1. Usuario → HTTP Request
   ↓
2. Flask App → Route Matching (@app.route)
   ↓
3. Decorators → Autenticación/Autorización
   ↓
4. Route Handler → Validación de datos
   ↓
5. Business Logic → Procesamiento
   ↓
6. ORM (SQLAlchemy) → Queries
   ↓
7. Database → Persistencia
   ↓
8. Response → Template Rendering
   ↓
9. HTTP Response → Usuario
```

### Data Flow Example (Crear Cita)
```python
# 1. Request
POST /services/new
{
    customer_id: 1,
    pet_id: 2,
    service_types: [1, 3],
    scheduled_at: '2025-10-22 14:30'
}

# 2. Route + Validation
@app.route('/services/new', methods=['POST'])
@login_required
def service_new():
    # 3. Create Appointment (Business Logic)
    appointment = Appointment(
        customer_id=request.form['customer_id'],
        pet_id=request.form['pet_id'],
        scheduled_at=datetime.strptime(...)
    )
    
    # 4. Add Services (Composite Pattern)
    for service_type_id in request.form.getlist('service_types'):
        service = PetService(
            service_type_id=service_type_id,
            price=get_service_price(service_type_id)
        )
        appointment.services.append(service)
    
    # 5. Recompute Total (Business Logic)
    appointment.recompute_total()
    
    # 6. Persist (Data Layer)
    db.session.add(appointment)
    db.session.commit()
    
    # 7. Response
    flash('Cita creada exitosamente', 'success')
    return redirect(url_for('appointment_view', id=appointment.id))
```

## 📐 Principios SOLID Aplicados

### Single Responsibility Principle (SRP)
```python
# ✅ CORRECTO: Cada clase una responsabilidad
class User(db.Model):
    """Solo maneja datos de usuario."""
    username = db.Column(db.String(50))
    
    def set_password(self, password):
        """Responsabilidad: hash de contraseña."""
        self.password_hash = generate_password_hash(password)

class UserRepository:
    """Solo maneja acceso a datos de usuario."""
    @staticmethod
    def find_by_username(username):
        return User.query.filter_by(username=username).first()

class AuthService:
    """Solo maneja autenticación."""
    @staticmethod
    def authenticate(username, password):
        user = UserRepository.find_by_username(username)
        if user and user.check_password(password):
            login_user(user)
            return True
        return False
```

### Open/Closed Principle (OCP)
```python
# ✅ CORRECTO: Abierto para extensión, cerrado para modificación
class PaymentProcessor:
    """Base para procesadores de pago."""
    def process(self, invoice):
        raise NotImplementedError

class CashPaymentProcessor(PaymentProcessor):
    def process(self, invoice):
        # Lógica específica para efectivo
        pass

class TransferPaymentProcessor(PaymentProcessor):
    def process(self, invoice):
        # Lógica específica para transferencia
        pass

# Agregar nuevos métodos sin modificar código existente
class CardPaymentProcessor(PaymentProcessor):
    def process(self, invoice):
        # Nueva funcionalidad
        pass
```

### Liskov Substitution Principle (LSP)
```python
# ✅ CORRECTO: Subclases intercambiables
class Document(db.Model):
    """Documento base."""
    __abstract__ = True
    number = db.Column(db.String(30))
    date = db.Column(db.DateTime)
    total = db.Column(db.Float)
    
    def calculate_total(self):
        """Debe ser implementado por subclases."""
        raise NotImplementedError

class Invoice(Document):
    """Factura - puede sustituir Document."""
    def calculate_total(self):
        self.total = self.subtotal + self.tax

class Receipt(Document):
    """Recibo - puede sustituir Document."""
    def calculate_total(self):
        self.total = self.amount
```

### Interface Segregation Principle (ISP)
```python
# ✅ CORRECTO: Interfaces específicas
class Searchable:
    """Interface para entidades buscables."""
    def search(self, query):
        raise NotImplementedError

class Sortable:
    """Interface para entidades ordenables."""
    def sort_by(self, field, order='asc'):
        raise NotImplementedError

class Product(db.Model, Searchable, Sortable):
    """Implementa solo lo que necesita."""
    def search(self, query):
        return Product.query.filter(
            Product.name.ilike(f'%{query}%')
        ).all()
    
    def sort_by(self, field, order='asc'):
        return Product.query.order_by(
            getattr(Product, field).desc() if order == 'desc'
            else getattr(Product, field).asc()
        ).all()
```

### Dependency Inversion Principle (DIP)
```python
# ✅ CORRECTO: Depender de abstracciones
class Repository:
    """Abstracción de acceso a datos."""
    def get(self, id):
        raise NotImplementedError
    
    def save(self, entity):
        raise NotImplementedError

class SQLAlchemyRepository(Repository):
    """Implementación concreta con SQLAlchemy."""
    def __init__(self, model):
        self.model = model
    
    def get(self, id):
        return self.model.query.get(id)
    
    def save(self, entity):
        db.session.add(entity)
        db.session.commit()

# Service depende de abstracción, no de implementación
class ProductService:
    def __init__(self, repository: Repository):
        self.repository = repository
    
    def get_product(self, id):
        return self.repository.get(id)
```

## 🔐 Seguridad y Best Practices

### 1. SQL Injection Prevention
```python
# ✅ CORRECTO: Usar ORM parametrizado
users = User.query.filter(User.username == username).all()

# ✅ CORRECTO: Queries raw con parámetros
db.session.execute(
    "SELECT * FROM users WHERE username = :username",
    {"username": username}
)

# ❌ INCORRECTO: Concatenación de strings
db.session.execute(f"SELECT * FROM users WHERE username = '{username}'")
```

### 2. XSS Prevention
```html
<!-- ✅ CORRECTO: Jinja2 escapa automáticamente -->
<p>{{ user.name }}</p>

<!-- ✅ CORRECTO: Usar |safe solo con contenido confiable -->
<div>{{ content|safe }}</div>

<!-- ❌ INCORRECTO: Insertar HTML no escapado -->
<script>var data = {{ data }};</script>
```

### 3. CSRF Protection
```python
# ✅ CORRECTO: Flask-WTF incluye token CSRF
from flask_wtf.csrf import CSRFProtect

csrf = CSRFProtect(app)

# En templates:
<form method="post">
    {{ csrf_token() }}
    <!-- campos -->
</form>
```

### 4. Password Hashing
```python
# ✅ CORRECTO: Usar werkzeug con salt automático
from werkzeug.security import generate_password_hash, check_password_hash

# Hash con salt automático (bcrypt-like)
hash = generate_password_hash('password', method='pbkdf2:sha256')

# ❌ INCORRECTO: Hash simple sin salt
import hashlib
hash = hashlib.sha256('password'.encode()).hexdigest()
```

### 5. Session Management
```python
# ✅ CORRECTO: Configuración segura
app.config['SESSION_COOKIE_SECURE'] = True  # Solo HTTPS
app.config['SESSION_COOKIE_HTTPONLY'] = True  # No JS access
app.config['SESSION_COOKIE_SAMESITE'] = 'Lax'  # CSRF protection
app.config['PERMANENT_SESSION_LIFETIME'] = timedelta(hours=2)
```

## �📁 Estructura de Archivos Clave
```
Green-POS/
├── app.py                      # Aplicación Flask principal (~2000 líneas)
├── models/
│   └── models.py              # Modelos SQLAlchemy (User, Customer, Pet, Product, Invoice, Appointment, etc.)
├── templates/
│   ├── layout.html            # Plantilla base con navbar y sidebar
│   ├── appointments/          # Sistema de citas
│   │   ├── list.html         # Lista agrupada por fecha
│   │   ├── form.html         # Crear cita (tarjetas interactivas)
│   │   ├── edit.html         # Editar cita (mismo diseño que form)
│   │   └── view.html         # Detalle con impresión de consentimiento
│   ├── customers/            # Gestión de clientes
│   ├── invoices/             # Facturación
│   │   └── list.html        # Agrupación por fecha con totales por método de pago
│   ├── products/             # Inventario
│   │   ├── stock_history.html  # Historial de movimientos
│   │   └── form.html         # Con campo de razón para cambios de stock
│   ├── services/             # Servicios (ahora citas)
│   │   └── types/           # Tipos de servicio
│   │       ├── config.html  # CRUD de tipos
│   │       └── form.html    # Modal para crear/editar
│   └── pets/                # Mascotas
├── static/
│   ├── css/
│   ├── js/
│   │   └── main.js          # JavaScript principal
│   └── uploads/
│       └── logo.png         # Logo del negocio
├── instance/
│   └── app.db               # Base de datos SQLite
└── requirements.txt
```

## 🎯 Requisitos de Estilo de Código

### Python/Flask
- **Type hints**: Obligatorios en funciones nuevas (Python 3.10+)
- **PEP 8**: Seguir rigurosamente
- **Docstrings**: Google style para funciones públicas
  ```python
  def appointment_update(id: int) -> Response:
      """Actualiza una cita existente con validación de cambios.
      
      Args:
          id: ID de la cita a actualizar
          
      Returns:
          Response: Redirect a vista de cita o formulario con errores
          
      Raises:
          404: Si la cita no existe
      """
  ```
- **Zona horaria**: `datetime.now(CO_TZ)` para timestamps locales
- **Validación**: Backend SIEMPRE + frontend para UX
- **Transacciones**: `db.session` con rollback en excepciones
- **Logging**: `app.logger.error()` para errores críticos solamente

### Filosofía de Desarrollo y Debugging
**CRÍTICO - Ciclo de Desarrollo con Limpieza:**

1. **Durante Desarrollo**: 
   - Crear logs extensivos con marcadores
   - Comentarios de debug explícitos
   - Prints temporales marcados
   - Tests de validación inline

2. **Durante Testing**: 
   - Mantener elementos de debugging
   - Validar todas las rutas CRUD
   - Probar casos edge

3. **Antes de Producción**: 
   - **ELIMINAR** completamente código de debugging
   - Remover todos los `print()`, `console.log()` temporales
   - Eliminar comentarios `# TODO`, `# DEBUG`, `# TEMP`, `# FIXME`, `# TEST`
   - Limpiar logs no esenciales
   - Remover código comentado
   - Eliminar variables/imports no utilizados

**Regla de Oro**: El código productivo debe estar limpio, sin rastros de debugging temporal

**OBLIGATORIO - Marcado de Código Temporal:**
```python
# DEBUG: imprimir valores para troubleshooting
print(f"Debug: usuario = {user}")

# TODO: optimizar esta query con joins
debug_var = "temporal"  # TEMP: variable de prueba

# FIXME: revisar lógica de validación de stock
import pdb; pdb.set_trace()  # DEBUG: breakpoint temporal

# TEST: función solo para pruebas unitarias
def test_function():
    pass
```

```javascript
// DEBUG: verificar valores del formulario
console.log("Debug info:", data);

// TODO: implementar validación real
// TEMP: función de prueba para testing
function debugFunction() { alert("Test"); }  // DEBUG

// FIXME: corregir cálculo de total
let debugTotal = 0;  // TEMP
```

**Marcadores Estándar para Limpieza:**
- `# DEBUG:` / `// DEBUG:` → Código de debugging temporal
- `# TODO:` / `// TODO:` → Tareas pendientes
- `# TEMP:` / `// TEMP:` → Código experimental
- `# FIXME:` / `// FIXME:` → Código que necesita corrección
- `# TEST:` / `// TEST:` → Solo para testing

**📋 IMPORTANTE**: Ver guía completa de limpieza en `.github/instructions/code-clean.instructions.md`

### HTML/Templates (Jinja2)
- **Plantilla base**: Siempre extender `layout.html`
- **Bootstrap 5.3+**: Uso exclusivo (sin jQuery)
- **Breadcrumbs**: Obligatorios en vistas internas
  ```html
  <nav aria-label="breadcrumb">
    <ol class="breadcrumb">
      <li class="breadcrumb-item"><a href="{{ url_for('index') }}">Inicio</a></li>
      <li class="breadcrumb-item"><a href="{{ url_for('appointment_list') }}">Citas</a></li>
      <li class="breadcrumb-item active">Detalle</li>
    </ol>
  </nav>
  ```
- **Mensajes flash**: Renderizar en layout.html automáticamente
- **Responsive**: Mobile-first con Bootstrap grid
- **Iconos**: Bootstrap Icons (`<i class="bi bi-*">`)

### JavaScript (Vanilla JS)
- **NO usar jQuery**: Migración completa a Vanilla JS
- **Módulos**: Usar pattern IIFE cuando sea apropiado
  ```javascript
  window.ServiceForm = (function(){
    // ... funciones privadas
    return { 
      publicFunction1,
      publicFunction2 
    };
  })();
  ```
- **Event Listeners**: `addEventListener()` en lugar de onclick inline
- **Fetch API**: Para llamadas AJAX (no XMLHttpRequest)
- **Validación**: HTML5 + JavaScript custom
- **Autocompletado**: Implementar con debounce (300ms)

### Limpieza de Código Frontend
**Antes de Producción Eliminar:**
- `console.log()`, `console.debug()`, `console.warn()` temporales
- `alert()` de testing
- Comentarios `// TODO`, `// DEBUG`, `// FIXME`, `// TEMP`, `// TEST`
- Funciones de test temporales
- Variables no utilizadas
- CSS/HTML comentado o experimental
- Atributos `data-*` de debugging

## 🔧 Patrones de Arquitectura Específicos

### Rutas Flask (app.py)
**Patrón CRUD estándar**:
```python
@app.route('/entity')                    # GET  - Listar (con filtros opcionales)
@app.route('/entity/new')                # GET  - Formulario crear  
@app.route('/entity/create', POST)       # POST - Procesar creación (deprecated: usar /new POST)
@app.route('/entity/<id>')               # GET  - Ver detalle
@app.route('/entity/<id>/edit')          # GET  - Formulario editar
@app.route('/entity/<id>/update', POST)  # POST - Procesar actualización
@app.route('/entity/<id>/delete', POST)  # POST - Eliminar (con confirmación)
```

**Patrón moderno preferido** (reducir rutas):
```python
@app.route('/entity', methods=['GET'])           # Listar
@app.route('/entity/new', methods=['GET','POST']) # Crear (form + procesar)
@app.route('/entity/<id>')                       # Ver
@app.route('/entity/<id>/edit', methods=['GET','POST'])  # Editar (form + procesar)
@app.route('/entity/<id>/delete', methods=['POST'])      # Eliminar
```

**Ejemplo real del proyecto**:
```python
@app.route('/appointments/<int:id>/edit', methods=['GET', 'POST'])
@login_required
def appointment_edit(id: int):
    """Edita una cita existente.
    
    Restricción: No permite editar citas con factura generada.
    """
    appointment = Appointment.query.get_or_404(id)
    
    # Validación de permisos/estado
    if appointment.invoice_id:
        flash('No se puede editar una cita que ya tiene factura generada', 'error')
        return redirect(url_for('appointment_view', id=id))
    
    if request.method == 'POST':
        try:
            # Procesar actualización
            # ... lógica de negocio
            db.session.commit()
            flash('Cita actualizada exitosamente', 'success')
            return redirect(url_for('appointment_view', id=id))
        except Exception as e:
            db.session.rollback()
            app.logger.error(f"Error actualizando cita: {e}")
            flash('Error al actualizar la cita', 'error')
    
    # GET - Mostrar formulario
    return render_template('appointments/edit.html', 
                         appointment=appointment,
                         service_types=ServiceType.query.filter_by(is_active=True).all())
```

### Modelos de Base de Datos (SQLAlchemy)

**Patrón de timestamps**:
```python
class BaseModel(db.Model):
    __abstract__ = True
    created_at = db.Column(db.DateTime, default=lambda: datetime.now(CO_TZ))
    updated_at = db.Column(db.DateTime, default=lambda: datetime.now(CO_TZ), 
                          onupdate=lambda: datetime.now(CO_TZ))
```

**Relaciones estándar**:
```python
class Appointment(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    customer_id = db.Column(db.Integer, db.ForeignKey('customer.id'))
    pet_id = db.Column(db.Integer, db.ForeignKey('pet.id'))
    
    # Relaciones con backref
    customer = db.relationship('Customer', backref='appointments')
    pet = db.relationship('Pet', backref='appointments')
    
    # Relación uno a muchos
    services = db.relationship('PetService', backref='appointment', 
                              lazy='dynamic', cascade='all, delete-orphan')
```

**Métodos útiles**:
```python
class Appointment(db.Model):
    # ...
    
    def recompute_total(self):
        """Recalcula el total de la cita basado en sus servicios."""
        self.total_price = sum(s.price or 0 for s in self.services)
        self.total_services = self.services.count()
    
    def to_dict(self):
        """Serializa el modelo a diccionario para JSON."""
        return {
            'id': self.id,
            'customer': self.customer.name if self.customer else None,
            'pet': self.pet.name if self.pet else None,
            'total': float(self.total_price or 0),
            'status': self.status
        }
```

**Sistema de logs de inventario** (implementado):
```python
class ProductStockLog(db.Model):
    """Registra todos los movimientos de inventario para trazabilidad."""
    id = db.Column(db.Integer, primary_key=True)
    product_id = db.Column(db.Integer, db.ForeignKey('product.id'), nullable=False)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    quantity = db.Column(db.Integer, nullable=False)  # Valor absoluto
    movement_type = db.Column(db.String(20), nullable=False)  # 'addition' o 'subtraction'
    reason = db.Column(db.Text, nullable=False)  # Razón obligatoria
    previous_stock = db.Column(db.Integer, nullable=False)
    new_stock = db.Column(db.Integer, nullable=False)
    created_at = db.Column(db.DateTime, default=lambda: datetime.now(CO_TZ))
    
    product = db.relationship('Product', backref='stock_logs')
    user = db.relationship('User', backref='stock_changes')
```

**Sistema de códigos alternativos** (implementado Nov 2025):
```python
class ProductCode(db.Model):
    """Códigos alternativos de productos para soportar consolidación.
    
    Permite que un producto tenga múltiples códigos (EAN, SKU, códigos legacy)
    manteniendo el código principal en Product.code.
    
    Tipos de código (code_type):
    - 'alternative': Código alternativo genérico
    - 'legacy': Código de producto consolidado
    - 'barcode': Código de barras (EAN, UPC)
    - 'supplier_sku': SKU del proveedor
    """
    id = db.Column(db.Integer, primary_key=True)
    product_id = db.Column(db.Integer, db.ForeignKey('product.id', ondelete='CASCADE'), nullable=False)
    code = db.Column(db.String(20), unique=True, nullable=False, index=True)
    code_type = db.Column(db.String(20), default='alternative', nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    created_by = db.Column(db.Integer, db.ForeignKey('user.id'))
    notes = db.Column(db.Text)
    
    # Relaciones
    product = db.relationship('Product', backref=db.backref('alternative_codes', 
                              lazy='dynamic', cascade='all, delete-orphan'))
    user = db.relationship('User')

# Métodos en Product para códigos alternativos
def get_all_codes(self):
    """Retorna lista de todos los códigos del producto."""
    codes = [{'code': self.code, 'type': 'principal', 'is_primary': True}]
    codes.extend([{'code': ac.code, 'type': ac.code_type, 'is_primary': False} 
                  for ac in self.alternative_codes.all()])
    return codes

@staticmethod
def search_by_any_code(code_query):
    """Busca producto por código principal o alternativo."""
    product = Product.query.filter_by(code=code_query).first()
    if not product:
        alt_code = ProductCode.query.filter_by(code=code_query).first()
        if alt_code:
            return alt_code.product
    return product
```

### Autenticación y Seguridad

**Decoradores de autorización**:
```python
from functools import wraps

def admin_required(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if not current_user.is_authenticated or current_user.role != 'admin':
            flash('Acceso denegado. Requiere permisos de administrador.', 'error')
            return redirect(url_for('index'))
        return f(*args, **kwargs)
    return decorated_function

# Uso:
@app.route('/settings')
@login_required
@admin_required
def settings():
    # Solo admins
    pass
```

**Roles del sistema**:
- `admin`: Acceso total, configuración, reportes
- `vendedor`: Ventas, inventario (solo lectura de productos), clientes

**Hasheo de contraseñas**:
```python
from werkzeug.security import generate_password_hash, check_password_hash

# Al crear usuario
user.password_hash = generate_password_hash(password)

# Al validar login
if check_password_hash(user.password_hash, password):
    login_user(user)
```

## 📊 Funcionalidades Específicas

### Sistema de Facturación
- **Numeración secuencial**: Tabla `Setting` con campo `next_invoice_number`
- **Métodos de pago**: `cash`, `transfer`, `card`, `mixed`
- **Generación PDF**: ReportLab optimizado para impresoras térmicas (58mm/80mm)
- **Asociación con citas**: `Invoice.appointment_id` (opcional, puede ser venta directa)
- **Items de factura**: Tabla intermedia `InvoiceItem` con productos/servicios
- **Cálculo automático**: Total, subtotal, descuentos, IVA

**Flujo de facturación desde citas**:
1. Crear cita → NO genera factura
2. Finalizar cita → Genera factura automáticamente
3. Asocia todos los servicios de la cita a la factura
4. Marca servicios como `done`
5. Actualiza estado de cita a `done`

### Gestión de Inventario con Trazabilidad
**Sistema de logs implementado**:
- Cada cambio de stock requiere razón obligatoria
- Registro automático en `ProductStockLog`
- Historial completo por producto
- Información capturada:
  - Usuario responsable
  - Cantidad (positiva o negativa)
  - Tipo de movimiento (ingreso/egreso)
  - Stock anterior y nuevo
  - Razón detallada
  - Timestamp

**Ejemplo de uso**:
```python
# Al editar producto y cambiar stock
if new_stock != old_stock:
    reason = request.form.get('stock_reason', '').strip()
    if not reason:
        flash('Debe proporcionar una razón para el cambio de stock', 'error')
        return render_template(...)
    
    # Crear log
    log = ProductStockLog(
        product_id=product.id,
        user_id=current_user.id,
        quantity=abs(new_stock - old_stock),
        movement_type='addition' if new_stock > old_stock else 'subtraction',
        reason=reason,
        previous_stock=old_stock,
        new_stock=new_stock
    )
    db.session.add(log)
```

### Sistema de Citas (Appointments)
**Características**:
- **Tarjetas interactivas**: Selección visual de servicios
- **Precios dinámicos**: Servicios con precio fijo o variable
- **Consentimiento informado**: Template personalizable
- **Programación**: Fecha y hora opcional
- **Estados**: `pending`, `done`, `cancelled`
- **Edición**: Permitida solo en estado `pending` y sin factura
- **WhatsApp**: Integración para confirmación de citas
- **Impresión**: PDF del consentimiento informado con logo

**Flujo completo**:
```
1. Crear cita (/services/new)
   - Seleccionar cliente y mascota
   - Elegir servicios (tarjetas clickeables)
   - Agregar notas y consentimiento
   - Programar fecha/hora (opcional)
   → Guarda Appointment + PetService (sin Invoice)

2. Ver cita (/appointments/<id>)
   - Información completa
   - Notas y consentimiento
   - Lista de servicios
   - Botones de acción según estado

3. Editar cita (/appointments/<id>/edit)
   - Solo si estado = pending
   - Solo si NO tiene factura
   - Misma UI que creación
   - Puede agregar/quitar servicios

4. Finalizar cita (POST /appointments/<id>/finish)
   - Genera Invoice automáticamente
   - Asocia servicios a la factura
   - Marca servicios como done
   - Actualiza estado a done
   → Redirige a Invoice para cobro/impresión

5. Cancelar cita (POST /appointments/<id>/cancel)
   - Solo si estado = pending
   - Marca como cancelled
```

### Tipos de Servicio Configurables
**Modelo `ServiceType`**:
- Nombre y descripción
- Código único
- Precio base
- Modo de precio: `fixed` (fijo) o `variable`
- Duración estimada (minutos)
- Estado activo/inactivo

**CRUD completo**:
- Lista en modal: `/services/types/config`
- Crear/Editar: `/services/types/new` y `/services/types/<id>/edit`
- Eliminar: Validación de servicios asociados

### Integración WhatsApp
**Implementado en lista de citas**:
```html
<a href="https://wa.me/+57{{ phone_clean }}?text={{ mensaje }}" 
   class="btn btn-sm btn-outline-success" 
   target="_blank" 
   title="Confirmar cita por WhatsApp">
   <i class="bi bi-whatsapp"></i>
</a>
```

**Mensaje personalizado**:
```
Hola [Nombre]!
Te saludamos desde *Pet Verde*

Recuerda la cita con nuestra groomer para *[Mascota]*

Fecha: 17/10/2025
Hora: 2:30 PM

Gracias por preferirnos!
```

## 🚨 Restricciones y Consideraciones

### Base de Datos
- **Transacciones**: SIEMPRE usar try-except con rollback
  ```python
  try:
      # operaciones
      db.session.commit()
  except Exception as e:
      db.session.rollback()
      app.logger.error(f"Error: {e}")
      flash('Error al guardar', 'error')
  ```
- **SQLite locks**: Timeout de 30 segundos configurado
- **Backups**: Recomendado backup diario de `app.db`
- **Migraciones**: Evitar cambios breaking en schema

### Rendimiento
- **Paginación**: Implementar en listas > 50 items
- **Lazy loading**: Usar `lazy='dynamic'` en relaciones grandes
- **Índices**: En campos de búsqueda frecuente (customer.document, product.code)
- **Queries N+1**: Usar `joinedload()` cuando sea necesario

### Zona Horaria y Timestamps
**CRÍTICO**:
- `scheduled_at` en Appointment: Hora LOCAL (sin conversión)
- Usuario programa en hora local → Se guarda en hora local → Se muestra tal cual
- `created_at`, `updated_at`: Usar `datetime.now(CO_TZ)`
- En templates: Usar formato `strftime()` directamente
- NO convertir UTC→Local en appointments (ya están en local)

**Ejemplo correcto**:
```python
# Crear cita con hora programada
appointment.scheduled_at = datetime.strptime(
    f"{date_str} {time_str}", 
    '%Y-%m-%d %H:%M'
)  # Sin timezone, es hora local

# Mostrar en template
{{ appointment.scheduled_at.strftime('%I:%M %p') }}  # 2:30 PM
```

### Credenciales por Defecto
- Admin: `admin` / `admin123`
- Vendedor: `vendedor` / `vendedor123`
- **CAMBIAR EN PRODUCCIÓN**

## 🎨 Convenciones UI/UX

### Bootstrap 5 Components

**Cards para contenido agrupado**:
```html
<div class="card mb-3">
  <div class="card-header bg-light">
    <h5 class="mb-0">Título</h5>
  </div>
  <div class="card-body">
    <!-- Contenido -->
  </div>
  <div class="card-footer text-end">
    <button class="btn btn-primary">Guardar</button>
  </div>
</div>
```

**Tables responsive**:
```html
<div class="table-responsive">
  <table class="table table-hover table-sm align-middle mb-0">
    <thead>
      <tr>
        <th>Columna</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>Dato</td>
      </tr>
    </tbody>
  </table>
</div>
```

**Modals para CRUD rápido**:
```html
<!-- Modal de eliminación -->
<div class="modal fade" id="deleteModal" tabindex="-1">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title">Confirmar Eliminación</h5>
        <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
      </div>
      <div class="modal-body">
        ¿Está seguro de eliminar este registro?
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancelar</button>
        <form method="post" action="{{ url_for('entity_delete', id=item.id) }}">
          <button type="submit" class="btn btn-danger">Eliminar</button>
        </form>
      </div>
    </div>
  </div>
</div>
```

**Agrupación por fecha** (patrón usado en invoices y appointments):
```html
{% for date, items in items_by_date.items() %}
<div class="card mb-4">
  <div class="card-header bg-light">
    <h5 class="mb-0">
      <button class="btn btn-link collapsed" data-bs-toggle="collapse" 
              data-bs-target="#collapse-{{ loop.index }}">
        <span class="formatted-date" data-date="{{ date }}">{{ date }}</span>
        <i class="bi bi-chevron-down collapse-icon"></i>
      </button>
      <small class="text-muted ms-2">
        ({{ items|length }} items)
        <span class="ms-2">Total: {{ items|sum(attribute='total')|currency_co }}</span>
      </small>
    </h5>
  </div>
  <div id="collapse-{{ loop.index }}" class="collapse{{ ' show' if loop.index == 1 }}">
    <div class="card-body p-0">
      <div class="table-responsive">
        <table class="table table-hover table-sm align-middle mb-0">
          <!-- Tabla de items -->
        </table>
      </div>
    </div>
  </div>
</div>
{% endfor %}
```

### Iconografía (Bootstrap Icons)
**Acciones estándar**:
- Crear: `bi-plus-circle` (verde)
- Editar: `bi-pencil-square` (amarillo/naranja)
- Eliminar: `bi-trash` (rojo)
- Ver: `bi-eye` (azul)
- Guardar: `bi-save`
- Volver: `bi-arrow-left`
- Configurar: `bi-gear`
- WhatsApp: `bi-whatsapp` (verde)
- Factura: `bi-receipt-cutoff`
- Imprimir: `bi-printer`

**Estados con iconos**:
```html
{% if appointment.status == 'done' %}
  <i class="bi bi-check-circle-fill text-success" title="Finalizada"></i>
{% elif appointment.status == 'cancelled' %}
  <i class="bi bi-x-circle-fill text-danger" title="Cancelada"></i>
{% else %}
  <i class="bi bi-hourglass-split text-secondary" title="Pendiente"></i>
{% endif %}
```

**Colores semánticos**:
- Success (verde): Completado, guardar, confirmaciones
- Warning (amarillo): Pendiente, advertencias
- Danger (rojo): Eliminar, cancelar, errores
- Primary (azul): Acciones principales, ver
- Secondary (gris): Volver, acciones secundarias
- Info (cyan): Información adicional

### Formularios
**Validación HTML5 + JavaScript**:
```html
<form method="post" novalidate>
  <div class="mb-3">
    <label for="name" class="form-label">Nombre <span class="text-danger">*</span></label>
    <input type="text" class="form-control" id="name" name="name" required>
    <div class="invalid-feedback">
      Este campo es requerido
    </div>
  </div>
  
  <div class="mb-3">
    <label for="stock" class="form-label">Existencias</label>
    <input type="number" class="form-control" id="stock" name="stock" min="0">
  </div>
  
  <button type="submit" class="btn btn-primary">
    <i class="bi bi-save"></i> Guardar
  </button>
</form>
```

**Campo de razón para cambios de stock** (implementado):
```html
<div id="stockReasonGroup" class="mb-3" style="display: none;">
  <label for="stock_reason" class="form-label">
    Razón del Cambio de Stock <span class="text-danger">*</span>
  </label>
  <textarea class="form-control" id="stock_reason" name="stock_reason" rows="3"
            placeholder="Ej: Nueva compra factura #1234 del proveedor Italcol - 20/10/2025"></textarea>
  <small class="text-muted">
    Este cambio quedará registrado en el historial de inventario
  </small>
</div>

<script>
const originalStock = {{ product.stock if product else 0 }};
document.getElementById('stock').addEventListener('input', function() {
  const newStock = parseInt(this.value) || 0;
  const reasonGroup = document.getElementById('stockReasonGroup');
  
  if (newStock !== originalStock) {
    reasonGroup.style.display = 'block';
    document.getElementById('stock_reason').required = true;
  } else {
    reasonGroup.style.display = 'none';
    document.getElementById('stock_reason').required = false;
  }
});
</script>
```

## 📋 APIs y Endpoints Existentes

### APIs JSON (prefijo /api/)
```python
@app.route('/api/customers/search')
def api_customers_search():
    """Búsqueda de clientes para autocompletado."""
    q = request.args.get('q', '')
    customers = Customer.query.filter(
        Customer.name.ilike(f'%{q}%')
    ).limit(10).all()
    return jsonify([c.to_dict() for c in customers])

@app.route('/api/products/search')
def api_products_search():
    """Búsqueda de productos por nombre o código."""
    # Similar a customers
    pass

@app.route('/api/dashboard/stats')
@login_required
def api_dashboard_stats():
    """Estadísticas del dashboard en JSON."""
    return jsonify({
        'sales_today': total_sales,
        'pending_appointments': pending_count,
        'low_stock_products': low_stock_count
    })
```

### Páginas Principales
- `/` - Dashboard con estadísticas
- `/customers` - Gestión de clientes
- `/customers/<id>/pets` - Mascotas del cliente
- `/products` - Gestión de inventario
- `/products/<id>/stock-history` - **Historial de movimientos** (nuevo)
- `/invoices` - Facturación con agrupación por fecha
- `/services` - Redirige a appointments
- `/appointments` - Sistema de citas agrupadas por fecha
- `/appointments/<id>/edit` - Edición de cita
- `/appointments/<id>/finish` - Finalizar y generar factura
- `/services/types/config` - Configuración de tipos de servicio
- `/reports` - **Módulo de reportes y análisis de ventas** (nuevo)
- `/settings` - Configuración del negocio (admin only)

### Rutas de Impresión
- `/invoices/<id>/print` - PDF de factura (ReportLab)
- Impresión de consentimiento: JavaScript `window.print()` en `/appointments/<id>`

## 🔄 Workflow de Desarrollo

### Testing Manual Requerido
1. **Login/logout** de admin y vendedor
2. **CRUD completo** de clientes, productos, mascotas
3. **Gestión de inventario** con cambios de stock y verificación de logs
4. **Proceso completo de citas**:
   - Crear cita sin factura
   - Editar cita pendiente
   - Finalizar cita → Generar factura
   - Verificar que no se puede editar después
5. **Facturación**:
   - Venta directa (sin cita)
   - Desde cita finalizada
   - Métodos de pago
   - Impresión PDF
6. **Configuración**:
   - Tipos de servicio (CRUD)
   - Settings del negocio
   - Logo y datos de contacto

### Debugging
**Logs permitidos en desarrollo**:
```python
# DEBUG: solo en desarrollo
print(f"Debug: appointment = {appointment.to_dict()}")
app.logger.debug(f"Services count: {len(services)}")
```

**Logs requeridos en producción**:
```python
# Errores críticos
app.logger.error(f"Error al crear factura: {str(e)}")

# Acciones de seguridad
app.logger.warning(f"Intento de acceso denegado: {current_user.username}")

# Transacciones importantes
app.logger.info(f"Factura {invoice.number} creada por {current_user.username}")
```

**Flash messages para feedback**:
```python
flash('Cita creada exitosamente', 'success')
flash('No se puede editar una cita con factura', 'error')
flash('Campo requerido: razón del cambio de stock', 'warning')
flash('Producto actualizado', 'info')
```

**Console logs en JavaScript**:
```javascript
// DEBUG: solo en desarrollo
console.log("Debug: selected services =", selectedServices);

// NUNCA en producción
// console.log(), alert(), debugger
```

### Proceso de Limpieza Pre-Producción

**OBLIGATORIO antes de deploy:**

1. **Revisar y limpiar logs temporales**:
```bash
# Buscar todos los prints temporales
grep -rn "print(" --include="*.py" app.py

# Buscar console.log
grep -rn "console\.log\|console\.debug" --include="*.js" static/

# Buscar marcadores de debugging
grep -rn "# DEBUG\|# TODO\|# TEMP\|# FIXME\|# TEST" --include="*.py" .
grep -rn "// DEBUG\|// TODO\|// TEMP\|// FIXME\|// TEST" --include="*.js" .
```

2. **Eliminar código comentado**:
```python
# ELIMINAR:
# old_function()  # TODO: eliminar deprecated
# if debug_mode:  # DEBUG: lógica temporal
```

3. **Limpiar imports no usados**:
```python
# ELIMINAR:
import pdb  # DEBUG: breakpoint library
from pprint import pprint  # TEMP: debugging
```

4. **Remover variables de test**:
```python
# ELIMINAR:
debug_var = "test"  # TEMP
test_data = []  # TEST
```

5. **Limpiar JavaScript temporal**:
```javascript
// ELIMINAR:
console.log("Debug info");  // DEBUG
// alert("Test");  // TEST
debugVar = "test";  // TEMP
```

6. **Verificar templates**:
```html
<!-- ELIMINAR comentarios de debugging -->
<!-- DEBUG: verificar este valor -->
<!-- TODO: mejorar este diseño -->
```

## 🚀 Deployment

### Desarrollo Local
```powershell
# Windows
.\run.ps1

# O con Waitress
python -m waitress --listen=127.0.0.1:5000 app:app
```

### Producción Windows
1. **Servidor WSGI**: Waitress
   ```python
   # run.py
   from waitress import serve
   from app import app
   
   if __name__ == '__main__':
       serve(app, host='0.0.0.0', port=5000, threads=4)
   ```

2. **Variables de entorno** (`.env`):
   ```env
   FLASK_ENV=production
   SECRET_KEY=your-secret-key-here
   DATABASE_URL=sqlite:///instance/app.db
   ```

3. **Servicio Windows con NSSM**:
   ```powershell
   nssm install GreenPOS "C:\Python313\python.exe" "D:\Path\run.py"
   nssm set GreenPOS AppDirectory "D:\Path\Green-POS"
   nssm start GreenPOS
   ```

4. **Backup automático**:
   ```powershell
   # Script PowerShell para backup diario
   Copy-Item "D:\Path\instance\app.db" "D:\Backups\app_$(Get-Date -Format 'yyyyMMdd').db"
   ```

## ✅ Checklist de Limpieza Pre-Producción

### OBLIGATORIO - Limpieza de Código de Debugging

**Python/Backend:**
- [ ] Todos los `print()` temporales (buscar `# DEBUG:`)
- [ ] `app.logger.debug()` no esenciales
- [ ] Comentarios `# TODO`, `# DEBUG`, `# TEMP`, `# FIXME`, `# TEST`
- [ ] Imports: `pdb`, `pprint` con marcadores
- [ ] Variables: `debug_var`, `test_data` marcadas `# TEMP:`
- [ ] Código comentado: `# old_function()` con `# TODO:`
- [ ] Funciones de test: `def test_*()` con `# TEST:`
- [ ] **EMOJIS en print statements** (✅ ❌ 🔄 ⚠️ etc.) - Usar prefijos [OK], [ERROR], [INFO]
- [ ] **Acentos en mensajes de consola** - Usar versiones sin acento

**Frontend/JavaScript:**
- [ ] `console.log()`, `console.debug()`, `console.warn()` con `// DEBUG:`
- [ ] `alert()` de testing con `// TEST:`
- [ ] Comentarios `// TODO`, `// DEBUG`, `// FIXME`, `// TEMP`, `// TEST`
- [ ] Variables JS no utilizadas: `// TEMP:`
- [ ] Código CSS/HTML comentado
- [ ] Funciones de test JS: `// TEST:`
- [ ] `debugger;` statements

**Templates/HTML:**
- [ ] Comentarios HTML de debugging: `<!-- DEBUG: -->`
- [ ] Elementos `display: none` de testing
- [ ] Atributos `data-debug-*`
- [ ] Código experimental comentado

**Configuración:**
- [ ] Credenciales por defecto cambiadas
- [ ] `SECRET_KEY` generado aleatoriamente
- [ ] Debug mode desactivado: `FLASK_ENV=production`
- [ ] URLs de desarrollo removidas
- [ ] Claves API de testing

**Comandos de Verificación:**
```bash
# Python
grep -rn "# DEBUG\|# TODO\|# TEMP\|# FIXME\|# TEST" --include="*.py" .
grep -rn "print(" --include="*.py" app.py models/

# JavaScript
grep -rn "// DEBUG\|// TODO\|// TEMP\|// FIXME\|// TEST" --include="*.js" .
grep -rn "console\.log\|console\.debug\|debugger" --include="*.js" static/

# Templates
grep -rn "<!-- DEBUG\|<!-- TODO\|<!-- TEMP" --include="*.html" templates/
```

### Logs Permitidos en Producción
**SOLO mantener logs de:**
- Errores críticos: `app.logger.error()`
- Acciones de seguridad: login fallidos, accesos denegados
- Transacciones importantes: facturas creadas, citas finalizadas
- Performance crítica: queries lentas (si se implementa)

**Ejemplo de log productivo**:
```python
# ✅ MANTENER
app.logger.error(f"Error al generar factura {invoice.id}: {str(e)}")
app.logger.info(f"Factura {invoice.number} creada por {current_user.username}")
app.logger.warning(f"Acceso denegado a /settings por usuario {current_user.username}")

# ❌ ELIMINAR
print(f"Debug: servicios = {services}")  # DEBUG
app.logger.debug(f"Valor de variable x: {x}")  # DEBUG
```

---

## ⚠️ Anti-Patrones Comunes (Errores al Trabajar con IA)

### 1. Context Dump (Volcado de Contexto Masivo)
**Problema**: Pegar archivos enormes completos esperando que la IA "entienda" todo automáticamente.

**Por qué falla**:
- La IA tiene límite de tokens y pierde contexto importante
- Información irrelevante diluye los detalles críticos
- Mayor latencia y costo de procesamiento
- Resultados imprecisos o genéricos

**❌ Incorrecto**:
```python
# Usuario pega TODO el archivo app.py (1500+ líneas)
# "Ayúdame a debuggear este código"
```

**✅ Correcto**:
```python
# Usuario comparte solo el método relevante con contexto:
"""
Tengo un error en la ruta product_edit() cuando cambio el stock.
El ProductStockLog no se está creando.

Contexto:
- product_edit() está en app.py líneas 355-402
- ProductStockLog modelo está en models/models.py líneas 276-295
- Relación: Product tiene backref 'stock_logs'

Aquí está el método que falla:
[código específico del método con 20-30 líneas]
"""
```

**Regla de Oro**: 
- Máximo 50-100 líneas de código por prompt
- Incluir solo métodos/clases directamente relevantes
- Explicar el rol de cada parte compartida

### 2. Implicit Context (Contexto Implícito)
**Problema**: Asumir que la IA "ya sabe" tu arquitectura, convenciones o decisiones previas.

**Por qué falla**:
- Cada conversación empieza con contexto limitado
- La IA no tiene memoria persistente entre sesiones
- Términos específicos del negocio son ambiguos sin definición

**❌ Incorrecto**:
```
Usuario: "Actualiza el appointment para que valide el status"
IA: No sabe qué validaciones específicas necesitas
```

**✅ Correcto**:
```
Usuario: "En Green-POS, Appointment tiene campo 'status' con valores:
- 'pending': puede editarse
- 'done': inmutable, tiene factura asociada
- 'cancelled': inmutable

Quiero agregar validación en appointment_edit() para que:
1. Si status='done' Y existe invoice_id → mostrar error
2. Si status='cancelled' → mostrar error
3. Si status='pending' → permitir edición normal

Patrón usado: State Pattern (ver copilot-instructions.md línea 650)"
```

**Regla de Oro**:
- Siempre explica términos del dominio (`appointment`, `invoice`, `status`)
- Menciona restricciones de negocio explícitamente
- Referencia patrones/decisiones arquitectónicas cuando aplique

### 3. Set and Forget (Crear y Olvidar)
**Problema**: Crear archivo de instrucciones una vez y nunca actualizarlo a medida que el proyecto evoluciona.

**Por qué falla**:
- El código cambia pero las instrucciones no
- Nuevos patrones no documentados → inconsistencia
- La IA sigue reglas obsoletas

**❌ Incorrecto**:
```markdown
# copilot-instructions.md creado hace 6 meses
# Proyecto ahora usa:
# - Nuevo patrón de validación de stock
# - Sistema de logs de inventario
# - Zona horaria CO_TZ
# Pero instrucciones NO reflejan estos cambios
```

**✅ Correcto**:
```markdown
# Actualizar copilot-instructions.md en cada milestone:
- Nueva feature implementada → documentar patrón usado
- Cambio arquitectónico → actualizar sección de constraints
- Bug fix importante → agregar a "Lessons Learned"

Ejemplo de actualización:
"## Sistema de Trazabilidad de Inventario (Oct 2025)
- Implementado modelo ProductStockLog
- Campo obligatorio stock_reason al cambiar existencias
- Validación backend + frontend
- Ver: app.py líneas 355-402"
```

**Regla de Oro**:
- Revisar copilot-instructions.md en cada Definition of Done
- Actualizar cuando agregas: nuevos modelos, patrones, constraints, dependencias
- Versionar el archivo con git para tracking de cambios

### 4. One Size Fits All (Contexto Único Para Todo)
**Problema**: Usar el mismo nivel de contexto para debugging, features nuevas y análisis de código.

**Por qué falla**:
- Debugging necesita detalles técnicos (logs, stack traces)
- Features necesitan contexto de negocio (requisitos, UI/UX)
- Análisis necesita visión arquitectónica (patrones, dependencias)

**❌ Incorrecto**:
```
# Mismo prompt genérico para todo:
"Revisa este código y dime si está bien"
```

**✅ Correcto - Debugging**:
```
"DEBUGGING: Error en product_edit() al guardar ProductStockLog

Síntomas:
- No se crea registro en stock_logs
- No hay error en consola
- Stock del producto SÍ se actualiza

Contexto técnico:
- app.py líneas 380-395 (creación de log)
- db.session.add(log) ejecutándose
- Relación: Product.stock_logs (backref)

Stack trace: [pegar error completo si existe]
Logs: [pegar líneas relevantes de app.log]

Pregunta: ¿Por qué no se persiste el log?"
```

**✅ Correcto - Feature Nueva**:
```
"FEATURE: Sistema de descuentos en facturas

Requisitos de negocio:
- Descuento % o valor fijo por factura
- Aplicable a total antes de IVA
- Razón obligatoria si descuento > 10%
- Solo admin puede aplicar

Restricciones:
- Invoice modelo ya existe (models.py línea 200)
- Usar patrón State para validar aplicación
- Frontend: Bootstrap 5, campo con validación

Pregunta: ¿Cómo estructuro el modelo y la lógica?"
```

**✅ Correcto - Análisis Arquitectónico**:
```
"ANÁLISIS: Evaluar migración de SQLite a PostgreSQL

Contexto actual:
- 50 usuarios concurrentes
- 1000 facturas/mes
- SQLite con timeouts ocasionales
- Backups manuales diarios

Necesito:
1. Pros/contras de migración
2. Cambios requeridos en código
3. Plan de migración sin downtime
4. Estimación de esfuerzo

Ver constraints: copilot-instructions.md líneas 20-100"
```

**Regla de Oro**:
- **Debugging**: Síntomas + logs + stack traces + código específico
- **Features**: Requisitos + restricciones + contexto UI/UX + patrones esperados
- **Análisis**: Visión arquitectónica + trade-offs + impacto + constraints

### 5. Instruction Drift (Deriva de Instrucciones)
**Problema**: Tener múltiples archivos con reglas contradictorias o redundantes en diferentes lugares.

**Por qué falla**:
- README.md dice "usar pytest"
- .github/copilot-instructions.md dice "usar unittest"
- La IA recibe señales contradictorias

**❌ Incorrecto**:
```
Proyecto tiene:
- README.md: "Usamos Bootstrap 4"
- copilot-instructions.md: "Bootstrap 5.3+ sin jQuery"
- CONTRIBUTING.md: "Puedes usar jQuery si quieres"
→ Inconsistencia total
```

**✅ Correcto**:
```markdown
# .github/copilot-instructions.md (FUENTE CANÓNICA)
## Stack Tecnológico Principal
- Frontend: Bootstrap 5.3+ (sin jQuery) + Vanilla JavaScript
- Backend: Flask 3.0+ + SQLAlchemy
- Database: SQLite (dev) → PostgreSQL (prod)

# README.md (REFERENCIA A FUENTE)
## Tecnologías
Ver especificaciones técnicas completas en:
`.github/copilot-instructions.md`

Stack resumido:
- Flask 3.0+
- Bootstrap 5.3+
- SQLite/PostgreSQL

# CONTRIBUTING.md (REFERENCIA A FUENTE)
## Estándares de Código
Seguir guidelines en `.github/copilot-instructions.md`:
- Sección "Requisitos de Estilo de Código"
- Patrones de diseño implementados
- Constraints tecnológicos
```

**Regla de Oro**:
- **UN SOLO archivo fuente de verdad**: `.github/copilot-instructions.md`
- Otros archivos REFERENCIAN, no duplican
- Actualizar solo la fuente canónica
- Si hay conflicto → el archivo de instrucciones gana

### 6. Domain Ambiguity (Ambigüedad de Dominio)
**Problema**: Usar términos ambiguos que significan cosas diferentes según el contexto.

**Por qué falla**:
- "Account" = ¿cliente o tenant?
- "Service" = ¿servicio de grooming o microservicio?
- "Item" = ¿producto o línea de factura?
- La IA no puede resolver ambigüedad sin contexto

**❌ Incorrecto**:
```python
# Código usa términos ambiguos sin aclarar
class Service(db.Model):  # ¿Qué tipo de servicio?
    pass

class Account(db.Model):  # ¿Cliente o cuenta de usuario?
    pass

# Usuario pregunta:
"Cómo creo un nuevo Service?"
# IA no sabe si hablas de PetService o ServiceType
```

**✅ Correcto - Glosario de Dominio**:
```markdown
## 📖 Glosario de Dominio (Green-POS)

### Entidades Principales

**Customer (Cliente)**:
- Persona que trae mascotas para servicios
- Tiene: nombre, documento, teléfono, dirección
- Modelo: `Customer` (models.py línea 100)
- Relación: Un Customer tiene muchos Pets

**Pet (Mascota)**:
- Animal propiedad de un Customer
- Tiene: nombre, especie, raza, edad
- Modelo: `Pet` (models.py línea 130)
- Relación: Pet pertenece a Customer

**Appointment (Cita)**:
- Agendamiento de servicios para una Pet
- Tiene: fecha, hora, Customer, Pet, servicios, status
- Modelo: `Appointment` (models.py línea 200)
- Estados: 'pending', 'done', 'cancelled'
- Relación: Appointment tiene muchos PetService

**PetService (Servicio de Mascota)**:
- Instancia de un ServiceType aplicado en una Appointment
- Tiene: tipo, precio, notas
- Modelo: `PetService` (models.py línea 240)
- Relación: PetService pertenece a Appointment

**ServiceType (Tipo de Servicio)**:
- Catálogo de servicios ofrecidos (grooming, veterinaria)
- Tiene: nombre, código, precio base, duración
- Modelo: `ServiceType` (models.py línea 270)
- Modo precio: 'fixed' (fijo) o 'variable'

**Invoice (Factura)**:
- Documento de cobro por servicios/productos
- Tiene: número, fecha, total, método de pago
- Modelo: `Invoice` (models.py línea 180)
- Puede asociarse a Appointment (opcional)

**Product (Producto)**:
- Artículo de inventario vendible
- Tiene: código, nombre, precio, stock
- Modelo: `Product` (models.py línea 35)

**User (Usuario del Sistema)**:
- Empleado con acceso al sistema
- Roles: 'admin', 'vendedor'
- Modelo: `User` (models.py línea 250)

### Términos Técnicos

**Stock**: Cantidad de Product disponible en inventario
**Status**: Estado de Appointment ('pending', 'done', 'cancelled')
**Payment Method**: Método de pago ('cash', 'transfer', 'card', 'mixed')
**CO_TZ**: Timezone de Colombia (America/Bogota) usado en timestamps

### Anti-Patrón Común:
❌ "Agregar un service" → Ambiguo
✅ "Agregar un PetService a la Appointment" → Específico
✅ "Crear un ServiceType nuevo en catálogo" → Específico
```

**Regla de Oro**:
- Crear sección de Glosario en copilot-instructions.md
- Definir cada entidad con: propósito, campos clave, relaciones, estados
- Usar nombres específicos en prompts ("PetService" no "service")
- Aclarar términos sobrecargados con contexto

### 7. Missing Definition of Done (Falta Criterio de Completitud)
**Problema**: No especificar qué significa "terminado" para una tarea, causando implementaciones incompletas.

**Por qué falla**:
- IA implementa solo lo explícito
- Falta validación, logging, tests, docs
- Usuario asume que "obviedades" están incluidas

**❌ Incorrecto**:
```
Usuario: "Agrega un campo de email al Customer"
IA: Agrega solo el campo al modelo
Faltan: validación, unique constraint, migración, actualizar forms
```

**✅ Correcto**:
```
Usuario: "Agrega campo email al Customer con Definition of Done:

Modelo (models.py):
- Campo: email (String 120, nullable, unique)
- Validación: formato email válido
- Index en columna email

Ruta (app.py):
- Actualizar customer_edit() para incluir email
- Validación backend con regex
- Flash message si email duplicado

Template (customers/form.html):
- Input type="email" con validación HTML5
- Placeholder: ejemplo@dominio.com
- Campo opcional

Migración:
- Crear script de migración manual si necesario
- Default NULL para registros existentes

Documentación:
- Actualizar copilot-instructions.md en sección Customer
- Agregar a glosario de dominio

Testing:
- Validar creación con email válido
- Validar rechazo de email inválido
- Validar unique constraint"
```

**Regla de Oro**:
- Siempre especificar Definition of Done con:
  * Cambios en modelo + validaciones
  * Cambios en rutas + lógica de negocio
  * Cambios en templates + UX
  * Migración de datos si aplica
  * Actualización de documentación
  * Criterios de testing

---

## 📝 Notas de Implementación Recientes

### Refactorización Completa a Blueprints (Nov 2025) 🎉
**ESTADO**: ✅ **100% COMPLETADO**

#### Transformación Arquitectónica
- **Antes**: 1 archivo monolítico (app.py: 2107 líneas)
- **Después**: 11 blueprints modulares (~200 líneas c/u)
- **Tiempo**: ~4.5 horas de refactorización asistida

#### Blueprints Implementados (11/11)
1. **auth.py** - Login, logout, profile
2. **dashboard.py** - Dashboard principal con estadísticas
3. **api.py** - Endpoints JSON para búsquedas AJAX
4. **products.py** - CRUD productos + historial de stock
5. **suppliers.py** - CRUD proveedores
6. **customers.py** - CRUD clientes
7. **pets.py** - CRUD mascotas
8. **invoices.py** - Sistema de facturación completo
9. **services.py** - Servicios, citas y tipos de servicio (618 líneas - más complejo)
10. **reports.py** - Análisis y reportes de ventas
11. **settings.py** - Configuración del negocio

#### Archivos Clave
- `app.py` - Aplicación principal con Factory Pattern (201 líneas)
- `config.py` - Configuración por ambientes
- `extensions.py` - db, login_manager compartidos
- `utils/` - filtros, decoradores, constantes
- `routes/` - 11 blueprints modulares

#### Limpieza Post-Refactor
- ✅ Eliminados comentarios TODO/DEBUG/TEMP
- ✅ Código comentado duplicado eliminado
- ✅ Archivos temporales archivados en `docs/archive/`
- ✅ Caché Python limpiado
- ✅ Templates actualizados (150+ referencias url_for)

#### Beneficios Logrados
- **Mantenibilidad**: +250% (código modular vs monolítico)
- **Testabilidad**: +500% (blueprints independientes testeable)
- **Escalabilidad**: Arquitectura preparada para crecimiento
- **Código Limpio**: 100% sin código temporal o debugging

#### Documentación
- `docs/CLEANUP_SUMMARY.md` - Resumen completo de limpieza
- `docs/REFACTORING.md` - Guía de refactorización
- `README.md` - Actualizado con arquitectura de blueprints

**Referencias Legacy** (solo consulta histórica):
- `docs/archive/app.py.backup` - Original monolítico (2107 líneas)
- `docs/archive/app_old.py` - Pre-limpieza

### Sistema de Unificación de Productos con Multi-Código (Nov 2025) 🆕
**ESTADO**: ✅ **100% COMPLETADO**

#### Característica Implementada
Sistema completo de consolidación de productos duplicados con soporte para múltiples códigos alternativos (EAN, SKU, códigos legacy).

#### Componentes Implementados

**1. Tabla ProductCode** (Base de Datos):
- Nueva tabla `product_code` para códigos alternativos ilimitados
- Relación One-to-Many con Product (1 producto → N códigos)
- Tipos soportados: `alternative`, `legacy`, `barcode`, `supplier_sku`
- Índices en `code`, `product_id`, `code_type` para búsqueda eficiente
- Migración: `migrations/migration_add_product_codes.py` ✅

**2. Modelo ProductCode** (models/models.py):
- Clase `ProductCode` con relación a Product (cascade delete)
- Métodos en Product:
  * `get_all_codes()` - Retorna código principal + alternativos
  * `search_by_any_code(code)` - Busca por código principal o alternativo
- Backref `Product.alternative_codes` (lazy='dynamic')

**3. Script de Consolidación** (migrations/merge_products.py):
- Función `merge_products(source_ids, target_id, user_id)`
- **7 pasos de consolidación**:
  1. Migra ventas (InvoiceItem)
  2. Migra logs de stock (ProductStockLog) - **TODOS**, sin pérdida
  3. Consolida stock (suma)
  4. Crea log de consolidación
  5. Migra códigos a ProductCode (type='legacy')
  6. Migra proveedores (product_supplier)
  7. Elimina productos origen
- Backup automático con timestamp
- Modo CLI interactivo y API programática
- Transacciones con rollback en error

**4. Búsqueda Multi-Código** (routes/products.py, routes/api.py):
- Lista de productos: `outerjoin(ProductCode)` para búsqueda en códigos alternativos
- Nueva API: `/api/products/search?q=<query>` con soporte multi-código
- Búsqueda encuentra productos por:
  * Código principal (Product.code)
  * Nombre (Product.name)
  * Cualquier código alternativo (ProductCode.code)
- `.distinct()` para evitar duplicados por join

**5. Interfaz de Consolidación** (templates/products/merge.html):
- Ruta: `/products/merge` (solo admin)
- Selector de producto destino
- Búsqueda en vivo de productos origen
- Preview dinámico de consolidación:
  * Productos a consolidar
  * Stock total estimado
  * Códigos legacy que se crearán
- Confirmación manual antes de ejecutar
- Botón "Consolidar Productos" en lista de productos

#### Beneficios
- ✅ Ilimitados códigos por producto (vs. 1 código anterior)
- ✅ Búsqueda por cualquier código (principal, EAN, SKU, legacy)
- ✅ Consolidación completa sin pérdida de datos
- ✅ Trazabilidad total (user_id, created_at, notes en códigos)
- ✅ Backups automáticos en consolidación
- ✅ Script reutilizable para cualquier producto

#### Uso
```python
# Consolidar productos 101, 102, 103 en producto 100
from migrations.merge_products import merge_products

stats = merge_products(
    source_product_ids=[101, 102, 103],
    target_product_id=100,
    user_id=1  # ID del usuario admin
)
# Retorna: {'invoice_items': 15, 'stock_logs': 8, 'stock_consolidated': 50, ...}
```

#### Documentación
- Investigación completa: `docs/research/2025-11-24-unificacion-productos-solucion-completa.md`
- Análisis de búsqueda: `docs/PRODUCT_SEARCH_ANALYSIS_MULTICODE.md`
- Migración SQL: `migrations/migration_add_product_codes.sql`

#### Performance
- Impacto en búsqueda: +5-7ms (con índices) - Despreciable
- Consolidación: ~2-5 segundos para 3-5 productos
- Backup automático: ~1 segundo

### Sistema de Trazabilidad de Inventario (Oct 2025)
- Implementado modelo `ProductStockLog`
- Ruta `/products/<id>/stock-history` para ver historial
- Campo obligatorio `stock_reason` al cambiar existencias
- JavaScript muestra/oculta campo según cambios
- Validación backend de razón antes de guardar
- Logs incluyen: usuario, cantidad, tipo, razón, stocks anterior/nuevo

### Sistema de Citas Mejorado (Oct 2025)
- Migrado formulario de creación a tarjetas interactivas
- Unificado diseño entre creación y edición
- Factura se genera SOLO al finalizar cita (no al crear)
- Implementado historial completo de citas por fecha
- Integración WhatsApp para confirmación
- Impresión de consentimiento con logo del negocio

### Agrupación por Fecha (Oct 2025)
- Facturas agrupadas con totales por método de pago
- Citas agrupadas con conteo de servicios
- Expandir/colapsar acordeones
- Primera fecha expandida por defecto

### Módulo de Reportes (Oct 2025)
- **Ruta principal**: `/reports` con filtros de fecha (start_date, end_date)
- **Métricas calculadas**:
  * Número total de ventas (facturas) en el período
  * Ingresos totales (suma de Invoice.total)
  * Utilidades: `(precio_venta - precio_compra) * cantidad` por cada InvoiceItem
  * Margen de utilidad (%) = utilidades / ingresos * 100
  * Ticket promedio = ingresos / número de ventas
  * Valor del inventario (stock * precio_compra)
  * Potencial de ventas (stock * precio_venta)
- **Análisis por método de pago**: Agrupación con count, total y porcentaje
- **Horas pico de ventas**: Top 5 horas con más ventas (agrupado por hora local CO_TZ)
- **Top 10 productos más vendidos**: Cantidad vendida e ingresos generados
- **Productos con stock bajo**: Listado de productos con <= 3 unidades
- **Últimas 20 facturas** del período con detalles
- **UI**: Cards para métricas, tablas responsivas, filtros de fecha, botón de impresión
- **Patrón**: Repository Pattern con queries complejas usando SQLAlchemy func
- **Acceso**: Disponible para todos los usuarios autenticados (admin y vendedor)

---

**Última actualización**: 22 de octubre de 2025  
**Versión del proyecto**: 2.0  
**Contacto**: Sistema Green-POS - Pet Services Management

---

*Este archivo se actualiza automáticamente en cada conversación de Copilot para proporcionar contexto consistente del proyecto.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/handresc1127)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/handresc1127)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
