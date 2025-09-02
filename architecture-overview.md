# Responsabilidades por Camadas - Clean Architecture

## 📋 Visão Geral

Este documento define **responsabilidades específicas** de cada camada na Clean Architecture e explica **por que utilizamos abstrações** para manter a separação de responsabilidades e garantir flexibilidade e testabilidade.

### 🎯 Objetivos das Abstrações

- **Inversão de Dependências (DIP)**: Camadas superiores dependem de abstrações, não implementações
- **Testabilidade**: Facilita criação de mocks e testes unitários isolados
- **Flexibilidade**: Permite múltiplas implementações sem quebrar contratos
- **Manutenibilidade**: Mudanças em uma camada não propagam para outras
- **Princípio Aberto/Fechado (OCP)**: Aberto para extensão, fechado para modificação
- **Interface Segregation (ISP)**: Interfaces específicas e coesas

### 🔑 Conceitos Fundamentais

> **Interfaces definem O QUE fazer (contratos), Implementações definem COMO fazer (execução)**

**Domain**: Define regras de negócio e contratos (O QUE)
**Infrastructure**: Implementa coordenação e orquestração (COMO coordenar)  
**Data**: Executa comunicação externa real (COMO comunicar)

---

## 🏗️ Camadas e Suas Responsabilidades

### 🎯 Domain Layer (Núcleo do Negócio)

> **Filosofia**: "O que o sistema deve fazer, não como fazer"

#### 📋 Responsabilidades

**✅ O que a camada Domain FAZ:**
- Define **regras de negócio puras** através de Entities
- Estabelece **contratos de operações** através de Interfaces
- Especifica **tipos de erro de negócio** através de Failures  
- Define **valores constantes do domínio** através de Enums
- Especifica **casos de uso que devem existir** através de IUseCases
- Estabelece **contratos de acesso a dados** através de IRepositories

**❌ O que a camada Domain NÃO FAZ:**
- Não implementa comunicação externa (APIs, DB, cache)
- Não contém lógica de apresentação, UI ou formatação
- Não depende de frameworks, bibliotecas ou tecnologias específicas
- Não implementa protocolos de comunicação (HTTP, gRPC, etc.)
- Não define detalhes de persistência ou serialização
- Não contém configurações de ambiente ou infraestrutura

#### 🔍 Por que usar Interfaces no Domain?

```dart
// ✅ Interface define CONTRATO sem implementação
abstract class IUserRepository {
  Future<Either<IUserFailure, UserEntity>> getLoggedUser();
}

// ❌ Implementação concreta criaria dependência
class UserRepositoryImpl {
  // Dependeria de tecnologias específicas
}
```

**Vantagens das Interfaces:**
1. **Testabilidade**: Facilita criação de mocks
2. **Flexibilidade**: Múltiplas implementações possíveis
3. **Desacoplamento**: Domain não conhece infraestrutura
4. **Princípios SOLID**: Inversão de dependências

---

### 🔧 Infrastructure Layer (Orquestração)

> **Filosofia**: "Como implementar o que foi definido no Domain"

#### 📋 Responsabilidades

**✅ O que a camada Infrastructure FAZ:**
- **Implementa contratos do Domain** (IUseCases → UseCases, IRepositories → Repositories)
- **Coordena múltiplas fontes de dados** (cache + API + fallback)
- **Aplica regras de negócio complexas** orquestrando repositories
- **Transforma dados externos** (Models ↔ Entities) 
- **Define contratos de fontes externas** através de IDataSources
- **Trata erros técnicos** transformando em erros de domínio
- **Implementa estratégias** de cache, retry, circuit breaker

**❌ O que a camada Infrastructure NÃO FAZ:**
- Não executa comunicação externa direta (fica no Data)
- Não contém regras de negócio puras (ficam no Domain)
- Não define contratos de negócio (apenas implementa os existentes)
- Não contém lógica de apresentação ou formatação de UI
- Não implementa protocolos específicos (HTTP, SQL, etc.)

#### 🔍 Por que Abstrações na Infrastructure?

```dart
// ✅ UseCase implementa interface do Domain
class UserUsecase extends IUserUsecase {
  UserUsecase({required this.repository});
  
  final IUserRepository repository; // Depende da abstração
  
  @override
  Future<Either<IUserFailure, UserEntity>> getLoggedUser() {
    // Implementa regra de negócio usando repository
    return repository.getLoggedUser();
  }
}

// ✅ Repository implementa interface e coordena datasources
class UserRepository extends IUserRepository {
  UserRepository({required this.datasource});
  
  final IUserDatasource datasource; // Depende da abstração
  
  @override
  Future<Either<IUserFailure, UserEntity>> getLoggedUser() async {
    final result = await datasource.getLoggedUser();
    return result.fold(
      (error) => Left(UserServerError(error.message)),
      (response) => Right(UserModel.fromMap(response.data).toEntity),
    );
  }
}
```

