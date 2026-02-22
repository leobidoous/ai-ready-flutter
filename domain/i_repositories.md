# Repository Interfaces - Domain Layer

## 📋 O que são Repository Interfaces?

**Repository Interfaces** são contratos que definem **como acessar dados** sem especificar a implementação. Eles abstraem a origem dos dados (API, banco local, cache) e garantem que o Domain não dependa de detalhes técnicos.

### 🎯 Responsabilidades

**✅ O que Repository Interfaces FAZEM:**

- Definem **contratos de acesso aos dados** (CRUD operations)
- Especificam **assinaturas de métodos** sem implementação
- Retornam **Either<Failure, Success>** para tratamento de erros
- Trabalham com **Entities** (objetos de negócio puros)
- Abstraem a **origem dos dados** (API, DB, cache)

**❌ O que Repository Interfaces NÃO FAZEM:**

- Não implementam lógica (apenas definem contratos)
- Não conhecem detalhes de implementação (HTTP, SQL, etc.)
- Não fazem transformação de dados (Model ↔ Entity)
- Não contêm regras de negócio complexas

---

## 🏗️ Exemplo Completo: IUserRepository

```dart
import 'package:base_core/base_core.dart' show Either, Unit;
import '../entities/user_entity.dart';
import '../entities/user_filters_entity.dart';
import '../entities/user_result_entity.dart';
import '../failures/i_user_failures.dart';

abstract class IUserRepository {
  // 1. Buscar usuário logado
  Future<Either<IUserFailure, UserEntity>> getLoggedUser();

  // 2. Buscar usuário por ID
  Future<Either<IUserFailure, UserEntity>> getUserById({
    required String id,
  });

  // 3. Listar usuários com filtros
  Future<Either<IUserFailure, UserResultEntity>> fetchUsers({
    required UserFiltersEntity filters,
  });

  // 4. Criar novo usuário
  Future<Either<IUserFailure, UserEntity>> createUser({
    required UserEntity data,
  });

  // 5. Atualizar usuário
  Future<Either<IUserFailure, UserEntity>> updateUser({
    required UserEntity data,
  });

  // 6. Atualizar usuário por ID
  Future<Either<IUserFailure, UserEntity>> updateUserById({
    required String id,
    required UserEntity data,
  });

  // 7. Deletar conta do usuário
  Future<Either<IUserFailure, UserEntity>> deleteUserAccount();

  // 8. Alterar senha do usuário
  Future<Either<IUserFailure, Unit>> changeUserPassword({
    required String id,
    required String newPassword,
    required String currentPassword,
  });
}
```

### 🔑 Elementos Essenciais

1. **Classe abstrata** - Define contrato sem implementação
2. **Either<Failure, Success>** - Tratamento de erros tipado
3. **Entities** - Trabalha com objetos de negócio puros
4. **Named parameters** - Parâmetros explícitos e legíveis
5. **Future** - Operações assíncronas
6. **Unit** - Retorno quando não há dados (apenas sucesso/falha)

---

## 🎯 Padrões de Métodos

### 1. **Buscar Único (Get/Fetch)**

```dart
// Buscar por ID
Future<Either<IUserFailure, UserEntity>> getUserById({
  required String id,
});

// Buscar usuário logado (sem parâmetros)
Future<Either<IUserFailure, UserEntity>> getLoggedUser();
```

**Use para:** Buscar um único recurso.

### 2. **Listar com Filtros (Fetch/List)**

```dart
// Listar com filtros e paginação
Future<Either<IUserFailure, UserResultEntity>> fetchUsers({
  required UserFiltersEntity filters,
});
```

**Use para:** Buscar múltiplos recursos com filtros, paginação, ordenação.

### 3. **Criar (Create)**

```dart
// Criar novo recurso
Future<Either<IUserFailure, UserEntity>> createUser({
  required UserEntity data,
});
```

**Use para:** Criar novos recursos. Retorna o recurso criado.

### 4. **Atualizar (Update)**

```dart
// Atualizar recurso completo
Future<Either<IUserFailure, UserEntity>> updateUser({
  required UserEntity data,
});

// Atualizar por ID específico
Future<Either<IUserFailure, UserEntity>> updateUserById({
  required String id,
  required UserEntity data,
});
```

**Use para:** Atualizar recursos existentes. Retorna o recurso atualizado.

### 5. **Deletar (Delete)**

```dart
// Deletar recurso
Future<Either<IUserFailure, UserEntity>> deleteUserAccount();

// Deletar por ID
Future<Either<IUserFailure, Unit>> deleteUserById({
  required String id,
});
```

**Use para:** Remover recursos. Pode retornar Entity ou Unit.

### 6. **Operações Específicas**

```dart
// Operação específica de negócio
Future<Either<IUserFailure, Unit>> changeUserPassword({
  required String id,
  required String newPassword,
  required String currentPassword,
});
```

