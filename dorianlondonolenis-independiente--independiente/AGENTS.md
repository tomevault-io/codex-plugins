# Instrucciones del Proyecto

**Stack Technology:**
- Backend: NestJS + TypeORM + JWT + Passport
- Frontend: Angular 17+ + Standalone Components + Signals
- Databases: MySQL (principal) + SQL Server (proveedores)

## Principios de Desarrollo

### 1. Arquitectura Modular
- Organizar por dominio/feature
- Cada módulo contiene: controllers, services, entities, DTOs
- Mantener separación de responsabilidades

### 2. Estándares de Código
- **Naming**: PascalCase (clases), camelCase (métodos), kebab-case (archivos)
- **Imports**: Angular → NestJS → Proyecto → Tipos
- **Validación**: Usar class-validator en DTOs
- **Linting**: Respetar ESLint config (`eslint.config.mjs`)

### 3. Backend (NestJS)

**Estructura de Carpetas:**
```
src/
├── controllers/[domain]/
├── services/[domain]/
├── entities/[domain]/
├── auth/
└── app.module.ts
```

**Patrones:**
- Controllers: Solo manejo de HTTP, delegar lógica a servicios
- Services: Inyectable, contiene lógica de negocio, acceso a BD
- DTOs: Validación con decoradores de class-validator
- Guardias: JwtAuthGuard para rutas protegidas
- Manejo de Errores: Usar HttpException o custom exceptions
- DataSource: Dual setup (MySQL + SQL Server) en servicios

**CRUD Básico:**
```typescript
// Controlador
@Controller('domain')
export class DomainController {
  @Post()
  @UseGuards(JwtAuthGuard)
  create(@Body() dto: CreateDomainDto) { }

  @Get()
  @UseGuards(JwtAuthGuard)
  findAll() { }

  @Get(':id')
  @UseGuards(JwtAuthGuard)
  findOne(@Param('id') id: string) { }

  @Put(':id')
  @UseGuards(JwtAuthGuard)
  update(@Param('id') id: string, @Body() dto: UpdateDomainDto) { }

  @Delete(':id')
  @UseGuards(JwtAuthGuard)
  remove(@Param('id') id: string) { }
}
```

### 4. Frontend (Angular)

**Estructura de Carpetas:**
```
src/app/
├── features/[domain]/
│   ├── [domain].component.ts
│   ├── [domain].component.html
│   └── [domain].component.css
├── services/
├── app.routes.ts
└── app.config.ts
```

**Patrones:**
- Componentes: Standalone, Signal-based
- Servicios: Inyectable en root, consumir API backend
- Routing: Lazy loading para features grandes
- State: Usar Signals para reactividad local
- HttpClient: Para consumir endpoints backend

**Componente Básico:**
```typescript
@Component({
  selector: 'app-domain',
  standalone: true,
  imports: [CommonModule],
  template: `<div>{{ data() | json }}</div>`
})
export class DomainComponent {
  private service = inject(DomainService);
  data = signal([]);

  ngOnInit() {
    this.service.getAll().subscribe(res => this.data.set(res));
  }
}
```

### 5. Autenticación

**Backend:**
- JWT Token en headers: `Authorization: Bearer <token>`
- Estrategia Passport en `auth/jwt-auth/jwt.strategy.ts`
- Guard en `auth/jwt-auth/jwt-auth.guard.ts`
- Bcrypt para hash de contraseñas

**Frontend:**
- Guardar token en localStorage o sessionStorage (después del login)
- Adjuntar token a requests HTTP
- Redirigir a login si token expira

### 6. Base de Datos

**MySQL (Principal):**
- Aplicación principal
- Entidades con @Entity(), @Column(), etc.
- Raw queries: `mysqlDataSource.query()`

**SQL Server (Proveedores):**
- Consultas de proveedores
- Usar cuando sea necesario
- Mismo patrón: `sqlServerDataSource.query()`

### 7. Testing

**Backend (Jest):**
- Ubicar: `src/**/*.spec.ts` o `test/`
- Tests unitarios para servicios
- Tests e2e para endpoints
- Config: `jest-e2e.json` y `tsconfig.build.json`
- Command: `npm test`

**Frontend (Jasmine/Karma):**
- Ubicar: `src/**/*.spec.ts`
- Tests para componentes y servicios
- Config: `tsconfig.spec.json`
- Command: `npm test`

### 8. Deployment

**Backend:**
- Build: `npm run build`
- Start: `npm run start` o usar `start-backend.bat`

**Frontend:**
- Build: `npm run build`
- Start: `npm start` o usar `start-frontend.bat`

## Flujo de Desarrollo

1. **Crear Feature/Módulo**: Usa el skill `create-feature` para generar boilerplate
2. **Implementar Endpoint**: Backend controller → service → entity/DTO
3. **Crear Componente**: Frontend component → service → routing
4. **Testing**: Escribir tests unitarios mientras se desarrolla
5. **Validación**: Respetar ESLint, stándares de naming, patrones

## Herramientas y Comandos

**Backend:**
```bash
cd backend-app
npm install          # Instalar dependencias
npm run start        # Desarrollo
npm run build        # Build producción
npm test            # Ejecutar tests
npm run lint        # Verificar ESLint
```

**Frontend:**
```bash
cd frontend-app
npm install          # Instalar dependencias
npm start           # Desarrollo
npm run build       # Build producción
npm test            # Ejecutar tests
```

## Recursos

- Skills disponibles:
  - `create-feature`: Crear nuevo módulo/feature
  
- Documentación:
  - [NestJS Docs](https://docs.nestjs.com)
  - [Angular Docs](https://angular.io/docs)
  - [TypeORM Docs](https://typeorm. io)
  - [JWT Passport](https://docs.nestjs.com/security/authentication)

---

**Última actualización**: 2026-04-03

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dorianlondonolenis-independiente) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
