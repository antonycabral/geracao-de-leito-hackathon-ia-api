# 🏥 Documento de Refinamento - Sistema de Gestão de Leitos Hospitalares
## Versão 2.0 - Atualizado com Implementações Reais

## 1. Visão Geral do Sistema (The BluePrint)

O sistema opera como um **Gêmeo Digital Hospitalar**, mantendo uma máquina de estados sincronizada de **Localizações** e **Pacientes**. O objetivo é rastrear em tempo real onde cada paciente está e qual o próximo passo da sua jornada de cuidados.

### Tecnologias Implementadas:
- **Backend**: NestJS + TypeORM + SQLite
- **Frontend**: Angular 17 (Standalone Components)
- **Autenticação**: JWT com refresh tokens
- **Padrão**: FHIR-Compliant (Fast Healthcare Interoperability Resources)
- **Comunicação Real-time**: WebSockets

---

## 2. Estrutura Hierárquica (Geolocalização Interna) ✅ IMPLEMENTADO

O sistema respeita a árvore hierárquica completa de dependências hospitalares:

```
Unidade: Hospital Central
  └─ Bloco: Bloco A (Internação), Bloco B (Emergência/Triagem)
      └─ Andar: 1º ao 5º Andar
          └─ Ala/Setor: Ala Sul (Cardiologia), Ala Norte (Infectologia)
              └─ Leito: Identificador único (Ex: 302-A)
```

**Entidade Location implementada com:**
- `type`: UNIDADE | BLOCO | ANDAR | ALA | LEITO
- `parentId`: Referência hierárquica
- `alias`: Identificador único (Ex: "302-A")
- `specialty`: Especialidade médica
- `metadata`: Equipamentos e observações adicionais

---

## 3. Matriz de Estados Implementados

### 3.1 Status de Leitos (LocationStatus) ✅
| Status | Descrição | Condição de Uso |
|--------|-----------|-----------------|
| `DISPONIVEL` | Leito livre e limpo | Pronto para nova internação |
| `OCUPADO` | Paciente alocado | Paciente presente no leito |
| `OCUPADO_AUSENTE` | Paciente em exame/procedimento | Leito reservado, paciente volta |
| `HIGIENIZACAO_NECESSARIA` | Aguardando limpeza | Após alta ou incidente |
| `HIGIENIZACAO_EM_ANDAMENTO` | Limpeza ativa | Equipe trabalhando |
| `MANUTENCAO` | Manutenção de equipamentos | Bloqueado temporariamente |
| `BLOQUEADO` | Administrativamente bloqueado | Não alocável |

### 3.2 Status de Atendimentos (EncounterStatus) ✅
| Status | Gatilho | Impacto |
|--------|---------|---------|
| `EM_TRIAGEM` | Paciente chegou | Início do processo |
| `AGUARDANDO_LEITO` | Sem leito disponível | Fila de espera (NIR) |
| `EM_ATENDIMENTO` | Leito alocado | Internação ativa |
| `EM_MEDICACAO` | Medicação em curso | Bloqueio de transporte |
| `EM_EXAME` | Paciente em exame | Leito OCUPADO_AUSENTE |
| `AGUARDANDO_VISITA` | Aguardando médico | Prioridade no dashboard |
| `PREVISAO_ALTA` | EDD definida | Preparação logística |
| `ALTA_CONFIRMADA` | Médico aprovou | Gatilho de limpeza |
| `ALTA_REALIZADA` | Paciente saiu | Liberar leito |
| `TRANSFERIDO` | Mudança de leito | Histórico mantido |
| `CANCELADO` | Internação cancelada | Registro de auditoria |

---

## 4. Requisitos Funcionais Implementados (RF)

### RF.01 - Triagem e Admissão ✅ IMPLEMENTADO
**Endpoint**: `POST /triagem/realizar`

O sistema implementa triagem automatizada com:
- Captura de dados Manchester (cor de risco, sinais vitais, queixa principal)
- **Algoritmo de Match Automático** de leito baseado em:
  - Classificação de risco (Vermelho > Laranja > Amarelo > Verde > Azul)
  - Especialidade necessária
  - Disponibilidade imediata
- Se não há leito: **Fila de Espera Digital (NIR)** ordenada por prioridade Manchester
- Registro automático de paciente + encounter + alocação/fila

**Lógica Implementada:**
```typescript
// Prioridades por cor Manchester
VERMELHO → UTI, Emergência, Cardiologia
LARANJA → Emergência, Cardiologia
AMARELO → Emergência, Clínica Médica
VERDE → Clínica Médica, Pediatria
AZUL → Clínica Médica
```