**Use para:** Operações específicas que não são CRUD simples.

---

## 🎨 Padrões e Convenções

### ✅ Nomenclatura

| Tipo          | Padrão                         | Exemplo                                 |
| ------------- | ------------------------------ | --------------------------------------- |
| **Interface** | `I{Contexto}Repository`        | `IUserRepository`, `IProductRepository` |
| **Arquivo**   | `i_{contexto}_repository.dart` | `i_user_repository.dart`                |
| **Métodos**   | `verbo + Substantivo`          | `getUser`, `fetchUsers`, `createUser`   |

### ✅ Estrutura do Arquivo

```dart
// 1. Imports
import 'package:base_core/base_core.dart' show Either, Unit;
import '../entities/user_entity.dart';
import '../failures/i_user_failures.dart';

// 2. Interface abstrata
abstract class IUserRepository {
  // 3. Métodos (agrupados por tipo de operação)

  // GET operations
  Future<Either<IUserFailure, UserEntity>> getLoggedUser();
  Future<Either<IUserFailure, UserEntity>> getUserById({required String id});

  // LIST operations
  Future<Either<IUserFailure, UserResultEntity>> fetchUsers({
    required UserFiltersEntity filters,
  });

  // CREATE operations
  Future<Either<IUserFailure, UserEntity>> createUser({
    required UserEntity data,
  });

  // UPDATE operations
  Future<Either<IUserFailure, UserEntity>> updateUser({
    required UserEntity data,
  });

  // DELETE operations
  Future<Either<IUserFailure, Unit>> deleteUser({required String id});
}
```

### ✅ Retornos Comuns

```dart
// 1. Retornar Entity única
Future<Either<IUserFailure, UserEntity>> getUser();

// 2. Retornar lista de Entities
Future<Either<IUserFailure, List<UserEntity>>> getUsers();

// 3. Retornar resultado paginado
Future<Either<IUserFailure, UserResultEntity>> fetchUsers();

// 4. Retornar Unit (sem dados, apenas sucesso/falha)
Future<Either<IUserFailure, Unit>> deleteUser();

// 5. Retornar bool
Future<Either<IUserFailure, bool>> checkUserExists();
```

---

## 📋 Checklist de Implementação

Ao criar um Repository Interface:

- [ ] **Nomenclatura** segue padrão (`I{Contexto}Repository`)
- [ ] **Arquivo nomeado** corretamente (`i_{contexto}_repository.dart`)
- [ ] **Classe abstrata** (sem implementação)
- [ ] **Todos os métodos** retornam `Either<Failure, Success>`
- [ ] **Usa Entities** do domain (não Models)
- [ ] **Usa Failures** específicos do contexto
- [ ] **Named parameters** para clareza
- [ ] **Métodos agrupados** por tipo de operação (GET, LIST, CREATE, UPDATE, DELETE)
- [ ] **Imports apenas do domain** (entities, failures)

---

## 🚀 Benefícios dos Repository Interfaces

### ✅ Inversão de Dependências (DIP)

```dart
// ❌ Domain dependendo de implementação
class UserUsecase {
  final UserRepositoryImpl repository;  // Dependência concreta
}

// ✅ Domain dependendo de abstração
class UserUsecase {
  final IUserRepository repository;  // Dependência abstrata
}
```

### ✅ Testabilidade

```dart
// Fácil criar mock para testes
class MockUserRepository implements IUserRepository {
  @override
  Future<Either<IUserFailure, UserEntity>> getUserById({
    required String id,
  }) async {
    return Right(UserEntity(/* dados de teste */));
  }

  // ... outros métodos
}

// Usar no teste
final usecase = UserUsecase(MockUserRepository());
```

### ✅ Múltiplas Implementações

```dart
// Implementação com API
class UserApiRepository implements IUserRepository { ... }

// Implementação com banco local
class UserLocalRepository implements IUserRepository { ... }

// Implementação com cache
class UserCacheRepository implements IUserRepository { ... }

// Trocar implementação sem afetar Domain
final repository = UserApiRepository();  // ou UserLocalRepository()
```

### ✅ Abstração da Origem dos Dados

```dart
// UseCase não sabe de onde vêm os dados
class GetUserUsecase {
  final IUserRepository repository;

  Future<Either<IUserFailure, UserEntity>> call(String id) {
    // Pode ser API, DB local, cache... não importa!
    return repository.getUserById(id: id);
  }
}
```

---

## 🔗 Próximos Passos

1. **[Definir UseCase Interfaces](./i_usecases.md)** - Contratos de operações de negócio
2. **[Implementar Repository](../infra/repositories.md)** - Implementação real na Infrastructure
3. **[Criar DataSource Interfaces](../infra/i_datasources.md)** - Contratos de comunicação externa

---

_Repository Interfaces abstraem o acesso aos dados, permitindo que o Domain permaneça independente de detalhes técnicos._
