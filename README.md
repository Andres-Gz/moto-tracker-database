# 🗄️ Moto Tracker - Database

Scripts SQL, migraciones y configuración de base de datos para MotoTracker.

## 📋 Descripción

Repositorio dedicado a la gestión de la base de datos PostgreSQL del proyecto MotoTracker, incluyendo:
- Esquema completo de base de datos
- Migraciones versionadas con Flyway
- Scripts de inicialización
- Datos de prueba (seeds)
- Backups y restore procedures

## 🛠️ Tecnologías

- **Database:** PostgreSQL 15+
- **Migration Tool:** Flyway 9.x
- **Admin Tool:** pgAdmin 4 / DBeaver

## 📁 Estructura del Proyecto

```
database/
├── migrations/              # Migraciones Flyway
│   ├── V1__create_users_table.sql
│   ├── V2__create_vehicles_table.sql
│   ├── V3__create_maintenances_table.sql
│   ├── V4__create_expenses_table.sql
│   └── V5__create_indexes.sql
├── seeds/                   # Datos de prueba
│   ├── dev/                 # Seeds para desarrollo
│   │   └── dev_data.sql
│   └── test/                # Seeds para testing
│       └── test_data.sql
├── functions/               # Funciones almacenadas
│   └── utility_functions.sql
├── views/                   # Vistas SQL
│   └── dashboard_views.sql
├── backups/                 # Scripts de backup
│   └── backup_script.sh
├── docs/                    # Documentación
│   ├── schema_diagram.png
│   └── queries_examples.sql
└── docker/                  # Docker compose para desarrollo
    └── docker-compose.yml
```

## 🏗️ Esquema de Base de Datos

### Diagrama ER

```
┌─────────────┐
│    USERS    │
├─────────────┤
│ id (PK)     │
│ email       │
│ password    │
│ ...         │
└──────┬──────┘
       │ 1:N
       │
┌──────▼──────────┐
│    VEHICLES     │
├─────────────────┤
│ id (PK)         │
│ user_id (FK)    │
│ brand           │
│ model           │
│ ...             │
└────────┬────────┘
         │
    ┌────┴─────┐
    │ 1:N      │ 1:N
    │          │
┌───▼────────┐ │
│MAINTENANCES│ │
├────────────┤ │
│ id (PK)    │ │
│vehicle_id  │ │
│ ...        │ │
└────────────┘ │
               │
        ┌──────▼─────┐
        │  EXPENSES  │
        ├────────────┤
        │ id (PK)    │
        │vehicle_id  │
        │ ...        │
        └────────────┘
```

### Tablas Principales

1. **users** - Usuarios del sistema
2. **vehicles** - Vehículos registrados
3. **maintenances** - Mantenimientos realizados
4. **expenses** - Gastos operativos