**Vantagens:**
1. **Coordenação**: Orquestra múltiplas fontes de dados
2. **Transformação**: Converte dados externos para entities
3. **Tratamento de Erro**: Transforma erros técnicos em erros de domínio
4. **Cache/Fallback**: Implementa estratégias de dados

---

### 💾 Data Layer (Comunicação Externa)

> **Filosofia**: "Como se comunicar com o mundo externo"

#### 📋 Responsabilidades

**✅ O que a camada Data FAZ:**
- **Implementa comunicação externa real** com APIs, databases, cache
- **Executa protocolos específicos** (HTTP, GraphQL, SQL, NoSQL, gRPC)
- **Serializa/Deserializa dados** para formato de transporte/storage
- **Gerencia conexões** físicas, autenticação e autorização
- **Implementa estratégias de rede** (retry, timeout, circuit breaker)
- **Otimiza performance** (connection pooling, batch requests)
- **Implementa contratos IDataSources** definidos na Infrastructure

**❌ O que a camada Data NÃO FAZ:**
- Não contém regras de negócio ou validações de domínio
- Não define contratos ou interfaces (apenas implementa)
- Não transforma dados em Entities (Model → Entity fica na Infra)
- Não coordena múltiplas operações de negócio
- Não contém lógica de cache ou fallback (coordenação fica na Infra)

#### 🔍 Por que Abstrações no Data?

```dart
// ✅ DataSource implementa interface da Infrastructure
class UserDatasource extends IUserDatasource {
  UserDatasource({required this.httpClient});
  
  final HttpClient httpClient;
  
  @override
  Future<Either<HttpErrorResponse, HttpDriverResponse>> getLoggedUser() async {
    try {
      final response = await httpClient.get('/api/users/me');
      return Right(HttpDriverResponse(data: response.data));
    } catch (e) {
      return Left(HttpErrorResponse(message: e.toString()));
    }
  }
}
```

---

## 🔄 Fluxo de Dependências

### 📊 Direção das Dependências (SOLID DIP)

```
┌─────────────────────────────────────────────────────────────┐
│                    PRESENTATION                             │
│                   (Controllers, Pages)                     │
│                          │ calls                            │
│                          ▼                                 │
└─────────────────────────────────────────────────────────────┘
                             │
┌─────────────────────────────────────────────────────────────┐
│                     DOMAIN                                 │
│                   (Contracts)                              │
│                                                             │
│    IUserUsecase ◄─┐  ┌─► IUserRepository ◄─┐              │
│                   │  │                     │              │
└───────────────────┼──┼─────────────────────┼───────────────┘
                    │  │                     │                
┌───────────────────┼──┼─────────────────────┼───────────────┐
│                   │  │  INFRASTRUCTURE     │               │
│                   │  │   (Implementations) │               │
│                   │  │                     │               │
│    UserUsecase ───┘  │  UserRepository ────┘               │
│         │             │         │                          │
│         │             │         ▼                          │
│         │             │  IUserDatasource ◄─┐               │
└─────────┼─────────────┼─────────────────────┼───────────────┘
          │             │                     │                
┌─────────┼─────────────┼─────────────────────┼───────────────┐
│         │             │       DATA          │               │
│         │             │  (External I/O)     │               │
│         │             │                     │               │
│         └─────────────┼──► UserDatasource ──┘               │
│                       │           │                        │
│                       └───────────┼─► HTTP/API/DB          │
└─────────────────────────────────────┼─────────────────────────┘
                                      ▼
                                External Systems
```

**🔑 Pontos Chave:**
- **Setas apontam para abstrações** (interfaces), nunca para implementações
- **Infrastructure depende do Domain**, nunca o contrário
- **Data implementa contratos** da Infrastructure, mas não os define
- **Presentation usa abstrações** do Domain através da Infrastructure

### 🎯 Princípio da Inversão de Dependências

**Antes (Dependência Direta):**
```dart
// ❌ UseCase depende de implementação concreta
class UserUsecase {
  final UserDatasource datasource; // Dependência direta
  
  Future<UserEntity> getUser() {
    // Violação: UseCase conhece detalhes de implementação
  }
}
```

