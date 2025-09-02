# Responsabilidades por Camadas - Clean Architecture

## 📋 Visão Geral

Este documento define **responsabilidades específicas** de cada camada na Clean Architecture e explica **por que utilizamos abstrações** para manter a separação de responsabilidades e garantir flexibilidade e testabilidade.

### 🎯 Objetivos das Abstrações

- **Inversão de Dependências**: Camadas superiores não dependem de implementações concretas
- **Testabilidade**: Facilita criação de mocks e testes unitários
- **Flexibilidade**: Permite múltiplas implementações sem quebrar o código
- **Manutenibilidade**: Mudanças em uma camada não afetam outras
- **Princípio Aberto/Fechado**: Aberto para extensão, fechado para modificação

---

## 🏗️ Camadas e Suas Responsabilidades

### 🎯 Domain Layer (Núcleo do Negócio)

> **Filosofia**: "O que o sistema deve fazer, não como fazer"

#### 📋 Responsabilidades

**✅ O que a camada Domain FAZ:**
- Define **regras de negócio puras** sem dependências externas
- Estabelece **contratos** através de interfaces
- Representa **entidades de negócio** com suas validações
- Define **failures específicos** do domínio
- Especifica **casos de uso** que o sistema deve suportar

**❌ O que a camada Domain NÃO FAZ:**
- Não implementa acesso a dados externos
- Não contém lógica de apresentação ou UI
- Não depende de frameworks ou bibliotecas externas
- Não implementa protocolos de comunicação
- Não define tecnologias específicas

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
- **Implementa contratos** definidos no Domain
- **Coordena** comunicação entre camadas
- **Aplica regras de negócio** usando repositories
- **Transforma dados** entre formatos (Models)
- **Orquestra** múltiplas operações quando necessário

**❌ O que a camada Infrastructure NÃO FAZ:**
- Não faz comunicação direta com APIs/Banco de dados
- Não contém regras de negócio puras (ficam no Domain)
- Não define contratos (apenas implementa)
- Não contém lógica de apresentação

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
- **Implementa comunicação real** com APIs, bancos, cache
- **Executa protocolos** HTTP, SQL, NoSQL
- **Serializa/Deserializa** dados para formato de transporte
- **Gerencia conexões** e autenticação
- **Implementa retry** e circuit breaker quando necessário

**❌ O que a camada Data NÃO FAZ:**
- Não contém regras de negócio
- Não define contratos (apenas implementa)
- Não transforma dados em entities
- Não coordena múltiplas operações

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

### 📊 Direção das Dependências

```
┌─────────────────────────────────────────────────────────────┐
│                    PRESENTATION                             │
│                        │                                    │
│                        ▼                                    │
│                 IUserUsecase ◄─────────────────────────────┐│
└─────────────────────┬───────────────────────────────────────┘│
                      │                                        │
┌─────────────────────▼───────────────────────────────────────┐│
│                INFRASTRUCTURE                               ││
│                                                             ││
│  UserUsecase ──────► IUserRepository ◄──────────────────────┤│
│      │                   ▲                                 ││
│      │                   │                                 ││
│      ▼                   │                                 ││
│  UserRepository ────► IUserDatasource ◄─────────────────────┤│
└─────────────────────┬───────────────────────────────────────┘│
                      │                                        │
┌─────────────────────▼───────────────────────────────────────┐│
│                     DATA                                   ││
│                                                             ││
│            UserDatasource ──────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

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

## ✅ Checklist de Implementação Completa

### 📋 Para cada Funcionalidade

#### Domain Layer
- [ ] **Entity** criada com regras de negócio
- [ ] **Use Case Interface** definida
- [ ] **Repository Interface** definida
- [ ] **Failures** específicos criados
- [ ] **Enums** necessários definidos

#### Infrastructure Layer
- [ ] **Model** implementado (extends Entity + EquatableMixin)
- [ ] **Use Case** implementado (validações + orquestração)
- [ ] **Repository** implementado (coordenação + cache)
- [ ] **DataSource Interface** definida

#### Data Layer
- [ ] **DataSource Remote** implementado (HTTP calls)
- [ ] **DataSource Local** implementado (se necessário)
- [ ] **Error Handling** completo
- [ ] **Timeouts** configurados

#### Presentation Layer
- [ ] **Controller** implementado
- [ ] **Page/Widget** criado
- [ ] **Error Handling** na UI
- [ ] **Loading States** implementados

---

## 🚀 Benefícios desta Arquitetura

### ✅ Vantagens

1. **Testabilidade**: Cada camada pode ser testada isoladamente
2. **Manutenibilidade**: Mudanças em uma camada não afetam outras
3. **Escalabilidade**: Fácil adicionar novas funcionalidades
4. **Flexibilidade**: Troca de implementações sem afetar regras de negócio
5. **Reusabilidade**: Entities e Use Cases podem ser reutilizados
6. **Separação de Responsabilidades**: Cada classe tem uma responsabilidade clara

### 🎯 Casos de Uso Ideais

- **Aplicações complexas** com muitas regras de negócio
- **Teams grandes** que precisam trabalhar em paralelo
- **Projetos de longo prazo** que evoluem constantemente
- **Múltiplas plataformas** (mobile, web, desktop)
- **Diferentes fontes de dados** (REST, GraphQL, local)

---

## 🔗 Links de Referência

### Documentações Detalhadas
- [📖 Entities](./domain/entites.md) - Regras de negócio fundamentais
- [📖 Models](./infra/models.md) - Serialização e adaptação
- [📖 Use Cases](./domain/usecases.md) - Orquestração e regras de aplicação
- [📖 Repositories](./infra/repositories.md) - Coordenação de dados
- [📖 DataSources](./data/datasources.md) - Comunicação com fontes externas

### Próximas Documentações
- 🔜 **Failures & Error Handling** - Estratégias de tratamento de erros
- 🔜 **Dependency Injection** - Configuração de DI com GetIt
- 🔜 **Testing Strategies** - Testes por camada
- 🔜 **Performance & Optimization** - Cache, lazy loading, etc.

---

## 💡 Dicas de Implementação

### 🚀 Começando um Novo Projeto

1. **Defina as Entities** primeiro (regras de negócio)
2. **Crie os Use Cases** (fluxos da aplicação)
3. **Implemente os Models** (serialização)
4. **Configure DataSources** (APIs/Database)
5. **Conecte com Repositories** (coordenação)
6. **Finalize com Presentation** (UI)

### 🔧 Refatorando Projeto Existente

1. **Extraia Entities** das classes existentes
2. **Isole regras de negócio** em Use Cases
3. **Separe serialização** em Models
4. **Abstraia fontes de dados** em DataSources
5. **Teste cada camada** isoladamente
6. **Migre gradualmente** a UI

Esta arquitetura garante código limpo, testável e escalável para projetos Flutter de qualquer tamanho!
