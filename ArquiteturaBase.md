src/app/
├── components/           # Componentes globais e reutilizáveis (Shared)

├── core/                 # Lógica singleton e serviços globais
│   ├── guards/           # Proteção de rotas
│   ├── interceptors/     # Interceptação de requisições (Auth/Headers)
│   ├── models/           # Interfaces e Types (Encounter, Location, Patient)
│   └── services/         # Services (API, WebSockets, AuthService)
├── pages/                # Componentes que representam telas inteiras (Rotas)
├── app.component.html    # Layout principal (contém o <router-outlet>)
├── app.component.scss    # Estilos globais do componente raiz
├── app.component.ts      # Componente raiz Standalone
├── app.config.ts         # Configurações de Providers (Router, HttpClient, etc.)
└── app.routes.ts         # Definição das rotas com Lazy Loading