**Depois (Inversão de Dependências):**
```dart
// ✅ UseCase depende de abstração
class UserUsecase extends IUserUsecase {
  final IUserRepository repository; // Dependência invertida
  
  Future<Either<IUserFailure, UserEntity>> getUser() {
    // UseCase não conhece implementação
    return repository.getLoggedUser();
  }
}
```

---

## � Benefícios das Abstrações por Camada

### 🎯 Domain Layer - Interfaces

| Benefício | Descrição | Exemplo |
|-----------|-----------|---------|
| **Testabilidade** | Facilita criação de mocks | `MockUserRepository implements IUserRepository` |
| **Flexibilidade** | Múltiplas implementações | `DatabaseUserRepo`, `ApiUserRepo` |
| **Estabilidade** | Contratos estáveis | Interface não muda com implementação |
| **Documentação** | Contratos são autodocumentados | Métodos definem "o que" fazer |

### 🔧 Infrastructure Layer - Implementações

| Benefício | Descrição | Exemplo |
|-----------|-----------|---------|
| **Coordenação** | Orquestra múltiplas fontes | Cache + API + Fallback |
| **Transformação** | Adapta dados externos | `UserModel.fromMap().toEntity` |
| **Tratamento** | Converte erros técnicos | `HttpError` → `UserServerError` |
| **Estratégia** | Implementa políticas | Retry, Circuit Breaker |

### 💾 Data Layer - Comunicação

| Benefício | Descrição | Exemplo |
|-----------|-----------|---------|
| **Especialização** | Foca em protocolo específico | HTTP, GraphQL, gRPC |
| **Performance** | Otimizações de rede | Connection pooling, cache |
| **Tecnologia** | Usa bibliotecas específicas | Dio, Retrofit, SQLite |
| **Protocolo** | Implementa detalhes técnicos | Headers, Auth, Serialização |

---

## � Exemplos Práticos de Responsabilidades

### 🎯 Cenário: Buscar Usuário Logado

**1. Domain (IUserUsecase):**
```dart
/// Define O QUE deve ser feito
abstract class IUserUsecase {
  /// Obtém usuário logado aplicando regras de negócio
  Future<Either<IUserFailure, UserEntity>> getLoggedUser();
}
```

**2. Infrastructure (UserUsecase):**
```dart
/// Implementa COMO aplicar regras de negócio
class UserUsecase extends IUserUsecase {
  @override
  Future<Either<IUserFailure, UserEntity>> getLoggedUser() async {
    // 1. Coordena busca nos dados
    final result = await repository.getLoggedUser();
    
    // 2. Aplica regras de negócio
    return result.fold(
      (error) => Left(error),
      (user) {
        // Regra: usuário deve estar ativo
        if (!user.isActive) {
          return Left(UserInactiveError());
        }
        return Right(user);
      },
    );
  }
}
```

**3. Infrastructure (UserRepository):**
```dart
/// Implementa COMO coordenar fontes de dados
class UserRepository extends IUserRepository {
  @override
  Future<Either<IUserFailure, UserEntity>> getLoggedUser() async {
    // 1. Tenta cache primeiro
    final cacheResult = await cacheDatasource.getLoggedUser();
    if (cacheResult.isRight()) return cacheResult;
    
    // 2. Busca na API
    final apiResult = await apiDatasource.getLoggedUser();
    
    // 3. Transforma dados e trata erros
    return apiResult.fold(
      (error) => Left(UserServerError(error.message)),
      (response) {
        final user = UserModel.fromMap(response.data).toEntity;
        // 4. Salva no cache para próxima vez
        cacheDatasource.saveUser(user);
        return Right(user);
      },
    );
  }
}
```

**4. Data (UserDatasource):**
```dart
/// Implementa COMO se comunicar com fontes externas
class UserDatasource extends IUserDatasource {
  @override
  Future<Either<HttpErrorResponse, HttpDriverResponse>> getLoggedUser() async {
    try {
      // Comunicação real com API
      final response = await httpClient.get(
        '/api/users/me',
        headers: {'Authorization': 'Bearer $token'},
      );
      
      return Right(HttpDriverResponse(data: response.data));
    } on DioException catch (e) {
      return Left(HttpErrorResponse(
        message: e.message ?? 'Erro de conexão',
        statusCode: e.response?.statusCode,
      ));
    }
  }
}
```

---

## 📋 Checklist de Responsabilidades

### ✅ Domain Layer
- [ ] Define apenas **CONTRATOS** (interfaces)
- [ ] Contém **regras de negócio puras**
- [ ] **Não depende** de tecnologias externas
- [ ] Entities com **validações básicas**
- [ ] Failures **específicos do domínio**

