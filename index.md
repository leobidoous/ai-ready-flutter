# Clean Architecture - Cogna Resale
## 📖 Índice Principal de Documentação

### 🏗️ Visão Geral da Arquitetura

A Clean Architecture está organizada em **3 camadas principais** com responsabilidades bem definidas:

```
┌─────────────────────────────────────────────────────────────┐
│                  PRESENTATION LAYER                         │
│              🎨 Interface & Estado da UI                    │
│                                                             │
│  • Controllers (State Management)                           │
│  • Pages & Widgets                                          │
│  • ValueNotifier + Either Pattern                            │
└─────────────────────┬───────────────────────────────────────┘
                      │ calls
┌─────────────────────▼───────────────────────────────────────┐
│                    DOMAIN LAYER                             │
│         📋 Interfaces & Regras de Negócio                   │
│                                                             │
│  • UseCases (Interfaces)     • Entities                     │
│  • Repositories (Interfaces) • Failures                     │
│  • Enums                     • Value Objects                │
└─────────────────────┬───────────────────────────────────────┘
                      │ implements
┌─────────────────────▼───────────────────────────────────────┐
│                INFRASTRUCTURE LAYER                         │
│            🔧 Implementações & Coordenação                  │
│                                                             │
│  • UseCases (Implementações)                                │
│  • Repositories (Implementações)                            │
│  • DataSources (Interfaces)                                 │
│  • Models                                                   │
└─────────────────────┬───────────────────────────────────────┘
                      │ calls
┌─────────────────────▼───────────────────────────────────────┐
│                     DATA LAYER                              │
│             💾 Comunicação Externa Real                     │
│                                                             │
│  • DataSources (Implementações)                             │
│  • External APIs                                            │
│  • Database Access                                          │
│  • Local Storage                                            │
└─────────────────────────────────────────────────────────────┘
```

## � Status da Documentação

| Componente | Status | Foco Atual |
|------------|--------|------------|
| **🎯 Domain - Abstrações** | ✅ | **O QUE** fazer (contratos puros) |
| └─ UseCase Interfaces | ✅ | Contratos de regras de negócio |
| └─ Repository Interfaces | ✅ | Contratos de acesso aos dados |
| └─ Entities | ✅ | Objetos de negócio com validações |
| └─ Enums | ✅ | Valores constantes e tipagem forte |
| └─ Failures | ✅ | Tipos de erro específicos do domínio |
| **🔧 Infra - Implementações** | ✅ | **COMO** fazer (coordenação) |
| └─ DataSource Interfaces | ✅ | Contratos de fontes externas |
| └─ Models | ✅ | Adaptadores de dados |
| └─ UseCase Implementations | ✅ | Implementações de negócio |
| └─ Repository Implementations | ✅ | Coordenação de dados |
| **💾 Data - Comunicação** | ✅ | Comunicação externa real |
| └─ DataSource Implementations | ✅ | Comunicação real com APIs/BD |
| **🎨 Presentation - Interface** | ✅ | Estado da UI e coordenação |
| └─ Controllers | ✅ Completo | Alta | **ValueNotifier + Either pattern** |
| └─ Pages | ✅ Completo | Alta | **Composição de interface e navegação** |
| └─ Routes | ✅ Completo | Alta | **Navegação tipificada e modular** |
| └─ Modules | ✅ Completo | Alta | **DI e configuração de rotas** |
| └─ Widgets | ✅ Completo | Alta | **Componentização e reutilização** |

### 🎯 Principais Melhorias Aplicadas:
- ✅ **Clareza nos Contratos**: Interfaces definem claramente O QUE fazer
- ✅ **Princípios SOLID**: DIP, ISP aplicados rigorosamente  
- ✅ **Tipagem Forte**: Either pattern obrigatório para todas as operações
- ✅ **Zero Dependências**: Interfaces dependem apenas de abstrações
- ✅ **Documentação Rica**: Contratos bem documentados com exemplos reais
- ✅ **Implementações Completas**: Todas as camadas documentadas com padrões

---

## 🏗️ Visão Geral da Arquitetura

📖 **[Responsabilidades e Abstrações](architecture-overview.md)**
- Explicação detalhada de cada camada
- Por que usar abstrações em cada nível
- Exemplos práticos de implementação
- Fluxo de dependências e inversão de controle
- Benefícios de testabilidade e manutenibilidade

---

## 🎯 Princípios Fundamentais das Abstrações (Interfaces I_)

> **As interfaces I_ definem regras de negócio: O QUE deve ser feito, nunca COMO implementar**

### 🔒 Regras INVIOLÁVEIS para Interfaces

