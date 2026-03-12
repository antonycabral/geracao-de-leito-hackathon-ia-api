# 📚 API Endpoints - Gestor de Leitos Hospitalares

## 🔧 Configuração Base

**Base URL:** `http://localhost:3000/api/v1`

**Headers Padrão:**
```json
{
  "Content-Type": "application/json",
  "Accept": "application/json"
}
```

**Headers com Autenticação:**
```json
{
  "Content-Type": "application/json",
  "Accept": "application/json",
  "Authorization": "Bearer {seu_token_jwt}"
}
```

---

## 🔐 1. Autenticação (Auth)

### 1.1 Registrar Usuário
**POST** `/auth/register`

**Body:**
```json
{
  "name": "João Silva",
  "email": "joao.silva@hospital.com",
  "password": "senha123456",
  "role": "NURSE"
}
```

**Roles disponíveis:** `ADMIN`, `DOCTOR`, `NURSE`, `COORDINATOR`

**Response (201):**
```json
{
  "user": {
    "id": "uuid",
    "name": "João Silva",
    "email": "joao.silva@hospital.com",
    "role": "NURSE",
    "isActive": true
  },
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

### 1.2 Login
**POST** `/auth/login`

**Body:**
```json
{
  "email": "joao.silva@hospital.com",
  "password": "senha123456"
}
```

**Response (200):**
```json
{
  "user": {
    "id": "uuid",
    "name": "João Silva",
    "email": "joao.silva@hospital.com",
    "role": "NURSE"
  },
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

## 👤 2. Pacientes (Patients)

### 2.1 Listar Todos os Pacientes
**GET** `/patients`

**Headers:** Bearer Token (qualquer role)

**Response (200):**
```json
[
  {
    "id": "uuid",
    "cpf": "12345678901",
    "name": "Maria Santos",
    "birthDate": "1980-05-15",
    "gender": "female",
    "phone": "(11) 98765-4321",
    "emergencyContact": {
      "name": "José Santos",
      "phone": "(11) 91234-5678",
      "relationship": "Esposo"
    },
    "medicalHistory": [
      {
        "condition": "Hipertensão",
        "diagnosedDate": "2020-01-10",
        "status": "active"
      }
    ],
    "allergies": ["Dipirona", "Penicilina"],
    "vitalSigns": {
      "bloodPressure": "130/85",
      "heartRate": 78,
      "temperature": 36.5,
      "oxygenSaturation": 98,
      "recordedAt": "2026-03-11T10:30:00Z"
    },
    "riskColor": "YELLOW",
    "isActive": true
  }
]
```

---

### 2.2 Buscar Paciente por ID
**GET** `/patients/:id`

**Headers:** Bearer Token

**Response (200):** Objeto do paciente (mesmo formato acima)

---

### 2.3 Criar Paciente
**POST** `/patients`

**Headers:** Bearer Token (NURSE, DOCTOR, COORDINATOR, ADMIN)

**Body:**
```json
{
  "cpf": "12345678901",
  "name": "Maria Santos",
  "birthDate": "1980-05-15",
  "gender": "female",
  "phone": "(11) 98765-4321",
  "emergencyContact": {
    "name": "José Santos",
    "phone": "(11) 91234-5678",
    "relationship": "Esposo"
  },
  "medicalHistory": [
    {
      "condition": "Hipertensão",
      "diagnosedDate": "2020-01-10",
      "status": "active"
    }
  ],
  "allergies": ["Dipirona"],
  "vitalSigns": {
    "bloodPressure": "130/85",
    "heartRate": 78,
    "temperature": 36.5,
    "oxygenSaturation": 98,
    "recordedAt": "2026-03-11T10:30:00Z"
  },
  "riskColor": "YELLOW"
}
```

**Response (201):** Objeto do paciente criado

---

### 2.4 Atualizar Paciente
**PUT** `/patients/:id`

**Headers:** Bearer Token (NURSE, DOCTOR, COORDINATOR, ADMIN)

**Body:** (mesmos campos do POST, todos opcionais)
```json
{
  "name": "Maria Santos Oliveira",
  "phone": "(11) 98765-9999",
  "vitalSigns": {
    "bloodPressure": "125/80",
    "heartRate": 72,
    "temperature": 36.3,
    "oxygenSaturation": 99,
    "recordedAt": "2026-03-11T14:00:00Z"
  },
  "riskColor": "GREEN"
}
```

**Response (200):** Objeto do paciente atualizado

---

### 2.5 Deletar Paciente (Soft Delete)
**DELETE** `/patients/:id`

**Headers:** Bearer Token (COORDINATOR, ADMIN)

**Response (200):**
```json
{
  "message": "Paciente desativado com sucesso"
}
```

---

## 🏥 3. Localizações/Leitos (Locations)

### 3.1 Listar Todas as Localizações
**GET** `/locations`

**Headers:** Bearer Token

**Response (200):**
```json
[
  {
    "id": "uuid",
    "name": "UTI - Leito 101",
    "type": "BED",
    "status": "AVAILABLE",
    "parent": {
      "id": "uuid-parent",
      "name": "UTI Adulto",
      "type": "WARD"
    },
    "capacity": 1,
    "currentOccupancy": 0,
    "features": ["Ventilador Mecânico", "Monitor Cardíaco"],
    "isActive": true
  }
]
```

**Location Types:** `BUILDING`, `FLOOR`, `WARD`, `ROOM`, `BED`

**Location Status:** `AVAILABLE`, `OCCUPIED`, `MAINTENANCE`, `CLEANING`, `RESERVED`

---

### 3.2 Buscar Localização por ID
**GET** `/locations/:id`

**Headers:** Bearer Token

**Response (200):** Objeto da localização

---

### 3.3 Buscar Leitos Disponíveis
**GET** `/locations/available-beds`

**Headers:** Bearer Token

**Query Parameters (opcionais):**
- `type` - Filtrar por tipo (ex: UTI, WARD)
- `features` - Filtrar por recursos necessários

**Response (200):** Array de localizações com status AVAILABLE

---

### 3.4 Buscar Hierarquia de Localizações
**GET** `/locations/hierarchy`

**Headers:** Bearer Token

**Response (200):**
```json
[
  {
    "id": "uuid",
    "name": "Prédio Principal",
    "type": "BUILDING",
    "children": [
      {
        "id": "uuid",
        "name": "3º Andar",
        "type": "FLOOR",
        "children": [
          {
            "id": "uuid",
            "name": "UTI Adulto",
            "type": "WARD",
            "children": [
              {
                "id": "uuid",
                "name": "Leito 101",
                "type": "BED",
                "status": "AVAILABLE"
              }
            ]
          }
        ]
      }
    ]
  }
]
```

---

### 3.5 Criar Localização/Leito
**POST** `/locations`

**Headers:** Bearer Token (COORDINATOR, ADMIN)

**Body:**
```json
{
  "name": "UTI - Leito 102",
  "type": "BED",
  "status": "AVAILABLE",
  "parentId": "uuid-of-ward",
  "capacity": 1,
  "features": ["Ventilador Mecânico", "Monitor Cardíaco", "Bomba de Infusão"]
}
```

**Response (201):** Objeto da localização criada

---

### 3.6 Atualizar Localização
**PUT** `/locations/:id`

**Headers:** Bearer Token (COORDINATOR, ADMIN)

**Body:** (todos campos opcionais)
```json
{
  "name": "UTI - Leito 102A",
  "status": "MAINTENANCE",
  "features": ["Ventilador Mecânico", "Monitor Cardíaco", "Bomba de Infusão", "ECMO"]
}
```

**Response (200):** Objeto da localização atualizada

---

### 3.7 Atualizar Status da Localização
**PATCH** `/locations/:id/status`

**Headers:** Bearer Token (NURSE, DOCTOR, COORDINATOR, ADMIN)

**Body:**
```json
{
  "status": "CLEANING",
  "reason": "Limpeza terminal após alta de paciente COVID-19"
}
```

**Response (200):** Objeto da localização atualizada

---

### 3.8 Deletar Localização (Soft Delete)
**DELETE** `/locations/:id`

**Headers:** Bearer Token (ADMIN)

**Response (200):**
```json
{
  "message": "Localização desativada com sucesso"
}
```

---

## 🛏️ 4. Internações (Encounters)

### 4.1 Listar Todas as Internações
**GET** `/encounters`

**Headers:** Bearer Token

**Response (200):**
```json
[
  {
    "id": "uuid",
    "patient": {
      "id": "uuid",
      "name": "Maria Santos",
      "cpf": "12345678901"
    },
    "location": {
      "id": "uuid",
      "name": "UTI - Leito 101",
      "type": "BED"
    },
    "status": "IN_PROGRESS",
    "admissionDate": "2026-03-10T08:00:00Z",
    "estimatedDischargeDate": "2026-03-15T10:00:00Z",
    "dischargeDate": null,
    "dischargeType": null,
    "admissionReason": "Insuficiência respiratória aguda",
    "diagnosis": "Pneumonia bacteriana grave",
    "notes": [
      {
        "date": "2026-03-10T14:00:00Z",
        "author": "Dr. João Silva",
        "text": "Paciente estável, respondendo bem ao tratamento antibiótico"
      }
    ]
  }
]
```

**Encounter Status:** `PLANNED`, `IN_PROGRESS`, `FINISHED`, `CANCELLED`

**Discharge Types:** `HOME`, `TRANSFER`, `DEATH`, `EVASION`, `ADMINISTRATIVE`

---

### 4.2 Buscar Internação por ID
**GET** `/encounters/:id`

**Headers:** Bearer Token

**Response (200):** Objeto da internação

---

### 4.3 Buscar Internações Ativas
**GET** `/encounters/active`

**Headers:** Bearer Token

**Response (200):** Array de internações com status IN_PROGRESS

---

### 4.4 Criar Internação (Admissão)
**POST** `/encounters`

**Headers:** Bearer Token (DOCTOR, COORDINATOR, ADMIN)

**Body:**
```json
{
  "patientId": "uuid-of-patient",
  "locationId": "uuid-of-bed",
  "admissionDate": "2026-03-11T10:00:00Z",
  "estimatedDischargeDate": "2026-03-15T10:00:00Z",
  "admissionReason": "Insuficiência respiratória aguda",
  "diagnosis": "Pneumonia bacteriana grave"
}
```

**Response (201):** Objeto da internação criada

---

### 4.5 Atualizar Internação
**PUT** `/encounters/:id`

**Headers:** Bearer Token (DOCTOR, COORDINATOR, ADMIN)

**Body:** (todos campos opcionais)
```json
{
  "diagnosis": "Pneumonia bacteriana grave com melhora clínica",
  "estimatedDischargeDate": "2026-03-14T10:00:00Z"
}
```

**Response (200):** Objeto da internação atualizada

---

### 4.6 Realizar Alta/Discharge
**POST** `/encounters/:id/discharge`

**Headers:** Bearer Token (DOCTOR, COORDINATOR, ADMIN)

**Body:**
```json
{
  "dischargeDate": "2026-03-13T15:00:00Z",
  "dischargeType": "HOME",
  "dischargeNotes": "Paciente com melhora clínica significativa. Alta para acompanhamento ambulatorial."
}
```

**Response (200):** Objeto da internação finalizada

---

### 4.7 Atualizar Data Prevista de Alta
**PATCH** `/encounters/:id/estimated-discharge-date`

**Headers:** Bearer Token (DOCTOR, COORDINATOR, ADMIN)

**Body:**
```json
{
  "estimatedDischargeDate": "2026-03-16T10:00:00Z",
  "justification": "Paciente apresentou complicação, necessita mais 2 dias de observação"
}
```

**Response (200):** Objeto da internação atualizada

---

### 4.8 Adicionar Nota à Internação
**POST** `/encounters/:id/notes`

**Headers:** Bearer Token (DOCTOR, NURSE, COORDINATOR, ADMIN)

**Body:**
```json
{
  "text": "Paciente apresentou febre de 38.2°C às 14h. Iniciado antipirético conforme prescrição."
}
```

**Response (200):** Objeto da internação com nova nota

---

### 4.9 Deletar Internação
**DELETE** `/encounters/:id`

**Headers:** Bearer Token (ADMIN)

**Response (200):**
```json
{
  "message": "Internação removida com sucesso"
}
```

---

## ✅ 5. Tarefas (Tasks)

### 5.1 Listar Todas as Tarefas
**GET** `/tasks`

**Headers:** Bearer Token

**Query Parameters (opcionais):**
- `status` - Filtrar por status (PENDING, ACCEPTED, IN_PROGRESS, COMPLETED, CANCELLED)
- `type` - Filtrar por tipo (CLEANING, MAINTENANCE, TRANSPORT, CARE, ADMINISTRATIVE)

**Response (200):**
```json
[
  {
    "id": "uuid",
    "type": "CLEANING",
    "status": "IN_PROGRESS",
    "priority": "HIGH",
    "title": "Limpeza terminal - UTI Leito 101",
    "description": "Limpeza completa após alta de paciente COVID-19",
    "location": {
      "id": "uuid",
      "name": "UTI - Leito 101"
    },
    "encounter": null,
    "assignedTo": {
      "id": "uuid",
      "name": "João Silva",
      "role": "NURSE"
    },
    "createdBy": {
      "id": "uuid",
      "name": "Ana Paula"
    },
    "dueDate": "2026-03-11T16:00:00Z",
    "startedAt": "2026-03-11T14:30:00Z",
    "completedAt": null,
    "checklist": [
      {
        "item": "Remover roupa de cama",
        "completed": true
      },
      {
        "item": "Limpar superfícies com álcool 70%",
        "completed": true
      },
      {
        "item": "Desinfetar equipamentos médicos",
        "completed": false
      },
      {
        "item": "Colocar roupa de cama limpa",
        "completed": false
      }
    ]
  }
]
```

**Task Types:** `CLEANING`, `MAINTENANCE`, `TRANSPORT`, `CARE`, `ADMINISTRATIVE`

**Task Status:** `PENDING`, `ACCEPTED`, `IN_PROGRESS`, `COMPLETED`, `CANCELLED`

**Task Priority:** `LOW`, `MEDIUM`, `HIGH`, `CRITICAL`

---

### 5.2 Buscar Tarefa por ID
**GET** `/tasks/:id`

**Headers:** Bearer Token

**Response (200):** Objeto da tarefa

---

### 5.3 Buscar Minhas Tarefas
**GET** `/tasks/my-tasks`

**Headers:** Bearer Token

**Response (200):** Array de tarefas atribuídas ao usuário logado

---

### 5.4 Criar Tarefa
**POST** `/tasks`

**Headers:** Bearer Token (NURSE, DOCTOR, COORDINATOR, ADMIN)

**Body:**
```json
{
  "type": "CLEANING",
  "priority": "HIGH",
  "title": "Limpeza terminal - UTI Leito 101",
  "description": "Limpeza completa após alta de paciente COVID-19",
  "locationId": "uuid-of-location",
  "encounterId": null,
  "assignedToId": "uuid-of-user",
  "dueDate": "2026-03-11T16:00:00Z",
  "checklist": [
    { "item": "Remover roupa de cama", "completed": false },
    { "item": "Limpar superfícies com álcool 70%", "completed": false },
    { "item": "Desinfetar equipamentos médicos", "completed": false },
    { "item": "Colocar roupa de cama limpa", "completed": false }
  ]
}
```

**Response (201):** Objeto da tarefa criada

---

### 5.5 Atualizar Tarefa
**PUT** `/tasks/:id`

**Headers:** Bearer Token (NURSE, DOCTOR, COORDINATOR, ADMIN)

**Body:** (todos campos opcionais)
```json
{
  "priority": "CRITICAL",
  "dueDate": "2026-03-11T15:00:00Z",
  "description": "URGENTE: Limpeza terminal - paciente com bactéria multirresistente"
}
```

**Response (200):** Objeto da tarefa atualizada

---

### 5.6 Aceitar Tarefa
**POST** `/tasks/:id/accept`

**Headers:** Bearer Token (NURSE, COORDINATOR, ADMIN)

**Response (200):** Objeto da tarefa com status ACCEPTED

---

### 5.7 Iniciar Tarefa
**POST** `/tasks/:id/start`

**Headers:** Bearer Token (NURSE, COORDINATOR, ADMIN)

**Response (200):** Objeto da tarefa com status IN_PROGRESS

---

### 5.8 Completar Tarefa
**POST** `/tasks/:id/complete`

**Headers:** Bearer Token (NURSE, COORDINATOR, ADMIN)

**Body:**
```json
{
  "checklist": [
    { "item": "Remover roupa de cama", "completed": true },
    { "item": "Limpar superfícies com álcool 70%", "completed": true },
    { "item": "Desinfetar equipamentos médicos", "completed": true },
    { "item": "Colocar roupa de cama limpa", "completed": true }
  ],
  "completionNotes": "Limpeza realizada conforme protocolo. Leito pronto para novo paciente."
}
```

**Response (200):** Objeto da tarefa com status COMPLETED

---

### 5.9 Deletar Tarefa
**DELETE** `/tasks/:id`

**Headers:** Bearer Token (COORDINATOR, ADMIN)

**Response (200):**
```json
{
  "message": "Tarefa removida com sucesso"
}
```

---

## 💊 6. Medicações (Medications)

### 6.1 Listar Todas as Medicações
**GET** `/medications`

**Headers:** Bearer Token

**Response (200):**
```json
[
  {
    "id": "uuid",
    "encounter": {
      "id": "uuid",
      "patient": {
        "name": "Maria Santos"
      }
    },
    "medicationName": "Dipirona 1g",
    "dosage": "1g",
    "route": "IV",
    "frequency": "6/6h",
    "scheduledTime": "2026-03-11T14:00:00Z",
    "administeredAt": "2026-03-11T14:05:00Z",
    "administeredBy": {
      "id": "uuid",
      "name": "João Silva",
      "role": "NURSE"
    },
    "status": "COMPLETED",
    "notes": "Administrado sem intercorrências"
  }
]
```

**Medication Status:** `SCHEDULED`, `IN_PROGRESS`, `COMPLETED`, `SUSPENDED`, `CANCELLED`

**Routes:** `ORAL`, `IV`, `IM`, `SC`, `TOPICAL`, `INHALATION`, `NASAL`, `RECTAL`

---

### 6.2 Buscar Medicação por ID
**GET** `/medications/:id`

**Headers:** Bearer Token

**Response (200):** Objeto da medicação

---

### 6.3 Buscar Medicações Agendadas para Hoje
**GET** `/medications/scheduled-today`

**Headers:** Bearer Token

**Response (200):** Array de medicações com scheduledTime no dia atual

---

### 6.4 Buscar Medicações Atrasadas
**GET** `/medications/overdue`

**Headers:** Bearer Token

**Response (200):** Array de medicações com scheduledTime no passado e status SCHEDULED

---

### 6.5 Criar Medicação/Agendamento
**POST** `/medications`

**Headers:** Bearer Token (DOCTOR, NURSE, COORDINATOR, ADMIN)

**Body:**
```json
{
  "encounterId": "uuid-of-encounter",
  "medicationName": "Dipirona 1g",
  "dosage": "1g",
  "route": "IV",
  "frequency": "6/6h",
  "scheduledTime": "2026-03-11T14:00:00Z",
  "notes": "Administrar em caso de febre acima de 37.8°C"
}
```

**Response (201):** Objeto da medicação criada

---

### 6.6 Atualizar Medicação
**PUT** `/medications/:id`

**Headers:** Bearer Token (DOCTOR, NURSE, COORDINATOR, ADMIN)

**Body:** (todos campos opcionais)
```json
{
  "dosage": "500mg",
  "scheduledTime": "2026-03-11T15:00:00Z",
  "notes": "Dose reduzida devido à melhora do quadro"
}
```

**Response (200):** Objeto da medicação atualizada

---

### 6.7 Administrar Medicação
**POST** `/medications/:id/administer`

**Headers:** Bearer Token (NURSE, DOCTOR, COORDINATOR, ADMIN)

**Body:**
```json
{
  "administeredAt": "2026-03-11T14:05:00Z",
  "notes": "Administrado sem intercorrências. Paciente sem queixas."
}
```

**Response (200):** Objeto da medicação com status COMPLETED

---

### 6.8 Completar Medicação (manualmente)
**POST** `/medications/:id/complete`

**Headers:** Bearer Token (NURSE, DOCTOR, COORDINATOR, ADMIN)

**Body:**
```json
{
  "completionNotes": "Medicação suspensa por ordem médica"
}
```

**Response (200):** Objeto da medicação com status COMPLETED/SUSPENDED

---

### 6.9 Deletar Medicação
**DELETE** `/medications/:id`

**Headers:** Bearer Token (DOCTOR, COORDINATOR, ADMIN)

**Response (200):**
```json
{
  "message": "Medicação removida com sucesso"
}
```

---

## 🔬 7. Solicitações de Exames (Service Requests)

### 7.1 Listar Todas as Solicitações
**GET** `/service-requests`

**Headers:** Bearer Token

**Response (200):**
```json
[
  {
    "id": "uuid",
    "encounter": {
      "id": "uuid",
      "patient": {
        "name": "Maria Santos"
      }
    },
    "category": "IMAGING",
    "code": "RX-TORAX",
    "description": "Radiografia de Tórax PA e Perfil",
    "status": "REQUESTED",
    "priority": "ROUTINE",
    "requestedAt": "2026-03-11T10:00:00Z",
    "requestedBy": {
      "id": "uuid",
      "name": "Dr. João Silva",
      "role": "DOCTOR"
    },
    "scheduledFor": "2026-03-11T15:00:00Z",
    "performedAt": null,
    "completedAt": null,
    "requiresTransport": true,
    "transportStartedAt": null,
    "transportCompletedAt": null,
    "results": null,
    "notes": "Avaliar possível pneumonia"
  }
]
```

**Service Request Categories:** `LABORATORY`, `IMAGING`, `PROCEDURE`, `CONSULTATION`, `OTHER`

**Service Request Status:** `REQUESTED`, `SCHEDULED`, `IN_TRANSPORT`, `IN_PROGRESS`, `COMPLETED`, `CANCELLED`

**Service Request Priority:** `ROUTINE`, `URGENT`, `STAT`

---

### 7.2 Buscar Solicitação por ID
**GET** `/service-requests/:id`

**Headers:** Bearer Token

**Response (200):** Objeto da solicitação

---

### 7.3 Buscar Solicitações Pendentes
**GET** `/service-requests/pending`

**Headers:** Bearer Token

**Response (200):** Array de solicitações com status REQUESTED ou SCHEDULED

---

### 7.4 Criar Solicitação de Exame
**POST** `/service-requests`

**Headers:** Bearer Token (DOCTOR, COORDINATOR, ADMIN)

**Body:**
```json
{
  "encounterId": "uuid-of-encounter",
  "category": "IMAGING",
  "code": "RX-TORAX",
  "description": "Radiografia de Tórax PA e Perfil",
  "priority": "URGENT",
  "scheduledFor": "2026-03-11T15:00:00Z",
  "requiresTransport": true,
  "notes": "Avaliar possível pneumonia - paciente com dispneia"
}
```

**Response (201):** Objeto da solicitação criada

---

### 7.5 Atualizar Solicitação
**PUT** `/service-requests/:id`

**Headers:** Bearer Token (DOCTOR, COORDINATOR, ADMIN)

**Body:** (todos campos opcionais)
```json
{
  "priority": "STAT",
  "scheduledFor": "2026-03-11T14:00:00Z",
  "notes": "URGENTE: Paciente com piora súbita do quadro respiratório"
}
```

**Response (200):** Objeto da solicitação atualizada

---

### 7.6 Iniciar Transporte
**POST** `/service-requests/:id/start-transport`

**Headers:** Bearer Token (NURSE, COORDINATOR, ADMIN)

**Body:**
```json
{
  "transportStartedAt": "2026-03-11T14:30:00Z",
  "transportNotes": "Paciente transportado em maca com O2 suplementar"
}
```

**Response (200):** Objeto da solicitação com status IN_TRANSPORT

---

### 7.7 Iniciar Realização do Exame
**POST** `/service-requests/:id/start-performing`

**Headers:** Bearer Token (NURSE, DOCTOR, COORDINATOR, ADMIN)

**Body:**
```json
{
  "performedAt": "2026-03-11T14:45:00Z"
}
```

**Response (200):** Objeto da solicitação com status IN_PROGRESS

---

### 7.8 Completar Solicitação
**POST** `/service-requests/:id/complete`

**Headers:** Bearer Token (DOCTOR, NURSE, COORDINATOR, ADMIN)

**Body:**
```json
{
  "completedAt": "2026-03-11T15:00:00Z",
  "results": "Imagem radiográfica mostra infiltrado bilateral sugestivo de pneumonia. Sem sinais de derrame pleural.",
  "completionNotes": "Exame realizado sem intercorrências. Paciente retornou ao leito."
}
```

**Response (200):** Objeto da solicitação com status COMPLETED

---

### 7.9 Deletar Solicitação
**DELETE** `/service-requests/:id`

**Headers:** Bearer Token (DOCTOR, COORDINATOR, ADMIN)

**Response (200):**
```json
{
  "message": "Solicitação removida com sucesso"
}
```

---

## 📊 Códigos de Status HTTP

- **200 OK** - Requisição bem-sucedida
- **201 Created** - Recurso criado com sucesso
- **400 Bad Request** - Dados inválidos na requisição
- **401 Unauthorized** - Autenticação necessária ou token inválido
- **403 Forbidden** - Permissão insuficiente para acessar o recurso
- **404 Not Found** - Recurso não encontrado
- **409 Conflict** - Conflito com regras de negócio (ex: leito ocupado)
- **500 Internal Server Error** - Erro no servidor

---

## 🔒 Matriz de Permissões por Role

| Endpoint | ADMIN | COORDINATOR | DOCTOR | NURSE |
|----------|-------|-------------|---------|-------|
| **Auth** | ✅ | ✅ | ✅ | ✅ |
| **Patients - Read** | ✅ | ✅ | ✅ | ✅ |
| **Patients - Create/Update** | ✅ | ✅ | ✅ | ✅ |
| **Patients - Delete** | ✅ | ✅ | ❌ | ❌ |
| **Locations - Read** | ✅ | ✅ | ✅ | ✅ |
| **Locations - Create/Update** | ✅ | ✅ | ❌ | ❌ |
| **Locations - Status Update** | ✅ | ✅ | ✅ | ✅ |
| **Locations - Delete** | ✅ | ❌ | ❌ | ❌ |
| **Encounters - Read** | ✅ | ✅ | ✅ | ✅ |
| **Encounters - Create/Update** | ✅ | ✅ | ✅ | ❌ |
| **Encounters - Discharge** | ✅ | ✅ | ✅ | ❌ |
| **Encounters - Add Notes** | ✅ | ✅ | ✅ | ✅ |
| **Tasks - Read** | ✅ | ✅ | ✅ | ✅ |
| **Tasks - Create/Update** | ✅ | ✅ | ✅ | ✅ |
| **Tasks - Accept/Start/Complete** | ✅ | ✅ | ❌ | ✅ |
| **Medications - Read** | ✅ | ✅ | ✅ | ✅ |
| **Medications - Create/Update** | ✅ | ✅ | ✅ | ✅ |
| **Medications - Administer** | ✅ | ✅ | ✅ | ✅ |
| **Service Requests - Read** | ✅ | ✅ | ✅ | ✅ |
| **Service Requests - Create** | ✅ | ✅ | ✅ | ❌ |
| **Service Requests - Transport** | ✅ | ✅ | ❌ | ✅ |
| **Service Requests - Perform** | ✅ | ✅ | ✅ | ✅ |

---

## 🧪 Fluxo de Teste Completo no Postman

### Passo 1: Registrar Usuário ADMIN
```
POST /auth/register
Body: { "name": "Admin", "email": "admin@hospital.com", "password": "admin123", "role": "ADMIN" }
```

### Passo 2: Login e Copiar Token
```
POST /auth/login
Body: { "email": "admin@hospital.com", "password": "admin123" }
```
→ Copiar `access_token` e adicionar em todas as próximas requisições no header:
`Authorization: Bearer {token}`

### Passo 3: Criar Estrutura de Localizações
```
1. POST /locations - Criar Prédio (type: BUILDING)
2. POST /locations - Criar Andar (type: FLOOR, parentId: id-do-predio)
3. POST /locations - Criar Ala/Enfermaria (type: WARD, parentId: id-do-andar)
4. POST /locations - Criar Leito (type: BED, parentId: id-da-ala, status: AVAILABLE)
```

### Passo 4: Criar Paciente
```
POST /patients
Body: { cpf, name, birthDate, gender, phone, vitalSigns, riskColor }
```

### Passo 5: Realizar Internação
```
POST /encounters
Body: { patientId, locationId, admissionDate, admissionReason, diagnosis }
```
→ Status do leito mudará automaticamente para OCCUPIED

### Passo 6: Criar Tarefa de Medicação
```
POST /medications
Body: { encounterId, medicationName, dosage, route, scheduledTime }
```

### Passo 7: Administrar Medicação
```
POST /medications/{id}/administer
Body: { administeredAt, notes }
```

### Passo 8: Solicitar Exame
```
POST /service-requests
Body: { encounterId, category, code, description, priority, requiresTransport }
```

### Passo 9: Realizar Alta
```
POST /encounters/{id}/discharge
Body: { dischargeDate, dischargeType, dischargeNotes }
```
→ Status do leito mudará automaticamente para CLEANING

### Passo 10: Criar Tarefa de Limpeza
```
POST /tasks
Body: { type: CLEANING, locationId, priority, title, checklist }
```

### Passo 11: Completar Limpeza
```
POST /tasks/{id}/accept
POST /tasks/{id}/start
POST /tasks/{id}/complete
Body: { checklist completo, completionNotes }
```
→ Status do leito mudará automaticamente para AVAILABLE

---

## 💡 Dicas para Teste no Postman

1. **Criar Environment no Postman:**
   - `baseUrl`: `http://localhost:3000/api/v1`
   - `token`: (será preenchido após login)

2. **Usar Pre-request Script para Login Automático:**
```javascript
pm.environment.set("token", pm.response.json().access_token);
```

3. **Configurar Authorization Global:**
   - Type: Bearer Token
   - Token: `{{token}}`

4. **Criar Collection com todas as requisições organizadas por módulo**

5. **Usar Tests para validar responses:**
```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Response has data", function () {
    pm.response.to.have.jsonBody();
});
```

---

## 🚀 Como Iniciar a API

```bash
# Entrar na pasta do projeto
cd gestor-de-leitos-api

# Instalar pacote do Swagger (NECESSÁRIO)
npm install @nestjs/swagger

# Criar arquivo .env (copiar de .env.example)
cp .env.example .env

# Instalar dependências (se ainda não instalou)
npm install

# Iniciar em modo desenvolvimento
npm run start:dev

# Ou build e start em produção
npm run build
npm run start:prod
```

A API estará disponível em:
- **API:** http://localhost:3000/api/v1
- **Swagger UI:** http://localhost:3000/api/docs

---

## 📖 Documentação Swagger

A API possui documentação interativa Swagger disponível em **http://localhost:3000/api/docs**

### Recursos do Swagger:
- 📝 Documentação completa de todos os endpoints
- 🔐 Autenticação JWT integrada (botão "Authorize")
- 🧪 Testar requisições diretamente pelo navegador
- 📋 Schemas de todos os DTOs e modelos
- 💡 Exemplos de requisições e respostas

### Como usar o Swagger:
1. Acesse http://localhost:3000/api/docs
2. Faça login no endpoint `/auth/login` ou `/auth/register`
3. Copie o `access_token` da resposta
4. Clique no botão "Authorize" no topo da página
5. Cole o token (ele adicionará "Bearer" automaticamente)
6. Agora você pode testar qualquer endpoint protegido!

---

**Documentação criada em:** 11 de março de 2026
**Versão da API:** 1.0.0