Ver diagrama completo en: [Database Design](https://github.com/Andres-Gz/moto-tracker-docs/blob/main/docs/02-database-design.md)

## 🚀 Quick Start

### Prerrequisitos

- PostgreSQL 15+
- Java 11+ (para Flyway)
- Flyway CLI 9.x (opcional)

### Instalación con Docker

```bash
# Iniciar PostgreSQL con Docker
cd docker
docker-compose up -d

# Verificar que está corriendo
docker-compose ps
```

### Configuración Manual

```bash
# 1. Crear base de datos
createdb moto_tracker_dev

# 2. Conectarse a la base de datos
psql moto_tracker_dev

# 3. Ejecutar migraciones manualmente (o usar Flyway)
psql -d moto_tracker_dev -f migrations/V1__create_users_table.sql
psql -d moto_tracker_dev -f migrations/V2__create_vehicles_table.sql
# ... etc
```

### Usar Flyway para Migraciones

```bash
# Instalar Flyway CLI
# macOS: brew install flyway
# Windows: choco install flyway-commandline
# Linux: https://flywaydb.org/download

# Configurar flyway.conf
flyway.url=jdbc:postgresql://localhost:5432/moto_tracker_dev
flyway.user=postgres
flyway.password=postgres
flyway.locations=filesystem:./migrations

# Ejecutar migraciones
flyway migrate

# Ver estado
flyway info

# Limpiar base de datos (CUIDADO!)
flyway clean
```

## 📊 Migraciones

### Convención de Nombres

Las migraciones siguen el patrón Flyway:

```
V{VERSION}__{DESCRIPTION}.sql

Ejemplos:
V1__create_users_table.sql
V2__create_vehicles_table.sql
V3__add_vehicle_color_column.sql
```

### Crear Nueva Migración

```bash
# Crear archivo
touch migrations/V6__add_vehicle_photos.sql

# Escribir SQL
ALTER TABLE vehicles ADD COLUMN photo_url VARCHAR(500);

# Ejecutar migración
flyway migrate
```

## 🌱 Seeds (Datos de Prueba)

### Cargar datos de desarrollo

```bash
psql -d moto_tracker_dev -f seeds/dev/dev_data.sql
```

### Ejemplo de seed data

```sql
-- seeds/dev/dev_data.sql
INSERT INTO users (email, password_hash, first_name, last_name) 
VALUES ('andre@test.com', '$2a$10$...', 'Andre', 'García');

INSERT INTO vehicles (user_id, brand, model, year, license_plate, vehicle_type, current_mileage)
VALUES (1, 'KTM', '390 Duke', 2026, 'ABC123', 'MOTO', 1500);
```

## 🔍 Queries Útiles

### Estadísticas de un vehículo

```sql
-- Ver gastos totales por tipo
SELECT 
    expense_type,
    COUNT(*) as cantidad,
    SUM(amount) as total
FROM expenses
WHERE vehicle_id = 1
GROUP BY expense_type
ORDER BY total DESC;
```

### Últimos mantenimientos

```sql
-- Últimos 10 mantenimientos de todos los vehículos
SELECT 
    v.brand,
    v.model,
    m.maintenance_date,
    m.maintenance_type,
    m.cost
FROM maintenances m
JOIN vehicles v ON m.vehicle_id = v.id
ORDER BY m.maintenance_date DESC
LIMIT 10;
```

### Resumen mensual

```sql
-- Gastos del mes actual por vehículo
SELECT 
    v.id,
    v.brand || ' ' || v.model as vehicle,
    SUM(e.amount) as total_expenses
FROM vehicles v
LEFT JOIN expenses e ON v.id = e.vehicle_id
    AND DATE_TRUNC('month', e.expense_date) = DATE_TRUNC('month', CURRENT_DATE)
GROUP BY v.id, v.brand, v.model;
```

Más queries en: [docs/queries_examples.sql](docs/queries_examples.sql)

## 💾 Backup y Restore

### Crear Backup

```bash
# Backup completo
pg_dump -U postgres moto_tracker_dev > backups/backup_$(date +%Y%m%d).sql

# Backup solo schema
pg_dump -U postgres --schema-only moto_tracker_dev > backups/schema_$(date +%Y%m%d).sql

# Backup solo datos
pg_dump -U postgres --data-only moto_tracker_dev > backups/data_$(date +%Y%m%d).sql
```

### Restaurar Backup

```bash
# Restaurar desde backup
psql -U postgres moto_tracker_dev < backups/backup_20260212.sql
```

### Script Automatizado

```bash
# Ejecutar backup automático
chmod +x backups/backup_script.sh
./backups/backup_script.sh
```

## 🔐 Seguridad

### Usuarios y Permisos

```sql
-- Crear usuario de solo lectura
CREATE USER readonly_user WITH PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE moto_tracker_dev TO readonly_user;
GRANT USAGE ON SCHEMA public TO readonly_user;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_user;

-- Crear usuario de aplicación
CREATE USER app_user WITH PASSWORD 'app_password';
GRANT ALL PRIVILEGES ON DATABASE moto_tracker_dev TO app_user;
```

### Best Practices

- ✅ Usar contraseñas seguras
- ✅ Limitar permisos por usuario
- ✅ Encriptar conexiones (SSL)
- ✅ Backups regulares automatizados
- ✅ No commitear credenciales al repo

## 🐳 Docker Setup

### docker-compose.yml

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    container_name: moto_tracker_db
    environment:
      POSTGRES_DB: moto_tracker_dev
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./migrations:/docker-entrypoint-initdb.d
    restart: unless-stopped

volumes:
  postgres_data:
```

### Iniciar y Detener

```bash
# Iniciar
docker-compose up -d

# Ver logs
docker-compose logs -f postgres

# Detener
docker-compose down

# Detener y eliminar volúmenes
docker-compose down -v
```

## 📈 Performance

### Índices Importantes

Los índices ya están creados en las migraciones:

```sql
-- Índices en foreign keys
CREATE INDEX idx_vehicles_user_id ON vehicles(user_id);
CREATE INDEX idx_maintenances_vehicle_id ON maintenances(vehicle_id);
CREATE INDEX idx_expenses_vehicle_id ON expenses(vehicle_id);

-- Índices en campos de búsqueda
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_vehicles_license_plate ON vehicles(license_plate);
CREATE INDEX idx_maintenances_date ON maintenances(maintenance_date DESC);
CREATE INDEX idx_expenses_date ON expenses(expense_date DESC);
```

### Vacuum y Analyze

```sql
-- Optimizar tablas regularmente
VACUUM ANALYZE users;
VACUUM ANALYZE vehicles;
VACUUM ANALYZE maintenances;
VACUUM ANALYZE expenses;
```

## 🧪 Testing

### Crear DB de Testing

```bash
createdb moto_tracker_test
psql -d moto_tracker_test -f migrations/*.sql
psql -d moto_tracker_test -f seeds/test/test_data.sql
```

## 📚 Documentación Adicional

- [Documentación completa del proyecto](https://github.com/Andres-Gz/moto-tracker-docs)
- [Diseño detallado de BD](https://github.com/Andres-Gz/moto-tracker-docs/blob/main/docs/02-database-design.md)
- [Backend API](https://github.com/Andres-Gz/moto-tracker-backend)

## 🤝 Contribución

1. Fork el proyecto
2. Crear feature branch (`git checkout -b feature/nueva-migracion`)
3. Crear nueva migración siguiendo convenciones
4. Commit cambios (`git commit -m 'feat: add new migration'`)
5. Push al branch (`git push origin feature/nueva-migracion`)
6. Crear Pull Request

### Checklist para nuevas migraciones

- [ ] Versión correcta (V{N}__)
- [ ] Nombre descriptivo
- [ ] SQL válido y testeado
- [ ] Rollback considerado
- [ ] Índices agregados si son necesarios
- [ ] Documentación actualizada

## 📄 Licencia

MIT License - ver [LICENSE](LICENSE) para más detalles

## 👨‍💻 Autor

**Andre García**
- GitHub: [@Andres-Gz](https://github.com/Andres-Gz)

---

⭐ Si te gusta el proyecto, dale una estrella en GitHub!