#### 1. **Zero Dependências Externas**
```dart
// ✅ APENAS imports do domain
import '../entities/user_entity.dart';
import '../failures/i_user_failures.dart';

// ❌ NUNCA importar implementações
// import 'package:dio/dio.dart';
// import '../infra/models/user_model.dart';
```

#### 2. **Tipagem Forte Obrigatória**
```dart
// ✅ Sempre Either<Failure, Success>
Future<Either<IUserFailure, UserEntity>> getUser();

// ❌ NUNCA retornos sem tratamento de erro
// Future<UserEntity> getUser();
// UserEntity getUser();
```

#### 3. **Dependência de Abstrações (SOLID DIP)**
```dart
// ✅ Depende de interfaces/entities (abstrações)
abstract class IUserUsecase {
  Future<Either<IUserFailure, UserEntity>> getUser();
}

// ❌ NUNCA depender de implementações concretas
// abstract class IUserUsecase {
//   final UserRepository repository; // implementação concreta
// }
```

#### 4. **Documentação Clara do Contrato**
```dart
/// Interface que define O QUE deve ser feito com usuários
/// 
/// Estabelece contratos para:
/// - Operações CRUD básicas
/// - Regras de negócio aplicáveis
/// - Validações obrigatórias
abstract class IUserRepository {
  /// Obtém usuário por ID específico
  /// 
  /// [id] deve ser um identificador válido e não vazio
  /// 
  /// Retorna [Right] com usuário encontrado ou
  /// [Left] com erro específico se não encontrado
  Future<Either<IUserFailure, UserEntity>> getUserById({
    required String id,
  });
}
```

---

## 📂 Estrutura de Camadas

### 🎯 1. Domain Layer (Regras de Negócio Puras)

> **Responsabilidade**: Definir **O QUE** deve ser feito através de contratos e regras de negócio, sem dependências externas ou detalhes de implementação

#### [📖 UseCase Interfaces (Contratos de Negócio)](./domain/i_usecases.md) ✅
- **O que é**: Interfaces que definem **QUAIS** operações de negócio devem existir
- **Responsabilidade**: Estabelecer contratos dos casos de uso sem implementação
- **Princípios**: Tipagem forte, Either pattern, sem dependências externas
- **Exemplo**: `IUserUsecase`, `IProductUsecase`

#### [📖 Repository Interfaces (Contratos de Dados)](./domain/i_repositories.md) ✅
- **O que é**: Interfaces que definem **COMO** acessar dados (contrato)
- **Responsabilidade**: Estabelecer operações de persistência sem implementação
- **Princípios**: Either pattern, tipagem forte, abstrações SOLID
- **Exemplo**: `IUserRepository`, `IProductRepository`

#### [📖 Entities (Objetos de Negócio)](./domain/entities.md) ✅
- **O que é**: Objetos de negócio puros com regras de domínio
- **Responsabilidade**: Carregar dados, validações básicas e regras de negócio
- **Características**: `const` constructors, imutabilidade, validações com `assert`
- **Exemplo**: `UserEntity`, `ProductEntity`

#### [📖 Enums (Valores Constantes)](./domain/enums.md) ✅
- **O que é**: Valores constantes e bem definidos do domínio
- **Responsabilidade**: Tipagem forte para estados, tipos e categorias
- **Características**: Serialização consistente, nomes legíveis, validação automática
- **Exemplo**: `UserGenderType`, `AuthProviderType`, `OrderStatusType`

#### [📖 Failures (Tipos de Erro)](./domain/failures.md) ✅
- **O que é**: Definições de erros específicos do domínio
- **Responsabilidade**: Tipificar falhas de negócio para Either pattern
- **Princípios**: Herança de ICustomFailure, mensagens descritivas, granularidade
- **Exemplo**: `IUserFailure`, `UserNotFoundError`, `UserServerError`

---

### 🔧 2. Infrastructure Layer (Implementações e Coordenação)

> **Responsabilidade**: Implementar **COMO** fazer o que foi definido no Domain, coordenando múltiplas fontes de dados

#### [📖 DataSource Interfaces (Contratos de Fontes Externas)](./infra/i_datasources.md) ✅
- **O que é**: Interfaces que definem **COMO** comunicar com fontes externas (contrato)
- **Responsabilidade**: Estabelecer protocolos de comunicação sem implementação
- **Princípios**: Either pattern, tipagem forte, protocolos bem definidos
- **Exemplo**: `IUserDatasource`, `IProductDatasource`

#### [📖 UseCase Implementations (Coordenação de Negócio)](./infra/implementations/usecases.md) ✅
- **O que é**: Implementação real dos casos de uso
- **Responsabilidade**: Aplicar regras de negócio e coordenar repositories
- **Princípios**: Orquestração, validações, tratamento de erros
- **Exemplo**: `UserUsecase extends IUserUsecase`

