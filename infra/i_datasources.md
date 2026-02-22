# DataSource Interfaces - Infrastructure Layer

## 📋 O que são DataSource Interfaces?

**DataSource Interfaces** são contratos que definem **como comunicar com fontes externas** (APIs, databases, cache). Eles trabalham com dados brutos (JSON, HTTP responses) e são a última camada de abstração antes da comunicação real na camada Data.

### 🎯 Responsabilidades

**✅ O que DataSource Interfaces FAZEM:**

- Definem **contratos de comunicação externa** (HTTP, GraphQL, SQL, etc.)
- Retornam **Either<HttpErrorResponse, HttpDriverResponse>** (erros técnicos)
- Trabalham com **dados brutos** (JSON, não Entities)
- Especificam **endpoints e operações** de comunicação
- Abstraem **protocolos de comunicação** (REST, GraphQL, etc.)

**❌ O que DataSource Interfaces NÃO FAZEM:**

- Não implementam comunicação (apenas definem contratos)
- Não transformam dados em Entities (isso é no Repository)
- Não contêm regras de negócio
- Não retornam Failures de domínio (retornam erros técnicos HTTP)

---

## 🏗️ Exemplo Completo: IUserDatasource

```dart
import 'package:base_core/base_core.dart';
import '../../core/utils/http_error_response.dart';
import '../../domain/entities/user_entity.dart';
import '../../domain/entities/user_filters_entity.dart';

abstract class IUserDatasource {
  // 1. GET - Buscar usuário logado
  Future<Either<HttpErrorResponse, HttpDriverResponse>> getLoggedUser();

  // 2. GET - Buscar usuário por ID
  Future<Either<HttpErrorResponse, HttpDriverResponse>> getUserById({
    required String id,
  });

  // 3. GET - Listar usuários com filtros
  Future<Either<HttpErrorResponse, HttpDriverResponse>> fetchUsers({
    required UserFiltersEntity filters,
  });

  // 4. POST - Criar novo usuário
  Future<Either<HttpErrorResponse, HttpDriverResponse>> createUser({
    required UserEntity data,
  });

  // 5. PUT/PATCH - Atualizar usuário
  Future<Either<HttpErrorResponse, HttpDriverResponse>> updateUser({
    required UserEntity data,
  });

  // 6. PUT/PATCH - Atualizar usuário por ID
  Future<Either<HttpErrorResponse, HttpDriverResponse>> updateUserById({
    required String id,
    required UserEntity data,
  });

  // 7. DELETE - Deletar conta do usuário
  Future<Either<HttpErrorResponse, HttpDriverResponse>> deleteUserAccount();

  // 8. POST/PUT - Alterar senha do usuário
  Future<Either<HttpErrorResponse, HttpDriverResponse>> changeUserPassword({
    required String id,
    required String newPassword,
    required String currentPassword,
  });
}
```

### 🔑 Elementos Essenciais

1. **Classe abstrata** - Define contrato sem implementação
2. **Either<HttpErrorResponse, HttpDriverResponse>** - Sempre retorna erros técnicos HTTP
3. **HttpDriverResponse** - Resposta bruta (JSON) da API
4. **Named parameters** - Parâmetros explícitos e legíveis
5. **Entities como parâmetros** - Recebe Entities para enviar (serão serializadas)
6. **Operações CRUD completas** - GET, POST, PUT, DELETE

---

## 🎯 Diferença: Repository vs DataSource

### Domain: Repository Interface

```dart
// Trabalha com Entities e Failures de domínio
abstract class IUserRepository {
  Future<Either<IUserFailure, UserEntity>> getUserById({required String id});
  Future<Either<IUserFailure, UserEntity>> createUser({required UserEntity data});
}
```

### Infrastructure: DataSource Interface

```dart
// Trabalha com dados brutos HTTP
abstract class IUserDatasource {
  Future<Either<HttpErrorResponse, HttpDriverResponse>> getUserById({required String id});
  Future<Either<HttpErrorResponse, HttpDriverResponse>> createUser({required UserEntity data});
}
```

**Diferenças:**

- **Repository**: Retorna `Entity` + `Failure` de domínio
- **DataSource**: Retorna `HttpDriverResponse` (JSON bruto) + `HttpErrorResponse` (erro técnico)
- **Repository**: Transforma dados (Model ↔ Entity)
- **DataSource**: Apenas comunica (envia/recebe JSON)

---

## 🎨 Padrões e Convenções

### ✅ Nomenclatura