### RF.02 - Cronograma de Cuidados (Timeline) ⚠️ PARCIALMENTE IMPLEMENTADO
**Estruturas criadas**:
- `MedicationAdministration`: Registro de medicamentos
- `ServiceRequest`: Solicitação de exames
- `Task`: Tarefas de medicação, exames, visitas médicas

**Falta implementar**:
- Interface completa de timeline visual
- Dashboard consolidado por paciente

### RF.03 - Gestão de Prorrogação de Alta ✅ IMPLEMENTADO
**Endpoint**: `PATCH /encounters/:id/estimated-discharge-date`

Funcionalidades:
- Campo `estimatedDischargeDate` (EDD)
- Obrigatoriedade de justificativa para alteração
- Registro em `AuditLog` com:
  - ID do médico
  - Data anterior e nova
  - Justificativa
  - Timestamp imutável

### RF.04 - Portal do Acompanhante ⚠️ EM DESENVOLVIMENTO
**Role**: `UserRole.ACOMPANHANTE`

Estrutura criada:
- Autenticação específica para acompanhantes
- Acesso limitado a `GET /encounters/:id`
- Filtro de campos sensíveis (apenas status, timeline básica)

**Falta implementar**:
- Interface frontend específica
- Sistema de convite/código de acesso
- Notificações push

### RF.05 - Notificação de Limpeza de Emergência ✅ IMPLEMENTADO
**Endpoint**: `POST /tasks` com `type: LIMPEZA_EMERGENCIA`

Funcionalidades:
- Campo `priority` com valor `CRITICA` ou `URGENTE`
- Status: SOLICITADA → ACEITA → EM_ANDAMENTO → CONCLUIDA
- Timestamps: `acceptedAt`, `startedAt`, `completedAt`
- SLA tracking automático

---

## 5. Regras de Negócio Críticas (RN)

### RN.01 - Conflito de Agenda ⚠️ ESTRUTURA PRONTA
**Status**: Entidades criadas, validação falta implementar

Estrutura:
- `MedicationAdministration.effectiveDateTime`
- `ServiceRequest.occurrenceDateTime`

**Falta**: Middleware de validação cruzada

### RN.02 - Trava de Limpeza ✅ IMPLEMENTADO
**Lógica no LocationsService.updateStatus()**

Validação:
```typescript
if (newStatus === 'DISPONIVEL') {
  const limpezaTask = await verificarChecklistCompleto(locationId);
  if (!limpezaTask || !limpezaTask.checklist.every(item => item.completed)) {
    throw new BadRequestException('Checklist de limpeza incompleto');
  }
}
```

### RN.03 - Visibilidade do Acompanhante ⚠️ ESTRUTURA PRONTA
**Falta**: Cron job para revogação automática

Estrutura:
- `User.role = ACOMPANHANTE`
- `Encounter.actualDischargeDate`

**Implementar**: Worker que desativa acompanhante após 2h da alta

### RN.04 - Prioridade de Higienização ✅ IMPLEMENTADO
**Lógica no TasksService**

Ordenação de fila:
```typescript
task.priority === 'CRITICA' && location.specialty in ['UTI', 'Emergência']
  → Prioridade máxima (fura fila)
```

### RN.05 - Registro de Notas ✅ IMPLEMENTADO
**Entidade**: `AuditLog`

Campos registrados:
- `entityType: 'Encounter'`
- `action: 'UPDATE_EDD'`
- `userId`: ID do médico
- `oldValues`: { estimatedDischargeDate: oldDate }
- `newValues`: { estimatedDischargeDate: newDate }
- `justification`: Justificativa obrigatória
- `timestamp`: Data/hora da mudança

---## 6. Modelagem de Dados (Entidades Implementadas)

### 6.1 Entidades Principais ✅

#### **Patient** - Paciente
```typescript
{
  id: uuid,
  name: string,
  documentNumber: string (CPF único),
  birthDate: date,
  phone?: string,
  emergencyContact?: string,
  riskColor: 'vermelho' | 'laranja' | 'amarelo' | 'verde' | 'azul',
  chiefComplaint?: string,
  vitalSigns?: {
    bloodPressure, heartRate, temperature, 
    oxygenSaturation, respiratoryRate
  },
  allergies?: string[],
  active: boolean
}
```

#### **Location** - Leito/Localização
```typescript
{
  id: uuid,
  alias: string (único, ex: "302-A"),
  name: string,
  status: LocationStatus,
  type: 'leito' | 'ala' | 'andar' | 'bloco' | 'unidade',
  specialty?: string,
  floor?: string,
  building?: string,
  parentId?: uuid (hierarquia),
  capacity?: number,
  metadata?: Record<string, any>
}
```

