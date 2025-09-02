# Clean Architecture - Cogna Resale
## 📖 Índice Principal de Documentação

### 🏗️ Visão Geral da Arquitetura

A Clean Architecture está organizada em **3 camadas principais** com responsabilidades bem definidas:

```
┌───────────────────────────────────────────## 📊 Status da Documentação

| Componente | Status | Foco Atual |
|------------|--------|------------|
| **🎯 Domain - Abstrações** | ✅ | **O QUE** fazer (contratos puros) |
| └─ UseCase Interfaces | ✅ | Contratos de regras de negócio |
| └─ Repository Interfaces | ✅ | Contratos de acesso aos dados |
| └─ Entities | ✅ | Objetos de negócio com validações |
| **🔧 Infra - Implementações** | ⚡ | **COMO** fazer (coordenação) |
| └─ DataSource Interfaces | ✅ | Contratos de fontes externas |
| └─ Models | ✅ | Adaptadores de dados |
| └─ UseCase Implementations | 🔄 | Próximo: implementações de negócio |
| └─ Repository Implementations | 🔄 | Próximo: coordenação de dados |
| **💾 Data - Comunicação** | 🔄 | Comunicação externa real |

### 🎯 Principais Melhorias Aplicadas:
- ✅ **Clareza nos Contratos**: Interfaces definem claramente O QUE fazer
- ✅ **Princípios SOLID**: DIP, ISP aplicados rigorosamente  
- ✅ **Tipagem Forte**: Either pattern obrigatório para todas as operações
- ✅ **Zero Dependências**: Interfaces dependem apenas de abstrações
- ✅ **Documentação Rica**: Contratos bem documentados com exemplos reais

---────┐
│                    PRESENTATION LAYER                       │
│                  (Pages, Controllers, UI)                   │
└─────────────────────┬───────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────┐
│                    DOMAIN LAYER                            │
│         📋 Interfaces & Regras de Negócio                   │
│                                                             │
│  • UseCases (Interfaces)     • Entities                    │
│  • Repositories (Interfaces) • Failures                    │
│  • Enums                     • Value Objects               │
└─────────────────────┬───────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────┐
│                INFRASTRUCTURE LAYER                        │
│            🔧 Implementações & Coordenação                  │
│                                                             │
│  • UseCases (Implementações)                               │
│  • Repositories (Implementações)                           │
│  • DataSources (Interfaces)                                │
│  • Models                                                  │
└─────────────────────┬───────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────┐
│                     DATA LAYER                             │
│             💾 Comunicação Externa                          │
│                                                             │
│  • DataSources (Implementações)                            │
│  • External APIs                                           │
│  • Database Access                                         │
│  • Local Storage                                           │
└─────────────────────────────────────────────────────────────┘
```

---

## 📚 Documentação Clean Architecture

Esta documentação fornece guias práticos e templates para implementar Clean Architecture em projetos Dart/Flutter, com foco em padrões de código, responsabilidades por camadas e melhores práticas.

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

#### [📖 Failures (Tipos de Erro)](./domain/failures.md) 🔄
- **O que é**: Definições de erros específicos do domínio
- **Responsabilidade**: Tipificar falhas de negócio para Either pattern
- **Exemplo**: `IUserFailure`, `UserNotFoundError`

---

### 🔧 2. Infrastructure Layer (Implementações e Coordenação)

> **Responsabilidade**: Implementar **COMO** fazer o que foi definido no Domain, coordenando múltiplas fontes de dados

#### [📖 DataSource Interfaces (Contratos de Fontes Externas)](./infra/i_datasources.md) ✅
- **O que é**: Interfaces que definem **COMO** comunicar com fontes externas (contrato)
- **Responsabilidade**: Estabelecer protocolos de comunicação sem implementação
- **Princípios**: Either pattern, tipagem forte, protocolos bem definidos
- **Exemplo**: `IUserDatasource`, `IProductDatasource`