### ✅ Infrastructure Layer  
- [ ] **Implementa** contratos do Domain
- [ ] **Coordena** múltiplas fontes de dados
- [ ] **Transforma** dados externos em entities
- [ ] **Aplica** regras de negócio complexas
- [ ] **Trata** erros técnicos → erros de domínio

### ✅ Data Layer
- [ ] **Implementa** comunicação real externa
- [ ] **Executa** protocolos específicos
- [ ] **Serializa/Deserializa** dados
- [ ] **Gerencia** conexões e autenticação
- [ ] **Otimiza** performance de rede

---

## 🎯 Resumo dos Benefícios

### ✅ Testabilidade
- **Mocks fáceis**: Interfaces permitem criar implementações falsas
- **Testes isolados**: Cada camada pode ser testada independentemente
- **TDD natural**: Interfaces primeiro, implementação depois

### ✅ Manutenibilidade
- **Mudanças localizadas**: Alterações afetam apenas uma camada
- **Código limpo**: Responsabilidades bem definidas
- **Documentação viva**: Interfaces servem como contratos

### ✅ Escalabilidade
- **Múltiplas implementações**: Interface única, várias implementações
- **Plugins facilmente**: Trocar datasources sem afetar lógica
- **Features independentes**: Cada use case é isolado

### ✅ Flexibilidade
- **Adaptação rápida**: Mudanças de requisitos localizadas
- **Tecnologias intercambiáveis**: Database, HTTP client, cache
- **Ambientes diferentes**: Dev, staging, prod com implementações distintas

---