#### **Encounter** - Internação/Atendimento
```typescript
{
  id: uuid,
  patientId: uuid,
  locationId?: uuid,
  responsibleDoctorId?: uuid,
  status: EncounterStatus,
  startDateTime: datetime,
  estimatedDischargeDate?: datetime (EDD),
  actualDischargeDate?: datetime,
  dischargeReason?: string,
  diagnosis?: string,
  treatmentPlan?: string,
  notes?: Array<{
    timestamp: datetime,
    authorId: uuid,
    authorName: string,
    content: string
  }>
}
```

#### **Task** - Tarefas (Limpeza, Medicação, Exames)
```typescript
{
  id: uuid,
  type: 'limpeza' | 'limpeza_emergencia' | 'manutencao' | 
        'transporte' | 'medicacao' | 'exame' | 'visita_medica',
  priority: 'baixa' | 'normal' | 'alta' | 'urgente' | 'critica',
  status: 'solicitada' | 'aceita' | 'em_andamento' | 'concluida' | 'cancelada',
  encounterId?: uuid,
  locationId?: uuid,
  assignedToId?: uuid (profissional responsável),
  requestedById?: uuid,
  description?: string,
  scheduledFor?: datetime,
  acceptedAt?: datetime,
  startedAt?: datetime,
  completedAt?: datetime,
  checklist?: Array<{
    item: string,
    completed: boolean,
    timestamp?: datetime
  }>,
  metadata?: Record<string, any>
}
```

#### **MedicationAdministration** - Medicamentos
```typescript
{
  id: uuid,
  encounterId: uuid,
  patientId: uuid,
  medicationName: string,
  dosage: string,
  route: string (via de administração),
  frequency: string,
  status: 'planejado' | 'em_andamento' | 'concluido' | 'cancelado',
  scheduledDateTime: datetime,
  effectiveDateTime?: datetime (quando foi dado),
  administeredById?: uuid,
  notes?: string
}
```

#### **ServiceRequest** - Solicitação de Exames
```typescript
{
  id: uuid,
  encounterId: uuid,
  patientId: uuid,
  requestType: string (tipo de exame/procedimento),
  priority: 'rotina' | 'urgente' | 'emergencia',
  status: 'solicitado' | 'aprovado' | 'agendado' | 'em_andamento' | 
          'concluido' | 'cancelado',
  requestedById: uuid (médico solicitante),
  occurrenceDateTime?: datetime,
  notes?: string,
  result?: string
}
```

#### **User** - Usuários do Sistema
```typescript
{
  id: uuid,
  email: string (único),
  password: string (hash bcrypt),
  name: string,
  role: 'ADMIN' | 'MEDICO' | 'ENFERMEIRO' | 'ENFERMAGEM' | 
        'TRIAGEM' | 'LIMPEZA' | 'ACOMPANHANTE',
  active: boolean,
  cpf?: string,
  specialty?: string,
  crm?: string,
  coren?: string
}
```

#### **AuditLog** - Auditoria (RN.05)
```typescript
{
  id: uuid,
  entityType: string ('Encounter', 'Location', 'Task', etc),
  entityId: uuid,
  action: string ('CREATE', 'UPDATE', 'DELETE', 'STATUS_CHANGE', etc),
  userId?: uuid,
  oldValues?: Record<string, any>,
  newValues?: Record<string, any>,
  justification?: string,
  ipAddress?: string,
  userAgent?: string,
  timestamp: datetime
}
```

---

## 7. Endpoints da API Implementados

### 7.1 Autenticação
- `POST /auth/register` - Cadastro de usuário
- `POST /auth/login` - Login (retorna JWT)
- `GET /auth/profile` - Perfil do usuário autenticado

### 7.2 Triagem ✅ ESPECIALIZADO
- `POST /triagem/realizar` - Triagem completa com auto-alocação
- `GET /triagem/fila-espera` - Fila de espera (NIR) ordenada
- `PUT /triagem/alocar/:encounterId/:locationId` - Alocação manual

### 7.3 Pacientes
- `POST /patients` - Criar paciente
- `GET /patients` - Listar todos
- `GET /patients/search?documentNumber=` - Buscar por CPF
- `GET /patients/:id` - Detalhes do paciente
- `PUT /patients/:id` - Atualizar
- `DELETE /patients/:id` - Desativar

### 7.4 Leitos (Locations)
- `POST /locations` - Criar leito/local
- `GET /locations` - Listar todos
- `GET /locations/available-beds` - Leitos disponíveis
- `GET /locations/:id` - Detalhes
- `GET /locations/:id/hierarchy` - Caminho hierárquico
- `PUT /locations/:id` - Atualizar
- `PATCH /locations/:id/status` - Atualizar status
- `DELETE /locations/:id` - Excluir