#### [📖 Repository Implementations (Coordenação de Dados)](./infra/implementations/repositories.md) ✅
- **O que é**: Implementação real dos repositórios
- **Responsabilidade**: Coordenar datasources, cache, fallback
- **Princípios**: Transformação Model↔Entity, tratamento de erros técnicos
- **Exemplo**: `UserRepository extends IUserRepository`

---

### 💾 3. Data Layer (Comunicação Externa Real)

> **Responsabilidade**: Executar **COMO** comunicar realmente com fontes externas (APIs, DB, cache)

#### [📖 DataSource Implementations (Comunicação Real)](./data/datasources.md) ✅
- **O que é**: Implementação real de comunicação com fontes externas
- **Responsabilidade**: Executar protocolos HTTP, DB, cache, serialização
- **Princípios**: I/O real, performance, protocolo específico
- **Exemplo**: `UserDatasource extends IUserDatasource`

#### [📖 Models (Adaptadores de Dados)](./infra/models.md) ✅
- **O que é**: Adaptadores entre entities e dados externos
- **Responsabilidade**: Serialização/deserialização com tratamento robusto de dados
- **Características**: `const` constructors, EquatableMixin, tratamento de nulos
- **Exemplo**: `UserModel extends UserEntity`

---

### 🎨 4. Presentation Layer (Interface e Estado)

> **Responsabilidade**: Gerenciar **estado da UI** e **coordenar** operações de negócio com a camada Domain

#### [📖 Controllers (Gerenciamento de Estado)](./presentation/controllers.md) ✅
- **O que é**: Gerenciadores de estado baseados em ValueNotifier com Either pattern
- **Responsabilidade**: State management reativo, coordenação de UseCases, estados auxiliares
- **Características**: ValueNotifier integration, auto loading/error, callback injection
- **Exemplo**: `AppController`, `SessionController`, `LoginController`

#### [📖 Pages (Composição de Interface)](./presentation/pages.md) ✅
- **O que é**: Páginas que compõem interface e orquestram Controllers
- **Responsabilidade**: Composição da UI, navegação, tratamento de estados da interface
- **Características**: CustomListenableBuilder, args tipificados, lifecycle management
- **Exemplo**: `EnrollmentsPage`, `CreateEnrollmentPage`, `EnrollmentDetailsPage`

#### [📖 Routes (Navegação e Hierarquia)](./presentation/routes.md) ✅
- **O que é**: Definição de navegação e hierarquia de rotas da aplicação
- **Responsabilidade**: Estruturar navegação, organizar módulos, integrar pacotes
- **Características**: Rotas internas simples, rotas de pacotes configuráveis, singleton pattern
- **Exemplo**: `HomeRoutes`, `FunnelRoutes`, `CandidatesRoutes`

#### [📖 Widgets (Componentização e Reutilização)](./presentation/widgets.md) ✅
- **O que é**: Componentes reutilizáveis e específicos para organização da interface
- **Responsabilidade**: Componentizar UI, promover reutilização, controlar complexity das pages
- **Características**: Widgets globais vs específicos, part/part of, widgets privados, máximo 300 linhas por page
- **Exemplo**: `CustomButton`, `AddressFormWidget`, `_UserListItem`

#### [📖 Modules (Injeção de Dependências)](./presentation/modules.md) ✅
- **O que é**: Configuração de DI container e roteamento usando Flutter Modular
- **Responsabilidade**: Injetar dependências, definir rotas, importar módulos, exportar services
- **Características**: Binds organizados, routes estruturadas, imports auxiliares, exportedBinds
- **Exemplo**: `MainModule`, `FunnelModule`, `AuthModule`

---

## 🔄 Fluxo de Comunicação

### 📊 Hierarquia de Dependências

```
Presentation ──calls──> Infrastructure UseCase
                               │
Infrastructure UseCase ──uses──> Domain Repository Interface
                               │
Infrastructure Repository ──implements──> Domain Repository Interface
                               │
Infrastructure Repository ──calls──> Infrastructure DataSource Interface
                               │
Data DataSource ──implements──> Infrastructure DataSource Interface
                               │
Data DataSource ──communicates──> External APIs/DB
```

### 🎯 Responsabilidades por Camada

| Camada | O que FAZ | O que NÃO FAZ |
|--------|-----------|---------------|
| **Domain** | Define contratos e regras | Não implementa nem conhece infraestrutura |
| **Infrastructure** | Implementa contratos, coordena fluxo | Não faz comunicação externa direta |
| **Data** | Comunicação externa real | Não contém regras de negócio |
| **Presentation** | Gerencia estado UI, coordena UseCases | Não contém regras de negócio nem comunicação direta |

---

## 🚀 Guia de Implementação

### 📋 Para criar uma nova feature:

1. **[Comece pelo Domain](./domain/)** - Defina entities, failures e interfaces
2. **[Implemente na Infrastructure](./infra/)** - Crie as implementações e coordenação  
3. **[Finalize no Data](./data/)** - Implemente a comunicação externa
4. **[Crie a Presentation](./presentation/)** - Implemente controllers e pages para a UI

### 🔍 Para debuggar problemas:

1. **Domain**: Valide regras de negócio e contratos
2. **Infrastructure**: Verifique coordenação entre camadas
3. **Data**: Analise comunicação externa e parsing
4. **Presentation**: Verifique estado da UI e binding com controllers

---

## 📖 Documentações Auxiliares

### 🎨 Padrões e Convenções
- [📖 Naming Conventions](./conventions/naming.md)
- [📖 Code Style Guide](./conventions/code-style.md)
- [📖 Error Handling Patterns](./conventions/error-handling.md)

### 🧪 Testing Guidelines
- [📖 Unit Testing Strategy](./testing/unit-tests.md)
- [📖 Integration Testing](./testing/integration-tests.md)
- [📖 Mock Strategies](./testing/mocking.md)

### 🔧 Setup e Configuração
- [📖 Project Setup](./setup/project-setup.md)
- [📖 Dependency Injection](./setup/dependency-injection.md)
- [📖 Environment Configuration](./setup/environment.md)

---

## 🎯 Status das Documentações

| Documento | Status | Prioridade | Observações |
|-----------|--------|------------|-------------|
| **Domain - Abstrações** | | | |
| └─ UseCases Interfaces | ✅ Completo | Alta | Contratos bem definidos |
| └─ Repository Interfaces | ✅ Completo | Alta | Contratos bem definidos |
| └─ Entities | ✅ Completo | Alta | Objetos de negócio puros |
| └─ Enums | ✅ Completo | Alta | **Recém criado com exemplos reais** |
| └─ Failures | ✅ Completo | Alta | Tipos de erro específicos |
| **Infrastructure - Implementações** | | | |
| └─ UseCase Implementations | ✅ Completo | Alta | Orquestração de negócio |
| └─ Repository Implementations | ✅ Completo | Alta | Coordenação de dados |
| └─ DataSource Interfaces | ✅ Completo | Alta | Contratos de comunicação |
| └─ Models | ✅ Completo | Média | Adaptadores de dados |
| **Data - Comunicação** | | | |
| └─ DataSource Implementations | ✅ Completo | Alta | Comunicação externa real |
| **Presentation - Interface** | | | |
| └─ Controllers | ✅ Completo | Alta | **ValueNotifier + Either pattern** |
| └─ Pages | ✅ Completo | Alta | **Composição de interface e navegação** |
| └─ Routes | ✅ Completo | Alta | **Navegação tipificada e modular** |
| └─ Modules | ✅ Completo | Alta | **DI e configuração de rotas** |
| └─ Widgets | ✅ Completo | Alta | **Componentização e reutilização** |

---

## 🔄 Atualizações Recentes

### ✅ Suite Completa de Documentação - Setembro 2025
- **Documentação Clean Architecture Completa** criada do zero
- **Todas as 4 camadas documentadas** com exemplos reais e práticos:
  - **Domain Layer**: Interfaces, Entities, Enums, Failures com princípios SOLID
  - **Infrastructure Layer**: Implementações de UseCases, Repositories, DataSources
  - **Data Layer**: Comunicação externa real com APIs e databases
  - **Presentation Layer**: Controllers, Pages, Routes, Modules e Widgets com ValueNotifier + Either pattern
- **Templates e Checklists** para cada tipo de componente
- **Padrões SOLID rigorosamente aplicados** em todos os exemplos
- **Either pattern obrigatório** para tratamento de erros
- **Exemplos reais** baseados em UserEntity, AuthProviderType, AppController, etc.
- **Widgets componentizados** com estratégias de reutilização e organização

---

## 🆘 Precisa de Ajuda?

1. **📖 Leia a documentação** da camada correspondente primeiro
2. **🏗️ Consulte o [Guia de Arquitetura](./architecture-overview.md)** para entender responsabilidades
3. **🔍 Veja exemplos** nos templates de cada componente
4. **❓ Dúvidas sobre responsabilidades?** Consulte o fluxo de dependências acima
5. **🐛 Problemas na implementação?** Verifique princípios SOLID nos contratos
6. **🎯 Precisa implementar algo novo?** Use os templates e checklists atualizados

---

*Este índice é o ponto de partida para entender a Clean Architecture. Toda a documentação foi criada com exemplos reais e princípios SOLID rigorosamente aplicados. Consulte sempre o [architecture-overview.md](./architecture-overview.md) para entender as responsabilidades específicas de cada camada.*
