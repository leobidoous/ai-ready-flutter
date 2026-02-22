# Documentação - cogna-resale-core

Bem-vindo à documentação do **cogna-resale-core**. Este projeto segue os princípios da Clean Architecture, organizando o código em camadas bem definidas.

## Estrutura do Projeto

```
lib/src/
├── domain/          # Regras de negócio e contratos
├── infra/           # Implementações e coordenação
├── data/            # Comunicação com APIs e dados externos
└── presentation/    # UI e gerenciamento de estado
```

## Camadas da Arquitetura

### 1. Domain (Domínio)

Define **O QUE** a aplicação faz através de contratos e regras de negócio.

- [Entities](domain/entities.md) - Objetos de negócio puros
- [Enums](domain/enums.md) - Constantes tipadas
- [Failures](domain/i_failures.md) - Contratos de erros
- [Repositories](domain/i_repositories.md) - Contratos de acesso a dados
- [UseCases](domain/i_usecases.md) - Contratos de operações de negócio
- [Services](domain/i_services.md) - Contratos de serviços auxiliares

### 2. Infra (Infraestrutura)

Implementa **COMO** as operações são coordenadas e transformadas.

- [DataSources](infra/i_datasources.md) - Contratos de comunicação
- [Models](infra/models.md) - Serialização e transformação de dados
- [UseCases](infra/usecases.md) - Implementação de casos de uso
- [Repositories](infra/repositories.md) - Implementação de repositórios

### 3. Data (Dados)

Executa a **COMUNICAÇÃO** real com APIs, banco de dados e cache.

- [DataSources](data/datasources.md) - Implementação de comunicação
- [Drivers](data/drivers.md) - Abstrações de bibliotecas externas

### 4. Presentation (Apresentação)

Gerencia **ESTADO DA UI** e interação com usuário.

- [Controllers](presentation/controllers.md) - Gerenciamento de estado
- [Modules](presentation/modules.md) - Injeção de dependências e rotas
- [Validators](presentation/validators.md) - Validação de formulários e inputs

## Fluxo de Dados

```
UI → Controller → UseCase → Repository → DataSource → Driver → API
                     ↓          ↓            ↓          ↓       ↓
                  Valida   Transforma   Comunica   Executa  Responde
```

### Fluxo de Saída (Envio de dados)

```
Entity → Model → Map/FormData → HTTP → API
```

### Fluxo de Entrada (Recebimento de dados)

```
API → HTTP → Map → Model → Entity
```

## Princípios

### Separação de Responsabilidades

- **Domain**: Define contratos e regras
- **Infra**: Coordena e transforma
- **Data**: Comunica e executa
- **Presentation**: Gerencia estado da UI

### Inversão de Dependência

- Camadas externas dependem de interfaces das camadas internas
- Domain não conhece implementações
- Facilita testes e manutenção

### Testabilidade

- Cada camada pode ser testada isoladamente
- Interfaces facilitam criação de mocks
- Lógica de negócio separada de detalhes técnicos

## Convenções de Nomenclatura

### Interfaces

- Prefixo `I` para todas as interfaces
- Exemplos: `IUserRepository`, `IUserUsecase`, `IUserFailure`

### Implementações

- Sem prefixo, nome descritivo
- Exemplos: `UserRepository`, `UserUsecase`, `UserServerError`

### Models

- Sufixo `Model`
- Exemplo: `UserModel`, `UserResultModel`

### Entities

- Sufixo `Entity`
- Exemplo: `UserEntity`, `BalanceEntity`

### Controllers

- Sufixo `Controller`
- Exemplo: `UserController`, `SessionController`

## Padrões Utilizados

### Either

- Representa operações que podem falhar
- `Left`: Erro (Failure)
- `Right`: Sucesso (Data)

### Repository Pattern

- Abstrai acesso a dados
- Isola lógica de negócio de detalhes de implementação

### UseCase Pattern

- Encapsula operações de negócio
- Um UseCase = Uma operação específica

### Dependency Injection

- Injeção via construtor
- Facilita testes e desacoplamento

## Começando

Para criar um novo módulo, siga esta ordem:

1. **Domain**: Crie Entities, Enums, Failures, Repositories e UseCases
2. **Infra**: Crie DataSources (interface), Models, Repositories e UseCases (implementações)
3. **Data**: Crie DataSources (implementação)
4. **Presentation**: Crie Controllers e Module

Consulte a documentação de cada camada para exemplos completos.
