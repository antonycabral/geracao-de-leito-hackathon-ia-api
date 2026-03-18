# 🔌 API Endpoints - Sistema de Gestão de Leitos
## Versão 2.0 - Documentação Completa

**Base URL**: `http://localhost:3000/api/v1`  
**Autenticação**: Bearer Token (JWT)  
**Content-Type**: `application/json`

---

## 📋 Índice
1. [Autenticação](#autenticação)
2. [Triagem (Especializado)](#triagem-especializado)
3. [Pacientes](#pacientes)
4. [Leitos (Locations)](#leitos-locations)
5. [Internações (Encounters)](#internações-encounters)
6. [Tarefas (Tasks)](#tarefas-tasks)
7. [Medicações (Medications)](#medicações-medications)
8. [Exames (Service Requests)](#exames-service-requests)
9. [Usuários (Users)](#usuários-users)
10. [Códigos de Status HTTP](#códigos-de-status-http)

---

## 🔐 Autenticação

### POST `/auth/register`
**Descrição**: Cadastro de novo usuário no sistema  
**Acesso**: Público (apenas para primeiro admin, depois admin-only)

**Request Body**:
```json
{
  "email": "medico@hospital.com",
  "password": "senha123",
  "name": "Dr. João Silva",
  "role": "MEDICO", // ADMIN | MEDICO | ENFERMEIRO | ENFERMAGEM | TRIAGEM | LIMPEZA | ACOMPANHANTE
  "cpf": "12345678900",
  "specialty": "Cardiologia", // Opcional, para médicos
  "crm": "CRM-SP 123456" // Opcional, para médicos
}
```

**Response (201)**:
```json
{
  "id": "uuid",
  "email": "medico@hospital.com",
  "name": "Dr. João Silva",
  "role": "MEDICO",
  "active": true
}
```

---

### POST `/auth/login`
**Descrição**: Autenticação de usuário  
**Acesso**: Público

**Request Body**:
```json
{
  "email": "medico@hospital.com",
  "password": "senha123"
}
```

**Response (200)**:
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "uuid",
    "email": "medico@hospital.com",
    "name": "Dr. João Silva",
    "role": "MEDICO"
  }
}
```

---

### GET `/auth/profile`
**Descrição**: Obter perfil do usuário autenticado  
**Acesso**: Autenticado  
**Headers**: `Authorization: Bearer {token}`

**Response (200)**:
```json
{
  "id": "uuid",
  "email": "medico@hospital.com",
  "name": "Dr. João Silva",
  "role": "MEDICO",
  "specialty": "Cardiologia",
  "crm": "CRM-SP 123456",
  "active": true
}
```

---

## 🚑 Triagem (Especializado)

### POST `/triagem/realizar`
**Descrição**: Realiza triagem completa com auto-alocação de leito baseada no protocolo Manchester  
**Acesso**: TRIAGEM | ADMIN | ENFERMAGEM  
**Regra**: Se houver leito disponível, aloca automaticamente. Caso contrário, adiciona à fila de espera (NIR)

**Request Body**:
```json
{
  "name": "Maria Santos",
  "documentNumber": "12345678900",
  "birthDate": "1985-05-15",
  "phone": "11987654321",
  "emergencyContact": "João Santos",
  "emergencyPhone": "11912345678",
  "riskColor": "laranja", // vermelho | laranja | amarelo | verde | azul
  "chiefComplaint": "Dor no peito há 2 horas",
  "vitalSigns": {
    "bloodPressure": "150/90",
    "heartRate": 95,
    "temperature": 37.2,
    "oxygenSaturation": 98,
    "respiratoryRate": 18
  },
  "allergies": ["Penicilina"]
}
```

**Response (201)** - Leito Alocado:
```json
{
  "success": true,
  "data": {
    "patient": { /* Patient entity */ },
    "encounter": { /* Encounter entity */ },
    "bedAllocated": true,
    "location": {
      "id": "uuid",
      "alias": "302-A",
      "name": "Leito 302-A - Cardiologia",
      "status": "ocupado"
    }
  },
  "message": "Paciente Maria Santos triado com sucesso! Leito 302-A alocado."
}
```

**Response (201)** - Fila de Espera:
```json
{
  "success": true,
  "data": {
    "patient": { /* Patient entity */ },
    "encounter": { /* Encounter entity */ },
    "bedAllocated": false,
    "queuePosition": 3
  },
  "message": "Paciente Maria Santos triado com sucesso! Adicionado à fila de espera (Posição: 3)."
}
```

---

### GET `/triagem/fila-espera`
**Descrição**: Retorna fila de espera ordenada por prioridade Manchester e tempo de chegada  
**Acesso**: TRIAGEM | ADMIN | ENFERMAGEM | MEDICO

**Response (200)**:
```json
{
  "success": true,
  "total": 5,
  "data": [
    {
      "posicao": 1,
      "patient": {
        "id": "uuid",
        "name": "Carlos Silva",
        "riskColor": "vermelho",
        "chiefComplaint": "Infarto"
      },
      "encounter": {
        "id": "uuid",
        "status": "aguardando_leito",
        "startDateTime": "2026-03-18T10:30:00Z"
      },
      "tempoEspera": "45 minutos"
    },
    // ... mais pacientes
  ]
}
```

---

### PUT `/triagem/alocar/:encounterId/:locationId`
**Descrição**: Aloca paciente da fila de espera em leito específico (alocação manual)  
**Acesso**: TRIAGEM | ADMIN | ENFERMAGEM

**URL Params**:
- `encounterId`: ID do encounter (internação)
- `locationId`: ID do leito (location)

**Response (200)**:
```json
{
  "success": true,
  "data": { /* Encounter atualizado */ },
  "message": "Paciente alocado com sucesso!"
}
```

---

## 👤 Pacientes

### POST `/patients`
**Descrição**: Criar novo paciente  
**Acesso**: TRIAGEM | ADMIN | ENFERMAGEM | MEDICO

**Request Body**: (mesmo do `/triagem/realizar`)

**Response (201)**: Patient entity

---

### GET `/patients`
**Descrição**: Listar todos os pacientes ativos  
**Acesso**: ADMIN | MEDICO | ENFERMAGEM | TRIAGEM

**Query Params** (opcionais):
- `search`: Busca por nome
- `active`: true | false

**Response (200)**:
```json
[
  {
    "id": "uuid",
    "name": "Maria Santos",
    "documentNumber": "12345678900",
    "riskColor": "verde",
    "active": true
  }
  // ... mais pacientes
]
```

---

### GET `/patients/search`
**Descrição**: Buscar paciente por CPF  
**Acesso**: TRIAGEM | ADMIN | ENFERMAGEM | MEDICO

**Query Params**:
- `documentNumber`: CPF do paciente

**Response (200)**: Patient entity ou `null`

---

### GET `/patients/:id`
**Descrição**: Detalhes de um paciente  
**Acesso**: ADMIN | MEDICO | ENFERMAGEM | TRIAGEM

**Response (200)**: Patient entity completo

---

### PUT `/patients/:id`
**Descrição**: Atualizar dados do paciente  
**Acesso**: ADMIN | MEDICO | ENFERMAGEM | TRIAGEM

**Request Body**: Campos a atualizar (parcial)

**Response (200)**: Patient entity atualizado

---

### DELETE `/patients/:id`
**Descrição**: Desativar paciente (soft delete)  
**Acesso**: ADMIN

**Response (200)**:
```json
{
  "message": "Paciente desativado com sucesso"
}
```

---

## 🛏️ Leitos (Locations)

### POST `/locations`
**Descrição**: Criar novo leito ou localização  
**Acesso**: ADMIN

**Request Body**:
```json
{
  "alias": "302-A",
  "name": "Leito 302-A - Cardiologia",
  "type": "leito", // leito | ala | andar | bloco | unidade
  "status": "disponivel",
  "specialty": "Cardiologia",
  "floor": "3º Andar",
  "building": "Bloco A",
  "parentId": "uuid-do-ala", // Opcional, para hierarquia
  "metadata": {
    "equipamentos": ["Monitor Cardíaco", "Respirador"],
    "observacoes": "Leito com vista"
  }
}
```

**Response (201)**: Location entity

---

### GET `/locations`
**Descrição**: Listar todas as localizações  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO | LIMPEZA | TRIAGEM

**Query Params** (opcionais):
- `type`: Filtrar por tipo
- `status`: Filtrar por status
- `specialty`: Filtrar por especialidade

**Response (200)**: Array de Location entities

---

### GET `/locations/available-beds`
**Descrição**: Listar apenas leitos disponíveis  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO | TRIAGEM

**Query Params** (opcionais):
- `specialization`: Filtrar por especialidade

**Response (200)**:
```json
[
  {
    "id": "uuid",
    "alias": "302-A",
    "status": "disponivel",
    "specialty": "Cardiologia"
  }
  // ... mais leitos
]
```

---

### GET `/locations/:id`
**Descrição**: Detalhes de uma localização  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO | LIMPEZA | TRIAGEM

**Response (200)**: Location entity completo

---

### GET `/locations/:id/hierarchy`
**Descrição**: Obter caminho hierárquico completo  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO

**Response (200)**:
```json
{
  "path": "Hospital Central > Bloco A > 3º Andar > Ala Sul > 302-A"
}
```

---

### PUT `/locations/:id`
**Descrição**: Atualizar dados da localização  
**Acesso**: ADMIN

**Request Body**: Campos a atualizar (parcial)

**Response (200)**: Location entity atualizado

---

### PATCH `/locations/:id/status`
**Descrição**: Atualizar status do leito  
**Acesso**: ADMIN | ENFERMEIRO | LIMPEZA  
**Regra**: RN.02 - Não permite DISPONIVEL sem checklist completo

**Request Body**:
```json
{
  "status": "disponivel", // disponivel | ocupado | ocupado_ausente | higienizacao_necessaria | higienizacao_em_andamento | manutencao | bloqueado
  "reason": "Limpeza concluída" // Opcional
}
```

**Response (200)**: Location entity atualizado

**Response (400)** - RN.02 violada:
```json
{
  "statusCode": 400,
  "message": "Não é possível marcar leito como disponível. Checklist de limpeza incompleto.",
  "error": "Bad Request"
}
```

---

### DELETE `/locations/:id`
**Descrição**: Excluir localização  
**Acesso**: ADMIN  
**Regra**: Não pode excluir se houver encounter ativo

**Response (200)**:
```json
{
  "message": "Leito excluído com sucesso"
}
```

---

## 🏥 Internações (Encounters)

### POST `/encounters`
**Descrição**: Criar nova internação  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO

**Request Body**:
```json
{
  "patientId": "uuid",
  "locationId": "uuid", // Opcional, se ainda na fila
  "responsibleDoctorId": "uuid", // Opcional
  "status": "em_atendimento",
  "startDateTime": "2026-03-18T10:00:00Z",
  "diagnosis": "Diagnóstico inicial",
  "treatmentPlan": "Plano de tratamento"
}
```

**Response (201)**: Encounter entity

---

### GET `/encounters`
**Descrição**: Listar todas as internações  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO

**Query Params** (opcionais):
- `status`: Filtrar por status
- `responsibleDoctorId`: Filtrar por médico

**Response (200)**: Array de Encounter entities

---

### GET `/encounters/active`
**Descrição**: Listar apenas internações ativas  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO

**Response (200)**: Array de Encounter entities com status ativo

---

### GET `/encounters/:id`
**Descrição**: Detalhes de uma internação  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO | ACOMPANHANTE (limitado)

**Response (200)**: Encounter entity completo com patient, location, responsibleDoctor

---

### GET `/encounters/patient/:patientId`
**Descrição**: Histórico de internações de um paciente  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO

**Response (200)**: Array de Encounter entities do paciente

---

### PUT `/encounters/:id`
**Descrição**: Atualizar dados da internação  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO

**Request Body**: Campos a atualizar (parcial)

**Response (200)**: Encounter entity atualizado

---

### PATCH `/encounters/:id/status`
**Descrição**: Atualizar status da internação  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO

**Request Body**:
```json
{
  "status": "em_medicacao",
  "note": "Iniciando medicação endovenosa" // Opcional
}
```

**Response (200)**: Encounter entity atualizado

---

### PATCH `/encounters/:id/estimated-discharge-date`
**Descrição**: Definir ou alterar previsão de alta (EDD)  
**Acesso**: MEDICO  
**Regra**: RN.05 - Justificativa obrigatória, gera AuditLog

**Request Body**:
```json
{
  "estimatedDischargeDate": "2026-03-25T00:00:00Z",
  "justification": "Paciente apresentou melhora significativa nos sinais vitais"
}
```

**Response (200)**: Encounter entity atualizado

**Efeito Colateral**: Cria registro em AuditLog

---

### POST `/encounters/:id/notes`
**Descrição**: Adicionar nota de evolução  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO

**Request Body**:
```json
{
  "content": "Paciente apresenta melhora no quadro. Sinais vitais estáveis."
}
```

**Response (200)**: Encounter entity atualizado com nova nota

---

### POST `/encounters/:id/discharge`
**Descrição**: Realizar alta do paciente  
**Acesso**: MEDICO

**Request Body**:
```json
{
  "dischargeReason": "Melhora clínica completa",
  "notes": "Alta com prescrição de medicamentos"
}
```

**Response (200)**: Encounter entity atualizado

**Efeito Colateral**:
- Encounter.status → `alta_realizada`
- Location.status → `higienizacao_necessaria`
- Cria Task de limpeza automaticamente

---

## ✅ Tarefas (Tasks)

### POST `/tasks`
**Descrição**: Criar nova tarefa  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO | LIMPEZA | TRIAGEM

**Request Body**:
```json
{
  "type": "limpeza", // limpeza | limpeza_emergencia | manutencao | transporte | medicacao | exame | visita_medica
  "priority": "alta", // baixa | normal | alta | urgente | critica
  "locationId": "uuid",
  "encounterId": "uuid", // Opcional
  "description": "Limpeza completa do leito após alta",
  "scheduledFor": "2026-03-18T14:00:00Z", // Opcional
  "checklist": [
    { "item": "Trocar roupa de cama", "completed": false },
    { "item": "Desinfecção de superfícies", "completed": false },
    { "item": "Limpar banheiro", "completed": false }
  ]
}
```

**Response (201)**: Task entity

---

### GET `/tasks`
**Descrição**: Listar todas as tarefas  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO | LIMPEZA

**Query Params** (opcionais):
- `status`: Filtrar por status
- `type`: Filtrar por tipo
- `assignedToId`: Filtrar por responsável

**Response (200)**: Array de Task entities

---

### GET `/tasks/pending`
**Descrição**: Listar tarefas pendentes (solicitadas ou aceitas)  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO | LIMPEZA

**Response (200)**: Array de Task entities ordenadas por prioridade

---

### GET `/tasks/cleaning`
**Descrição**: Listar apenas tarefas de limpeza  
**Acesso**: ADMIN | LIMPEZA

**Response (200)**: Array de Task entities de limpeza  
**Ordenação**: Prioridade (CRITICA primeiro - RN.04), depois tempo

---

### GET `/tasks/:id`
**Descrição**: Detalhes de uma tarefa  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO | LIMPEZA

**Response (200)**: Task entity completo com location, assignedTo, requestedBy

---

### PUT `/tasks/:id`
**Descrição**: Atualizar dados da tarefa  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO | LIMPEZA

**Request Body**: Campos a atualizar (parcial)

**Response (200)**: Task entity atualizado

---

### PATCH `/tasks/:id/status`
**Descrição**: Atualizar status da tarefa  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO | LIMPEZA

**Request Body**:
```json
{
  "status": "em_andamento"
}
```

**Response (200)**: Task entity atualizado

---

### PATCH `/tasks/:id/assign`
**Descrição**: Atribuir tarefa a um profissional  
**Acesso**: ADMIN | LIMPEZA (auto-atribuição)

**Request Body**:
```json
{
  "assignedToId": "uuid"
}
```

**Response (200)**: Task entity atualizado

**Efeito Colateral**:
- Task.status → `aceita`
- Task.acceptedAt → timestamp atual

---

### POST `/tasks/:id/complete`
**Descrição**: Marcar tarefa como concluída  
**Acesso**: ADMIN | Profissional atribuído

**Request Body**:
```json
{
  "notes": "Limpeza concluída com sucesso"
}
```

**Response (200)**: Task entity atualizado

**Efeito Colateral**:
- Task.status → `concluida`
- Task.completedAt → timestamp atual
- Se task.type === 'limpeza', permite liberar leito (via PATCH /locations/:id/status)

---

## 💊 Medicações (Medications)

### POST `/medications`
**Descrição**: Registrar nova medicação  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO

**Request Body**:
```json
{
  "encounterId": "uuid",
  "patientId": "uuid",
  "medicationName": "Dipirona 500mg",
  "dosage": "1 comprimido",
  "route": "oral", // oral | endovenosa | intramuscular | subcutânea
  "frequency": "8/8h",
  "scheduledDateTime": "2026-03-18T14:00:00Z"
}
```

**Response (201)**: MedicationAdministration entity

---

### GET `/medications`
**Descrição**: Listar todas as medicações  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO

**Query Params** (opcionais):
- `status`: Filtrar por status
- `encounterId`: Filtrar por internação

**Response (200)**: Array de MedicationAdministration entities

---

### GET `/medications/encounter/:encounterId`
**Descrição**: Listar medicações de uma internação  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO

**Response (200)**: Array de MedicationAdministration entities

---

### GET `/medications/pending`
**Descrição**: Listar medicações pendentes de administração  
**Acesso**: ADMIN | ENFERMEIRO

**Response (200)**: Array de MedicationAdministration entities ordenadas por scheduledDateTime

---

### PATCH `/medications/:id/administer`
**Descrição**: Registrar administração da medicação  
**Acesso**: ADMIN | ENFERMEIRO

**Request Body**:
```json
{
  "effectiveDateTime": "2026-03-18T14:05:00Z", // Opcional, default: agora
  "notes": "Medicação administrada sem intercorrências"
}
```

**Response (200)**: MedicationAdministration entity atualizado

**Efeito Colateral**:
- MedicationAdministration.status → `concluido`
- MedicationAdministration.administeredById → ID do usuário
- Encounter.status temporariamente → `em_medicacao` (durante administração)

---

## 🔬 Exames (Service Requests)

### POST `/service-requests`
**Descrição**: Solicitar novo exame ou procedimento  
**Acesso**: ADMIN | MEDICO

**Request Body**:
```json
{
  "encounterId": "uuid",
  "patientId": "uuid",
  "requestType": "Raio-X de Tórax",
  "priority": "urgente", // rotina | urgente | emergencia
  "occurrenceDateTime": "2026-03-18T16:00:00Z", // Opcional
  "notes": "Suspeita de pneumonia"
}
```

**Response (201)**: ServiceRequest entity

---

### GET `/service-requests`
**Descrição**: Listar todas as solicitações de exames  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO

**Query Params** (opcionais):
- `status`: Filtrar por status
- `priority`: Filtrar por prioridade
- `encounterId`: Filtrar por internação

**Response (200)**: Array de ServiceRequest entities

---

### GET `/service-requests/encounter/:encounterId`
**Descrição**: Listar exames de uma internação  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO

**Response (200)**: Array de ServiceRequest entities

---

### PATCH `/service-requests/:id`
**Descrição**: Atualizar solicitação de exame  
**Acesso**: ADMIN | MEDICO | ENFERMEIRO

**Request Body**: Campos a atualizar (parcial)

**Response (200)**: ServiceRequest entity atualizado

---

### PATCH `/service-requests/:id/result`
**Descrição**: Registrar resultado do exame  
**Acesso**: ADMIN | MEDICO

**Request Body**:
```json
{
  "result": "Raio-X normal, sem alterações significativas",
  "status": "concluido" // Opcional, default: concluido
}
```

**Response (200)**: ServiceRequest entity atualizado

---

## 👥 Usuários (Users)

### GET `/users`
**Descrição**: Listar todos os usuários  
**Acesso**: ADMIN

**Query Params** (opcionais):
- `role`: Filtrar por role
- `active`: Filtrar por ativo/inativo

**Response (200)**: Array de User entities (sem senha)

---

### GET `/users/:id`
**Descrição**: Detalhes de um usuário  
**Acesso**: ADMIN

**Response (200)**: User entity (sem senha)

---

### PUT `/users/:id`
**Descrição**: Atualizar dados do usuário  
**Acesso**: ADMIN

**Request Body**: Campos a atualizar (parcial)

**Response (200)**: User entity atualizado

---

### DELETE `/users/:id`
**Descrição**: Desativar usuário (soft delete)  
**Acesso**: ADMIN

**Response (200)**:
```json
{
  "message": "Usuário desativado com sucesso"
}
```

---

## 📊 Códigos de Status HTTP

### Sucesso
- **200 OK**: Requisição bem-sucedida
- **201 Created**: Recurso criado com sucesso

### Erros do Cliente
- **400 Bad Request**: Dados inválidos ou regra de negócio violada
- **401 Unauthorized**: Token ausente ou inválido
- **403 Forbidden**: Sem permissão para acessar o recurso
- **404 Not Found**: Recurso não encontrado
- **409 Conflict**: Conflito (ex: CPF duplicado)

### Erros do Servidor
- **500 Internal Server Error**: Erro interno do servidor

---

## 🔒 Matriz de Permissões (RBAC)

| Endpoint | ADMIN | MEDICO | ENFERMEIRO | ENFERMAGEM | TRIAGEM | LIMPEZA | ACOMPANHANTE |
|----------|-------|--------|------------|------------|---------|---------|--------------|
| POST /auth/login | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| GET /auth/profile | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| POST /triagem/realizar | ✅ | ❌ | ❌ | ✅ | ✅ | ❌ | ❌ |
| GET /triagem/fila-espera | ✅ | ✅ | ❌ | ✅ | ✅ | ❌ | ❌ |
| POST /patients | ✅ | ✅ | ❌ | ✅ | ✅ | ❌ | ❌ |
| GET /patients | ✅ | ✅ | ❌ | ✅ | ✅ | ❌ | ❌ |
| POST /locations | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| PATCH /locations/:id/status | ✅ | ❌ | ✅ | ❌ | ❌ | ✅ | ❌ |
| POST /encounters | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| GET /encounters/:id | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ✅* |
| PATCH /encounters/:id/estimated-discharge-date | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| POST /encounters/:id/discharge | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| GET /tasks/cleaning | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| PATCH /tasks/:id/assign | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| POST /tasks/:id/complete | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| POST /medications | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| PATCH /medications/:id/administer | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |
| POST /service-requests | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| PATCH /service-requests/:id/result | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |

*Acompanhante: Acesso limitado apenas aos encounters vinculados ao seu paciente

---

## 🧪 Exemplos de Uso (cURL)

### Exemplo 1: Login e obter token
```bash
curl -X POST http://localhost:3000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "medico@hospital.com",
    "password": "senha123"
  }'
```

### Exemplo 2: Realizar triagem
```bash
curl -X POST http://localhost:3000/api/v1/triagem/realizar \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {seu-token}" \
  -d '{
    "name": "Maria Santos",
    "documentNumber": "12345678900",
    "birthDate": "1985-05-15",
    "riskColor": "amarelo",
    "chiefComplaint": "Febre alta há 2 dias"
  }'
```

### Exemplo 3: Listar leitos disponíveis
```bash
curl -X GET "http://localhost:3000/api/v1/locations/available-beds?specialization=Cardiologia" \
  -H "Authorization: Bearer {seu-token}"
```

### Exemplo 4: Aceitar tarefa de limpeza
```bash
curl -X PATCH http://localhost:3000/api/v1/tasks/{task-id}/assign \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {seu-token}" \
  -d '{
    "assignedToId": "{seu-user-id}"
  }'
```

### Exemplo 5: Definir previsão de alta
```bash
curl -X PATCH http://localhost:3000/api/v1/encounters/{encounter-id}/estimated-discharge-date \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {seu-token}" \
  -d '{
    "estimatedDischargeDate": "2026-03-25T00:00:00Z",
    "justification": "Paciente apresentou melhora significativa"
  }'
```

---

## 🚀 Como Testar no Postman

1. Importe a coleção (se disponível) ou crie manualmente
2. Configure environment com `baseUrl`: `http://localhost:3000/api/v1`
3. Faça login em `/auth/login` e salve o token
4. Configure Authorization > Bearer Token com o token obtido
5. Teste os endpoints seguindo a ordem lógica:
   - Criar usuário admin
   - Login
   - Criar leitos
   - Realizar triagem
   - Gerenciar internações
   - Criar tarefas

---

**Atualizado em**: 18 de março de 2026  
**Versão**: 2.0 - Documentação completa dos endpoints implementados
