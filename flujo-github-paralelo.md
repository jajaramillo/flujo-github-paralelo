# 🚀 Estrategia de Branching Git para MELI

**Gestión de desarrollo paralelo con integración continua y despliegues controlados**

---

## 📊 Diagrama de Flujo Completo

### 🎨 Leyenda de Colores

| Color | Significado |
|-------|-------------|
| 🔴 Rojo | Ramas principales (master) |
| 🟣 Morado | Ramas feature (feature/inic*, feature/*ToConsolidate) |
| 🔵 Azul claro | Ramas release/consolidate (feature/release-MMDD, feature/consolidate) |
| 🟠 Naranja | Flujo DEV - Despliegue a MELI DEV/UAT |
| 🟢 Verde | Flujo PRD - Despliegue a MELI PRD |
| ⚪ Gris | Decisiones y puntos de control |

### 📋 Código Mermaid del Diagrama

Para visualizar el diagrama, copia este código y pégalo en: https://mermaid.live

```mermaid
flowchart TD
    Start([Inicio:<br/>Código en Producción]) --> Master[Rama MASTER<br/>Código actual en PRD]
    
    
    Master --> CreateBase[Crear feature/consolidate<br/>desde master]
    Master --> CreateDev1[Crear feature/inic1111<br/>desde master]    
    Master --> CreateDevN[Crear feature/inicXXXX<br/>desde master]
       
    CreateBase --> Base[feature/consolidate]
    CreateDev1 --> Dev1[feature/inic1111<br/>Desarrollo en paralelo]    
    CreateDevN --> DevN[feature/inicXXXX<br/>Desarrollo en paralelo<br/>Mismo Flujo feature/inic1111]
    
    %% Flujo para MELI DEV
    Dev1 --> DecisionDev1{¿Listo para<br/>MELI DEV?}
    DecisionDev1 -->|Sí| CreateTrans1[Crear feature/inic1111ToConsolidate<br/>desde feature/inic1111]
    CreateTrans1 --> Trans1[feature/inic1111ToConsolidate<br/>Rama de transición]
    Trans1 --> MergeToConsolidate1[Merge feature/inic1111ToConsolidate<br/>→ feature/consolidate]
    
          
    Base --> BaseLista
    MergeToConsolidate1 --> BaseLista{feature/consolidate<br/>actualizada}
    MergeToConsolidate1 --> DeleteTrans1[Borrar<br/>feature/inic1111ToConsolidate]
    
    BaseLista --> PRDevExists{¿PR feature/consolidate → develop<br/>ya existe?}
    PRDevExists -->|No| CreatePRDev[Crear PR<br/>feature/consolidate → develop]
    PRDevExists -->|Sí| ApproveDev
    CreatePRDev --> ApproveDev[Solicitar aprobación<br/>de PR]
    ApproveDev --> DeployDev[Despliegue automático<br/>MELI DEV]
    DeployDev --> RequestIDDev[CI/CD genera<br/>Request ID]
    
    %% Flujo para MELI UAT
    RequestIDDev --> DecisionUAT{¿Llevar a<br/>MELI UAT?}
    DecisionUAT -->|Sí| TransportUAT[Transportar Request ID<br/>por medio de SOLMAN]
    TransportUAT --> DeployUAT[Despliegue<br/>MELI UAT]
    
    %% Flujo para MELI PRD
    Dev1 --> DecisionPRD1{¿Listo para<br/>MELI PRD?}
    DevN --> DecisionPRDN{¿Listo para<br/>MELI PRD?}
    
    DecisionPRD1 -->|Sí| CheckMultiple{¿Múltiples<br/>iniciativas<br/>a PRD?}
    DecisionPRDN -->|Sí| CheckMultiple
    
    CheckMultiple -->|No - Una sola| PRProdSingle[Crear PR<br/>feature/inic1111 → develop]
    CheckMultiple -->|Sí - Varias| CreateRelease[Crear feature/release-MMDD<br/>desde master]
    
    CreateRelease --> Release[feature/release-MMDD]
    Release --> MergeInit1[Merge feature/inic1111<br/>→ feature/release-MMDD]
    Release --> MergeInitN[Merge feature/inicXXXX<br/>→ feature/release-MMDD]
    
    MergeInit1 --> ReleaseReady
    MergeInitN --> ReleaseReady{feature/release-MMDD<br/>consolidada}
    
    ReleaseReady --> PRProdMulti[Crear PR<br/>feature/release-MMDD → develop]
    
    PRProdSingle --> ApproveProd
    PRProdMulti --> ApproveProd[Solicitar aprobación<br/>de PR]
    
    ApproveProd --> DeployDevProd[Despliegue<br/>MELI DEV]
    DeployDevProd --> RequestIDProd[CI/CD genera<br/>Request ID]
    
    RequestIDProd --> TransportProd[Transportar Request ID<br/>por SOLMAN a UAT]
    TransportProd --> UATCheck[Pasa por MELI UAT<br/>Validación aislada]
    UATCheck --> TransportPRD[Transporte a<br/>MELI PRD]
    
    %% Actualización post-producción    
    TransportPRD --> ConfirmPRD{¿Despliegue<br/>confirmado<br/>en PRD?}
    ConfirmPRD -->|Sí| MergeProd[Ejecutar MERGE de PR<br/>a develop]
    MergeProd --> CheckRelease{¿Es una<br/>release?}
    CheckRelease -->|Sí| UpdateMasterRelease[Actualizar master<br/>desde feature/release-MMDD]
    CheckRelease -->|No| UpdateMasterFeature[Actualizar master<br/>desde feature/inic1111]
    UpdateMasterRelease --> End([Proceso Completado<br/>PRD Actualizado])
    UpdateMasterFeature --> End
    
    %% Estilos
    classDef mainBranchStyle fill:#ff6b6b,stroke:#c92a2a,stroke-width:3px,color:#fff
    classDef featureBranchStyle fill:#a78bfa,stroke:#7c3aed,stroke-width:2px,color:#fff
    classDef releaseBranchStyle fill:#4dabf7,stroke:#1971c2,stroke-width:2px,color:#fff
    classDef devPathStyle fill:#ff922b,stroke:#e8590c,stroke-width:2px,color:#fff
    classDef prdPathStyle fill:#51cf66,stroke:#2f9e44,stroke-width:2px,color:#fff
    classDef decisionStyle fill:#f8f9fa,stroke:#495057,stroke-width:2px,color:#000
    
    %% Ramas principales: master
    class Master mainBranchStyle
    
    %% Ramas de trabajo: feature/*
    class Base,Dev1,DevN,Trans1,CreateTrans1 featureBranchStyle
    
    %% Rama release
    class Release,CreateRelease,MergeInit1,MergeInitN,ReleaseReady releaseBranchStyle
    
    %% Camino DEV (naranja)
    class PRDevExists,CreatePRDev,ApproveDev,DeployDev,RequestIDDev,TransportUAT,DeployUAT devPathStyle
    
    %% Camino PRD (verde)
    class CheckMultiple,PRProdSingle,PRProdMulti,ApproveProd,DeployDevProd,RequestIDProd,TransportProd,UATCheck,TransportPRD,ConfirmPRD,MergeProd,CheckRelease,UpdateMasterRelease,UpdateMasterFeature prdPathStyle
    
    %% Decisiones e informativos
    class DecisionDev1,DecisionPRD1,DecisionPRDN,DecisionUAT,BaseLista decisionStyle
```

---

## 🌳 Estructura de Ramas

| Rama | Propósito | Ciclo de Vida |
|------|-----------|---------------|
| `master` | Refleja el código en producción (MELI PRD) | Permanente |
| `develop` | Rama de integración para despliegues automáticos | Permanente |
| `feature/inicXXXX` | Desarrollo de iniciativas individuales en paralelo | Temporal - Se mantiene hasta después del despliegue a PRD |
| `feature/consolidate` | Rama de consolidación para múltiples iniciativas que van a DEV | Semi-permanente - Se recrea periódicamente |
| `feature/inicXXXXToConsolidate` | Rama de transición para resolver conflictos antes del merge | Temporal - Se borra después del merge |
| `feature/release-MMDD` | Consolida múltiples iniciativas listas para PRD | Temporal - Se usa solo cuando hay múltiples iniciativas |

---

## 🔄 Flujo de Trabajo Detallado

### Fase 1: Desarrollo de Iniciativa 🟠 DEV

**1. Crear rama de iniciativa desde master**
```bash
git checkout master
git pull origin master
git checkout -b feature/inic1111
```

**2. Desarrollar la funcionalidad**

Commits frecuentes en `feature/inic1111`

**3. Cuando esté listo para MELI DEV, crear rama de transición**
```bash
git checkout feature/inic1111
git checkout -b feature/inic1111ToConsolidate
```

**4. Resolver conflictos en la rama de transición**
```bash
git fetch origin
git merge origin/feature/consolidate
# Resolver conflictos si existen
git add .
git commit -m "Resolve conflicts with feature/consolidate"
```

**5. Hacer merge a feature/consolidate**
```bash
git checkout feature/consolidate
git merge feature/inic1111ToConsolidate
git push origin feature/consolidate
```

**6. Borrar rama de transición**
```bash
git branch -d feature/inic1111ToConsolidate
git push origin --delete feature/inic1111ToConsolidate
```

> 💡 **Nota sobre el PR a develop:** Si ya existe un PR abierto de `feature/consolidate → develop`, simplemente actualiza la rama. Si no existe, créalo y solicita aprobación.

---

### Fase 2: Despliegue a MELI DEV 🟠 DEV

**1. Aprobar y hacer merge del PR feature/consolidate → develop**

El merge dispara el despliegue automático a MELI DEV

**2. CI/CD genera Request ID automáticamente**

Este ID se usa para transportar el código a otros ambientes

**3. Validación en MELI DEV**

Probar la funcionalidad en el ambiente de desarrollo

---

### Fase 3: Transporte a MELI UAT (Opcional) 🟡 UAT

⚠️ **Importante:** El transporte a UAT es opcional y se usa cuando se necesita validación adicional antes de producción.

**1. Transportar Request ID a través de SOLMAN**

Usar la herramienta SOLMAN para mover el Request ID de DEV a UAT

**2. Validación en MELI UAT**

Realizar pruebas de aceptación de usuario

---

### Fase 4: Preparación para MELI PRD 🟢 PRD

🔀 **Decisión Importante:** ¿Una iniciativa o múltiples iniciativas van a PRD juntas?

#### Escenario A: Una sola iniciativa a PRD

**1. Crear PR directamente desde la feature**
```bash
Crear PR: feature/inic1111 → develop
```

**2. Aprobar el PR (NO hacer merge todavía)**

#### Escenario B: Múltiples iniciativas a PRD

**1. Crear rama de release con fecha**
```bash
git checkout master
git pull origin master
git checkout -b feature/release-1201
# Formato: MMDD (mes y día)
```

**2. Consolidar todas las iniciativas en la release**
```bash
git merge feature/inic1111
git merge feature/inic2222
git merge feature/inic3333
# Resolver conflictos si existen
git push origin feature/release-1201
```

**3. Crear PR desde la release**
```bash
Crear PR: feature/release-1201 → develop
```

**4. Aprobar el PR (NO hacer merge todavía)**

---

### Fase 5: Despliegue a MELI PRD 🟢 PRD

✅ **Flujo de Producción:** El PR aprobado pasa por DEV → UAT → PRD usando SOLMAN.

**1. El PR aprobado dispara despliegue a MELI DEV**

Genera un nuevo Request ID

**2. Transportar Request ID a MELI UAT vía SOLMAN**

Validación aislada: solo el contenido del PR, sin otras iniciativas de feature/consolidate

**3. Transportar Request ID a MELI PRD vía SOLMAN**

Despliegue final a producción

**4. Confirmar despliegue exitoso en PRD**

Validar que todo funciona correctamente

**5. Ejecutar el MERGE del PR a develop**

Ahora sí se hace el merge que estaba aprobado

**6. Actualizar master**
```bash
# Si fue una release:
git checkout master
git merge feature/release-1201
git push origin master

# Si fue una iniciativa individual:
git checkout master
git merge feature/inic1111
git push origin master
```

---

## ⚠️ Reglas y Mejores Prácticas

### 🚫 Prohibiciones Importantes

- **NUNCA** hacer merge directo de `feature/inicXXXX` a `feature/consolidate` sin usar rama de transición
- **NUNCA** hacer merge del PR a develop antes de confirmar el despliegue en PRD
- **NUNCA** mezclar iniciativas que van a DEV con iniciativas que van directo a PRD en el mismo PR
- **NUNCA** crear una release si solo una iniciativa va a PRD

### ✅ Mejores Prácticas

- **Usar ramas de transición** (`feature/inicXXXXToConsolidate`) para resolver conflictos de manera aislada
- **Borrar ramas de transición** inmediatamente después del merge exitoso
- **Mantener sincronizada** la rama `feature/consolidate` con develop periódicamente
- **Nombrar releases con fecha** en formato MMDD para facilitar el seguimiento
- **Validar en UAT** antes de PRD cuando sea crítico
- **Documentar Request IDs** generados por CI/CD para trazabilidad
- **Actualizar master** inmediatamente después de confirmar despliegue en PRD

### 📋 Checklist de Despliegue a PRD

- [ ] PR creado y aprobado (NO mergeado)
- [ ] Código desplegado automáticamente a MELI DEV
- [ ] Request ID generado por CI/CD
- [ ] Request ID transportado a MELI UAT vía SOLMAN
- [ ] Validación exitosa en MELI UAT
- [ ] Request ID transportado a MELI PRD vía SOLMAN
- [ ] Validación exitosa en MELI PRD
- [ ] Merge del PR a develop ejecutado
- [ ] Master actualizado con el contenido de la release o feature
- [ ] Ramas temporales borradas

---

## 🔧 Resolución de Conflictos

### Conflictos al integrar a feature/consolidate

```bash
# En la rama de transición feature/inic1111ToConsolidate
git fetch origin
git merge origin/feature/consolidate

# Si hay conflictos:
# 1. Revisar archivos con conflictos
git status

# 2. Editar archivos y resolver conflictos manualmente
# 3. Agregar archivos resueltos
git add .

# 4. Completar el merge
git commit -m "Resolve conflicts with feature/consolidate"

# 5. Hacer merge limpio a feature/consolidate
git checkout feature/consolidate
git merge feature/inic1111ToConsolidate
git push origin feature/consolidate
```

### Conflictos en feature/release

```bash
# Al consolidar múltiples iniciativas en la release
git checkout feature/release-1201
git merge feature/inic1111
# Resolver conflictos
git add .
git commit -m "Merge feature/inic1111 into release"

git merge feature/inic2222
# Resolver conflictos
git add .
git commit -m "Merge feature/inic2222 into release"

# Si el PR tiene nota "Si hay conflictos: prevalece feature/release"
# En caso de conflictos al hacer PR a develop, siempre mantener
# los cambios de feature/release sobre develop
```

---

## 📝 Convenciones de Nomenclatura

| Tipo | Formato | Ejemplo |
|------|---------|---------|
| Iniciativa | `feature/inicXXXX` | `feature/inic1111` |
| Transición | `feature/inicXXXXToConsolidate` | `feature/inic1111ToConsolidate` |
| Consolidación | `feature/consolidate` | `feature/consolidate` |
| Release | `feature/release-MMDD` | `feature/release-1201` |
| Request ID | Generado por CI/CD | REQ-2024-001234 |

---

## 🎯 Diagrama de Ambientes

### Flujo de Ambientes

**🟠 MELI DEV → 🟡 MELI UAT → 🟢 MELI PRD**

- **MELI DEV:** Despliegue automático al hacer merge a develop. Ambiente para desarrollo y pruebas iniciales.
- **MELI UAT:** Transporte manual vía SOLMAN. Ambiente para pruebas de aceptación de usuario. Puede contener múltiples iniciativas de feature/consolidate o validación aislada de releases.
- **MELI PRD:** Transporte manual vía SOLMAN. Ambiente de producción. Solo se despliega después de validación exitosa en UAT.

---

**Estrategia de Branching Git - MercadoLibre**  
Versión 2.0 - Diciembre 2024