### 7.5 Internações (Encounters)
- `POST /encounters` - Criar internação
- `GET /encounters` - Listar todas
- `GET /encounters/active` - Internações ativas
- `GET /encounters/:id` - Detalhes
- `GET /encounters/patient/:patientId` - Histórico do paciente
- `PUT /encounters/:id` - Atualizar
- `PATCH /encounters/:id/status` - Atualizar status
- `PATCH /encounters/:id/estimated-discharge-date` - Definir/alterar EDD
- `POST /encounters/:id/notes` - Adicionar nota
- `POST /encounters/:id/discharge` - Realizar alta

### 7.6 Tarefas (Tasks)
- `POST /tasks` - Criar tarefa
- `GET /tasks` - Listar todas
- `GET /tasks/pending` - Tarefas pendentes
- `GET /tasks/cleaning` - Tarefas de limpeza
- `GET /tasks/:id` - Detalhes
- `PUT /tasks/:id` - Atualizar
- `PATCH /tasks/:id/status` - Atualizar status
- `PATCH /tasks/:id/assign` - Atribuir a profissional
- `POST /tasks/:id/complete` - Marcar como concluída

### 7.7 Medicações
- `POST /medications` - Registrar medicação
- `GET /medications` - Listar todas
- `GET /medications/encounter/:encounterId` - Por internação
- `GET /medications/pending` - Pendentes de administração
- `PATCH /medications/:id/administer` - Registrar administração

### 7.8 Exames (Service Requests)
- `POST /service-requests` - Solicitar exame
- `GET /service-requests` - Listar todos
- `GET /service-requests/encounter/:encounterId` - Por internação
- `PATCH /service-requests/:id` - Atualizar
- `PATCH /service-requests/:id/result` - Registrar resultado

---

## 8. Requisitos Não Funcionais (RNF)

### RNF.01 - Sincronização ⚠️ ESTRUTURA PRONTA
**Status**: WebSocket service criado no frontend, falta implementar no backend

**Implementado**:
- `WebSocketService` no Angular
- Infraestrutura de conexão

**Falta**:
- Gateway WebSocket no NestJS
- Eventos de broadcast para mudanças críticas

### RNF.02 - Usabilidade 📱 PLANEJADO
**Status**: Design System Luminous criado

**Implementado**:
- Paleta de cores (#FFB800 → #FF5800)
- Componentes reutilizáveis

**Falta**:
- Interface mobile-first
- Botões grandes (mínimo 44x44px)
- Teste com luvas

### RNF.03 - Privacidade (LGPD) ✅ ESTRUTURA IMPLEMENTADA
**Controles**:
- Role-based access control (RBAC)
- Guards por endpoint
- Filtro de campos sensíveis para acompanhantes
- AuditLog de todas as ações
- Hash de senhas com bcrypt

**Falta**:
- Termo de consentimento
- Anonimização de dados antigos
- Exportação de dados (direito do titular)

---

## 9. Próximos Passos (Roadmap)

### 🔴 Prioridade Alta
1. **WebSockets real-time** no backend (NestJS Gateway)
2. **Interface de Timeline** completa por paciente
3. **Cron job** para revogação de acompanhantes (RN.03)
4. **Validação de conflito de agenda** (RN.01) - medicação vs exame
5. **Dashboard Analytics** com métricas-chave:
   - Taxa de ocupação por ala
   - Tempo médio de giro de leito
   - SLA de limpeza
   - Fila de espera (tempo médio)

### 🟡 Prioridade Média
6. **Portal do Acompanhante** (interface)
7. **Sistema de Notificações Push** (PWA)
8. **Relatórios gerenciais** (PDF/Excel)
9. **Integração com PEP** (Prontuário Eletrônico)
10. **API RESTful documentada** com Swagger completo

### 🟢 Prioridade Baixa
11. **Modo offline** (PWA com sync)
12. **Geolocalização indoor** (BLE beacons)
13. **IA preditiva** para previsão de demanda
14. **Integração com NIR nacional**
15. **App mobile nativo** (React Native)

---

## 10. Glossário

- **EDD**: Estimated Discharge Date (Previsão de Alta)
- **NIR**: Necessidade Imediata de Recursos (Fila de Espera)
- **Manchester**: Protocolo de Classificação de Risco
- **FHIR**: Fast Healthcare Interoperability Resources
- **SLA**: Service Level Agreement (Acordo de Nível de Serviço)
- **RBAC**: Role-Based Access Control
- **PEP**: Prontuário Eletrônico do Paciente
- **LGPD**: Lei Geral de Proteção de Dados

---

**Atualizado em**: 18 de março de 2026  
**Versão**: 2.0 - Reflete implementações reais do sistema