| Tipo          | Padrão                         | Exemplo                                 |
| ------------- | ------------------------------ | --------------------------------------- |
| **Interface** | `I{Contexto}Datasource`        | `IUserDatasource`, `IProductDatasource` |
| **Arquivo**   | `i_{contexto}_datasource.dart` | `i_user_datasource.dart`                |
| **Métodos**   | `verbo + Substantivo`          | `getUser`, `createUser`, `fetchUsers`   |

### ✅ Estrutura do Arquivo

```dart
// 1. Imports
import 'package:base_core/base_core.dart';
import '../../core/utils/http_error_response.dart';
import '../../domain/entities/user_entity.dart';

// 2. Interface abstrata
abstract class IUserDatasource {
  // 3. Métodos (agrupados por tipo de operação)

  // GET operations
  Future<Either<HttpErrorResponse, HttpDriverResponse>> getLoggedUser();
  Future<Either<HttpErrorResponse, HttpDriverResponse>> getUserById({
    required String id,
  });

  // POST operations
  Future<Either<HttpErrorResponse, HttpDriverResponse>> createUser({
    required UserEntity data,
  });

  // PUT/PATCH operations
  Future<Either<HttpErrorResponse, HttpDriverResponse>> updateUser({
    required UserEntity data,
  });

  // DELETE operations
  Future<Either<HttpErrorResponse, HttpDriverResponse>> deleteUser({
    required String id,
  });
}
```

### ✅ Retornos

```dart
// Sempre Either<HttpErrorResponse, HttpDriverResponse>
Future<Either<HttpErrorResponse, HttpDriverResponse>> getUser();

// HttpDriverResponse contém:
// - data: dynamic (JSON bruto da API)
// - statusCode: int?
// - headers: Map<String, dynamic>?

// HttpErrorResponse contém:
// - message: String (mensagem de erro)
// - statusCode: int? (código HTTP: 404, 500, etc.)
// - data: dynamic? (dados adicionais do erro)
```

---

## 📋 Checklist de Implementação

Ao criar um DataSource Interface:

- [ ] **Nomenclatura** segue padrão (`I{Contexto}Datasource`)
- [ ] **Arquivo nomeado** corretamente (`i_{contexto}_datasource.dart`)
- [ ] **Classe abstrata** (sem implementação)
- [ ] **Todos os métodos** retornam `Either<HttpErrorResponse, HttpDriverResponse>`
- [ ] **Named parameters** para clareza
- [ ] **Métodos correspondem** aos do Repository
- [ ] **Imports corretos** (base_core, http_error_response, entities)
- [ ] **Métodos agrupados** por tipo de operação (GET, POST, PUT, DELETE)

---

## 🚀 Benefícios dos DataSource Interfaces

### ✅ Abstração de Protocolo

```dart
// Interface não especifica HTTP, GraphQL, SQL, etc.
abstract class IUserDatasource {
  Future<Either<HttpErrorResponse, HttpDriverResponse>> getUser();
}

// Implementação pode ser HTTP REST
class UserHttpDatasource implements IUserDatasource { ... }

// Ou GraphQL
class UserGraphQLDatasource implements IUserDatasource { ... }

// Ou banco local
class UserLocalDatasource implements IUserDatasource { ... }
```

### ✅ Testabilidade

```dart
// Fácil criar mock para testes do Repository
class MockUserDatasource implements IUserDatasource {
  @override
  Future<Either<HttpErrorResponse, HttpDriverResponse>> getUserById({
    required String id,
  }) async {
    return Right(HttpDriverResponse(
      data: {
        'id': id,
        'name': 'Test User',
        'email': 'test@example.com',
        // ... outros campos
      },
    ));
  }

  // ... outros métodos
}

// Usar no teste do Repository
final repository = UserRepository(
  datasource: MockUserDatasource(),
);
```

### ✅ Separação de Responsabilidades

- **DataSource Interface (Infra)**: Define O QUE comunicar (contrato)
- **DataSource Implementation (Data)**: Executa comunicação real (HTTP, SQL, etc.)
- **Repository (Infra)**: Transforma dados brutos em Entities

---

## 🔗 Próximos Passos

1. **[Criar Models](./models.md)** - Transformar JSON ↔ Entity
2. **[Implementar Repository](./repositories.md)** - Usar DataSource e Models
3. **[Implementar DataSource](../data/datasources.md)** - Comunicação real na Data

---

_DataSource Interfaces abstraem a comunicação externa, trabalhando com dados brutos que serão transformados em Entities pelos Repositories._
