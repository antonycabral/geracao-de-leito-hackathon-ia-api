# 🏥 Sistema de Gestão de Leitos Hospitalares

Projeto MVP desenvolvido para o Hackathon de IA da Avanade, com foco em otimização e gestão inteligente de leitos hospitalares.

## 📋 Sobre o Projeto

Sistema backend completo para gerenciamento de leitos hospitalares, desenvolvido com NestJS e TypeScript. A solução oferece uma API RESTful robusta para controle de pacientes, localizações (leitos), encontros médicos e autenticação de usuários.

## ✨ Funcionalidades

- 🔐 **Autenticação e Autorização**: Sistema completo com JWT e controle de acesso baseado em roles
- 👥 **Gestão de Pacientes**: CRUD completo de pacientes com validação de dados
- 🏨 **Gestão de Localizações**: Controle de leitos, enfermarias e unidades hospitalares
- 📊 **Gestão de Encontros**: Registro e acompanhamento de internações e atendimentos
- 🔍 **API com Documentação Swagger**: Interface interativa para testes e documentação
- 🌐 **WebSockets**: Comunicação em tempo real para atualizações de status
- 📝 **Auditoria**: Registro de ações e alterações no sistema

## 🛠️ Tecnologias Utilizadas

- **Framework**: [NestJS](https://nestjs.com/) v11
- **Linguagem**: [TypeScript](https://www.typescriptlang.org/) v5.7
- **Banco de Dados**: SQLite com [TypeORM](https://typeorm.io/) v0.3
- **Autenticação**: JWT (JSON Web Tokens) + Passport.js
- **Documentação**: Swagger/OpenAPI
- **WebSockets**: Socket.io
- **Validação**: class-validator e class-transformer
- **Segurança**: bcrypt para hash de senhas
- **Testes**: Jest

## 📁 Estrutura do Projeto

```
gestor-de-leitos-api/
├── src/
│   ├── auth/              # Autenticação e autorização
│   │   ├── decorators/    # Decorators customizados
│   │   ├── guards/        # Guards de segurança
│   │   └── strategies/    # Estratégias de autenticação
│   ├── config/            # Configurações (DB, JWT, etc)
│   ├── controllers/       # Controladores da API
│   ├── dto/               # Data Transfer Objects
│   ├── models/            # Entidades do banco de dados
│   └── services/          # Lógica de negócio
├── test/                  # Testes E2E
└── package.json
```

## 🚀 Instalação

### Pré-requisitos

- Node.js (versão 18 ou superior)
- npm ou yarn

### Passos

1. Clone o repositório:
```bash
git clone https://github.com/seu-usuario/geracao-de-leito-hackathon-ia-api.git
cd geracao-de-leito-hackathon-ia-api/gestor-de-leitos-api
```

2. Instale as dependências:
```bash
npm install
```

3. Configure as variáveis de ambiente:
```bash
# Crie um arquivo .env na raiz do projeto gestor-de-leitos-api
cp .env.example .env
```

4. Configure o banco de dados (o SQLite será criado automaticamente):
```bash
# O arquivo gestor-de-leitos.sqlite será criado na primeira execução
```

## ▶️ Como Executar

### Desenvolvimento
```bash
npm run start:dev
```

### Produção
```bash
npm run build
npm run start:prod
```

### Testes
```bash
# Testes unitários
npm run test

# Testes E2E
npm run test:e2e

# Cobertura de testes
npm run test:cov
```

## 📚 Documentação da API

Após iniciar o servidor, acesse:

- **Swagger UI**: http://localhost:3000/api
- **API JSON**: http://localhost:3000/api-json

### Endpoints Principais

- `POST /auth/login` - Autenticação de usuário
- `POST /auth/register` - Registro de novo usuário
- `GET /patients` - Listar pacientes
- `POST /patients` - Criar novo paciente
- `GET /locations` - Listar localizações (leitos)
- `POST /locations` - Criar nova localização
- `GET /encounters` - Listar encontros médicos
- `POST /encounters` - Criar novo encontro

Para mais detalhes, consulte:
- [API_ENDPOINTS.md](API_ENDPOINTS.md) - Lista completa de endpoints
- [SWAGGER.md](gestor-de-leitos-api/SWAGGER.md) - Documentação detalhada da API

## 📖 Documentação Adicional

- [INSTALL.md](gestor-de-leitos-api/INSTALL.md) - Guia de instalação detalhado
- [USAGE_GUIDE.md](gestor-de-leitos-api/USAGE_GUIDE.md) - Guia de uso da API
- [PROJETO_COMPLETO.md](gestor-de-leitos-api/PROJETO_COMPLETO.md) - Visão geral do projeto
- [DocumentoDeRefinamento.md](DocumentoDeRefinamento.md) - Documentação de requisitos
- [DiagramaDeClasses.md](DiagramaDeClasses.md) - Diagrama de classes
- [DiagramaDeFLuxo.md](DiagramaDeFLuxo.md) - Diagrama de fluxo

## 🔧 Scripts Disponíveis

```bash
npm run build          # Compila o projeto
npm run start          # Inicia o servidor em produção
npm run start:dev      # Inicia em modo desenvolvimento
npm run start:debug    # Inicia com debug
npm run lint           # Executa o linter
npm run format         # Formata o código
npm run test           # Executa os testes
npm run test:watch     # Executa testes em modo watch
npm run test:cov       # Gera cobertura de testes
```

## 🔐 Segurança

- Senhas criptografadas com bcrypt
- Autenticação JWT com refresh tokens
- Guards de autorização baseados em roles
- Validação de dados de entrada
- Proteção contra injeção SQL via TypeORM

## 🤝 Contribuindo

1. Faça um fork do projeto
2. Crie uma branch para sua feature (`git checkout -b feature/MinhaFeature`)
3. Commit suas mudanças (`git commit -m 'Adiciona MinhaFeature'`)
4. Push para a branch (`git push origin feature/MinhaFeature`)
5. Abra um Pull Request

## 📝 Licença

Este projeto foi desenvolvido para o Hackathon de IA da Avanade.

## 👥 Autores

Equipe do Hackathon - Avanade IA

## 📞 Suporte

Para dúvidas ou problemas, abra uma issue no repositório.

---

**Desenvolvido com ❤️ para o Hackathon de IA da Avanade**