*Esta documentação serve como guia definitivo para entender e implementar corretamente a Clean Architecture, garantindo código maintível, testável e escalável.*
          .patch('/api/v1/users/${data.id}', data: model.toMap)
          .then((value) => value.fold(HttpErrorResponse.left, Right));
    } catch (exception) {
      return Left(HttpErrorResponse.unknown(message: '$exception'));
    }
  }
}
```

---

## ✅ Checklist de Implementação por Feature

### 📋 Implementação Completa de uma Feature

#### 1. **Domain Layer** (Contratos e Regras)
- [ ] **Entity** criada com validações e regras de negócio
- [ ] **UseCase Interface** definida com contratos claros
- [ ] **Repository Interface** definida para acesso aos dados
- [ ] **Failures** específicos criados para erros de domínio
- [ ] **Enums** necessários definidos para tipagem forte

#### 2. **Infrastructure Layer** (Coordenação)
- [ ] **Model** implementado (extends Entity + serialização)
- [ ] **UseCase Implementation** criada (orquestração + validações)
- [ ] **Repository Implementation** criada (coordenação + cache)
- [ ] **DataSource Interface** definida para comunicação externa

#### 3. **Data Layer** (Comunicação Externa)
- [ ] **DataSource Remote** implementado (APIs, HTTP calls)
- [ ] **DataSource Local** implementado (se necessário)
- [ ] **Error Handling** completo com Either pattern
- [ ] **Timeouts e Retry** configurados

#### 4. **Presentation Layer** (UI)
- [ ] **Controller/Cubit** implementado
- [ ] **Page/Widget** criado
- [ ] **Error Handling** na UI
- [ ] **Loading/Success/Error States** implementados

---

## 🚀 Benefícios da Clean Architecture

### ✅ Vantagens Técnicas

1. **Testabilidade Máxima**: Cada camada testada isoladamente com mocks
2. **Manutenibilidade**: Mudanças localizadas, sem efeito dominó
3. **Escalabilidade**: Novas features seguem padrão estabelecido
4. **Flexibilidade**: Troca de tecnologias sem afetar regras de negócio
5. **Reutilização**: Entities e UseCases reutilizáveis
6. **SOLID Compliance**: Princípios rigorosamente aplicados

### ✅ Vantagens de Negócio

1. **Time-to-Market**: Desenvolvimento paralelo em teams
2. **Qualidade**: Menos bugs com testes automatizados
3. **Evolução**: Fácil adaptar a mudanças de requisitos
4. **Múltiplas Plataformas**: Core compartilhado entre apps
5. **Manutenção**: Custo reduzido de manutenção a longo prazo

### 🎯 Casos de Uso Ideais

- **Aplicações complexas** com regras de negócio elaboradas
- **Teams distribuídos** que trabalham em paralelo
- **Projetos de longo prazo** com evolução constante
- **Múltiplas plataformas** (mobile, web, desktop)
- **Diferentes fontes de dados** (REST, GraphQL, cache, local)
- **Ambientes variados** (dev, staging, production)

---

## 🔗 Navegação da Documentação

### 📚 Documentações por Camada

#### 🎯 Domain (Contratos e Regras)
- **[📖 UseCase Interfaces](./domain/i_usecases.md)** - Contratos de operações de negócio
- **[📖 Repository Interfaces](./domain/i_repositories.md)** - Contratos de acesso aos dados
- **[📖 Entities](./domain/entities.md)** - Objetos de negócio com validações
- **[📖 Enums](./domain/enums.md)** - Valores constantes e tipagem forte
- **[📖 Failures](./domain/failures.md)** - Tipos de erro específicos

#### 🔧 Infrastructure (Coordenação)
- **[📖 UseCase Implementations](./infra/implementations/usecases.md)** - Implementação de orquestração
- **[📖 Repository Implementations](./infra/implementations/repositories.md)** - Coordenação de dados
- **[📖 DataSource Interfaces](./infra/i_datasources.md)** - Contratos de comunicação
- **[📖 Models](./infra/models.md)** - Adaptadores de dados

#### 💾 Data (Comunicação Externa)
- **[📖 DataSource Implementations](./data/datasources.md)** - Comunicação real com APIs/DB

#### 🎨 Presentation (Interface e Estado)
- **[📖 Controllers](./presentation/controllers.md)** - Gerenciamento de estado reativo

### 🎨 Documentações Auxiliares
- **[📖 Naming Conventions](./conventions/naming.md)** - Padrões de nomenclatura
- **[📖 Code Style Guide](./conventions/code-style.md)** - Estilo de código
- **[📖 Error Handling](./conventions/error-handling.md)** - Tratamento de erros
- **[📖 Testing Strategy](./testing/unit-tests.md)** - Estratégias de teste
## � Guia de Implementação Prática

### 🚀 Começando uma Nova Feature

1. **📋 Defina no Domain**
   - Crie a Entity com regras de negócio
   - Defina IUseCase com operações necessárias
   - Crie IRepository para acesso aos dados
   - Defina Failures específicos

2. **🔧 Implemente na Infrastructure**
   - Crie Model (extends Entity + serialização)
   - Implemente UseCase (orquestração + validações)
   - Implemente Repository (coordenação)
   - Defina IDataSource para comunicação

3. **💾 Execute na Data**
   - Implemente DataSource (comunicação real)
   - Configure tratamento de erros
   - Implemente timeouts e retry

4. **🎨 Conecte na Presentation**
   - Use IUseCase nas controllers
   - Implemente estados na UI
   - Trate erros adequadamente

### 🔧 Refatorando Código Existente

1. **📊 Analise dependências** atuais
2. **🎯 Extraia Entities** do código existente
3. **🔍 Identifique regras de negócio** e isole em UseCases
4. **📡 Separe comunicação externa** em DataSources
5. **🧪 Adicione testes** para cada camada
6. **🔄 Migre gradualmente** mantendo funcionalidade

---

## 🎯 Resumo dos Benefícios por Princípio SOLID

### ✅ **Single Responsibility Principle (SRP)**
- **Entity**: Apenas dados e validações de negócio
- **UseCase**: Apenas uma operação de negócio específica
- **Repository**: Apenas coordenação de acesso aos dados
- **DataSource**: Apenas comunicação com uma fonte externa

### ✅ **Open/Closed Principle (OCP)**
- **Interfaces estáveis**: Novos recursos via implementações
- **Extensibilidade**: Novas implementações sem modificar existentes
- **Evolução segura**: Mudanças não quebram código existente

### ✅ **Liskov Substitution Principle (LSP)**
- **Implementações intercambiáveis**: Qualquer implementação funciona
- **Contratos respeitados**: Interfaces garantem comportamento
- **Testes consistentes**: Mocks e implementações reais equivalentes

### ✅ **Interface Segregation Principle (ISP)**
- **Interfaces coesas**: Apenas métodos relacionados
- **Dependências mínimas**: Clients dependem só do necessário
- **Evolução independente**: Interfaces mudam independentemente

### ✅ **Dependency Inversion Principle (DIP)**
- **Abstrações estáveis**: Dependência de interfaces, não implementações
- **Inversão completa**: Camadas altas não conhecem baixas
- **Flexibilidade máxima**: Fácil trocar implementações

---

*Esta documentação serve como **guia definitivo** para entender e implementar Clean Architecture com princípios SOLID, garantindo código **maintível**, **testável** e **escalável** em projetos Dart/Flutter.*