#### [📖 Models (Adaptadores de Dados)](./infra/models.md) ✅
- **O que é**: Implementação real dos casos de uso
- **Responsabilidade**: Aplicar regras de negócio e coordenar repositories
- **Exemplo**: `UserUsecase extends IUserUsecase`

#### [📖 Repositories (Implementações)](./infra/repositories.md)
- **O que é**: Implementação real dos repositórios
- **Responsabilidade**: Coordenar datasources, cache, fallback
- **Exemplo**: `UserRepository extends IUserRepository`

#### [📖 DataSources (Interfaces)](./infra/i_datasources.md)
- **O que é**: Contratos para comunicação com dados externos
- **Responsabilidade**: Definir protocolos de acesso a dados
- **Exemplo**: `IUserDatasource`, `IProductDatasource`

#### [📖 Models](./infra/models.md)
- **O que é**: Adaptadores entre entities e dados externos
- **Responsabilidade**: Serialização/deserialização com tratamento robusto de dados
- **Características**: `const` constructors, EquatableMixin, tratamento de nulos
- **Exemplo**: `UserModel extends UserEntity`

---

### 💾 3. Data Layer (Comunicação Externa)

> **Responsabilidade**: Implementar comunicação real com APIs, databases e armazenamento

#### [📖 DataSources (Implementações)](./data/datasources.md)
- **O que é**: Implementação real da comunicação externa
- **Responsabilidade**: HTTP requests, database queries, file I/O
- **Exemplo**: `UserDatasource extends IUserDatasource`

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

---

## 🚀 Guia de Implementação

### 📋 Para criar uma nova feature:

1. **[Comece pelo Domain](./domain/)** - Defina entities, failures e interfaces
2. **[Implemente na Infrastructure](./infra/)** - Crie as implementações e coordenação  
3. **[Finalize no Data](./data/)** - Implemente a comunicação externa

### 🔍 Para debuggar problemas:

1. **Domain**: Valide regras de negócio e contratos
2. **Infrastructure**: Verifique coordenação entre camadas
3. **Data**: Analise comunicação externa e parsing

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
| Domain/UseCases | ✅ Completo | Alta | |
| Domain/Repositories | ✅ Completo | Alta | |
| Domain/Entities | ✅ **Atualizado** | Alta | **Aplicação de boas práticas** |
| Domain/Failures | 🔜 Pendente | Alta | |
| Infra/UseCases | ✅ Completo | Alta | |
| Infra/Repositories | ✅ Completo | Alta |  |
| Infra/DataSources | ✅ Completo | Alta | |
| Infra/Models | ✅ **Atualizado** | Média | **Aplicação de boas práticas** |
| Data/DataSources | 🔧 Revisando | Alta | |

---

## 🔄 Atualizações Recentes

### ✅ Entities e Models - Setembro 2025
- **Documentações atualizadas** com aplicação correta dos conceitos de Clean Architecture
- **Implementação de boas práticas** para cada camada:
  - **Domain Entities**: `const` constructors, validações com `assert`, regras de negócio
  - **Infrastructure Models**: Tratamento robusto de dados externos, serialização segura
- **Templates modernizados** com padrões atuais do Dart/Flutter
- **Exemplos práticos** demonstrando implementação real dos conceitos

---

## 🆘 Precisa de Ajuda?

1. **📖 Leia a documentação** da camada correspondente
2. **🔍 Veja exemplos** nos templates de Entities e Models 
3. **❓ Dúvidas sobre responsabilidades?** Consulte a hierarquia acima
4. **🐛 Problemas na implementação?** Verifique o fluxo de comunicação
5. **🎯 Precisa implementar algo novo?** Use os templates atualizados

---

*Este índice é o ponto de partida para entender a Clean Architecture. As documentações de **Entities** e **Models** foram recentemente atualizadas com as melhores práticas e conceitos modernos. Sempre consulte este documento antes de navegar para documentações específicas.*
