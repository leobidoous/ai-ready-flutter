# Failure Interfaces - Domain Layer

## 📋 O que são Failures?

**Failures** são tipos de erro específicos do domínio que representam falhas de negócio. Eles são usados com o **Either pattern** para tratamento de erros tipado e seguro, substituindo exceções por valores retornáveis.

### 🎯 Responsabilidades

**✅ O que Failures FAZEM:**

- Representam **erros específicos do domínio** (UserNotFound, ServerError, etc.)
- Fornecem **mensagens descritivas** sobre o erro
- Permitem **tratamento tipado** de erros via Either pattern
- Agrupam erros relacionados por **contexto** (User, Distributor, App, etc.)
- Herdam de **ICustomFailure** para padronização

**❌ O que Failures NÃO FAZEM:**

- Não são exceções (não usam throw/catch)
- Não contêm lógica de negócio
- Não fazem comunicação com APIs
- Não dependem de frameworks externos

---

## 🏗️ Estrutura de Failures

### Padrão: Interface Base + Implementações Específicas

Cada contexto (User, Distributor, App) tem:

1. **Interface abstrata** - Agrupa todos os erros do contexto
2. **Classes concretas** - Erros específicos que podem ocorrer

```dart
import 'package:base_core/base_core.dart' show ICustomFailure;

// 1. Interface base do contexto
abstract class IUserFailure extends ICustomFailure {
  IUserFailure({required super.message});
}

// 2. Erros específicos
class UserServerError extends IUserFailure {
  UserServerError({required super.message});
}

class UserUnknownError extends IUserFailure {
  UserUnknownError({required super.message});
}

class UserUnauthenticatedError extends IUserFailure {
  UserUnauthenticatedError({required super.message});
}

class UpdateUserDataError extends IUserFailure {
  UpdateUserDataError({required super.message});
}

class UserNotFoundError extends IUserFailure {
  UserNotFoundError({required super.message});
}
```

### 🔑 Elementos Essenciais

1. **Interface abstrata** - Agrupa erros relacionados (`IUserFailure`, `IDistributorFailure`)
2. **Herança de ICustomFailure** - Padronização via base_core
3. **Campo `message`** - Mensagem descritiva do erro
4. **Classes concretas** - Erros específicos que podem ocorrer
5. **Nomenclatura clara** - Nome do erro descreve o problema

---

## 🎨 Padrões e Convenções

### ✅ Nomenclatura

| Tipo                | Padrão                       | Exemplo                                  |
| ------------------- | ---------------------------- | ---------------------------------------- |
| **Interface**       | `I{Contexto}Failure`         | `IUserFailure`, `IDistributorFailure`    |
| **Arquivo**         | `i_{contexto}_failures.dart` | `i_user_failures.dart`                   |
| **Classe concreta** | `{Contexto}{Tipo}Error`      | `UserServerError`, `UpdateUserDataError` |

### ✅ Estrutura do Arquivo

```dart
// 1. Import do ICustomFailure
import 'package:base_core/base_core.dart' show ICustomFailure;

// 2. Interface abstrata do contexto
abstract class IUserFailure extends ICustomFailure {
  IUserFailure({required super.message});
}

// 3. Erros específicos (agrupados por tipo)
// Erros de servidor
class UserServerError extends IUserFailure {
  UserServerError({required super.message});
}

// Erros de autenticação
class UserUnauthenticatedError extends IUserFailure {
  UserUnauthenticatedError({required super.message});
}

// Erros de operação
class UpdateUserDataError extends IUserFailure {
  UpdateUserDataError({required super.message});
}

class UserNotFoundError extends IUserFailure {
  UserNotFoundError({required super.message});
}
```

### ✅ Tipos Comuns de Failures

```dart
// 1. ServerError - Erros de servidor (500, 502, 503)
class UserServerError extends IUserFailure {
  UserServerError({required super.message});
}

// 2. UnknownError - Erros inesperados/desconhecidos
class UserUnknownError extends IUserFailure {
  UserUnknownError({required super.message});
}

// 3. UnauthenticatedError - Erros de autenticação (401, 403)
class UserUnauthenticatedError extends IUserFailure {
  UserUnauthenticatedError({required super.message});
}

// 4. NotFoundError - Recurso não encontrado (404)
class UserNotFoundError extends IUserFailure {
  UserNotFoundError({required super.message});
}

// 5. ValidationError - Erros de validação de dados
class UserValidationError extends IUserFailure {
  UserValidationError({required super.message});
}

// 6. Erros específicos de operação
class UpdateUserDataError extends IUserFailure {
  UpdateUserDataError({required super.message});
}

class DeleteUserError extends IUserFailure {
  DeleteUserError({required super.message});
}
```

---

## 🎯 Como Usar Failures

### Em Repository Interfaces

```dart
import 'package:fpdart/fpdart.dart';
import '../entities/user_entity.dart';
import '../failures/i_user_failures.dart';

abstract class IUserRepository {
  // Either<Failure, Success>
  Future<Either<IUserFailure, UserEntity>> getUserById(String id);
  Future<Either<IUserFailure, UserEntity>> updateUser(UserEntity user);
  Future<Either<IUserFailure, List<UserEntity>>> getAllUsers();
}
```

### Em UseCase Interfaces

```dart
import 'package:fpdart/fpdart.dart';
import '../entities/user_entity.dart';
import '../failures/i_user_failures.dart';

abstract class IGetUserUsecase {
  Future<Either<IUserFailure, UserEntity>> call(String id);
}

abstract class IUpdateUserUsecase {
  Future<Either<IUserFailure, UserEntity>> call(UserEntity user);
}
```

**Regra:** Sempre use `Either<Failure, Success>` como retorno de métodos que podem falhar.

---

## 📋 Checklist de Implementação

Ao criar Failures para um novo contexto:

- [ ] **Interface abstrata** criada (`I{Contexto}Failure`)
- [ ] **Herda de ICustomFailure** do base_core
- [ ] **Arquivo nomeado** corretamente (`i_{contexto}_failures.dart`)
- [ ] **Erros comuns** implementados (ServerError, UnknownError, UnauthenticatedError)
- [ ] **Erros específicos** do contexto implementados
- [ ] **Nomenclatura clara** que descreve o problema
- [ ] **Campo `message`** obrigatório em todos os erros
- [ ] **Usado com Either** em repositories e usecases

---

## 🚀 Benefícios dos Failures Tipados

### ✅ Tratamento Explícito de Erros

```dart
// ❌ Exceções - Tratamento implícito e propenso a erros
try {
  final user = await getUser(id);
} catch (e) {
  // Que tipo de erro? Não sabemos!
  print('Erro: $e');
}

// ✅ Either + Failures - Tratamento explícito e tipado
final result = await getUser(id);
result.fold(
  (failure) {
    // Sabemos exatamente qual tipo de erro
    if (failure is UserNotFoundError) {
      // Tratamento específico
    }
  },
  (user) {
    // Sucesso
  },
);
```

### ✅ Compilador Garante Tratamento

```dart
// Either força você a tratar ambos os casos
final result = await getUser(id);

// ❌ Não compila - precisa tratar Left e Right
// final user = result;

// ✅ Compila - tratamento explícito
result.fold(
  (failure) => handleError(failure),
  (user) => handleSuccess(user),
);
```

### ✅ Pattern Matching por Tipo

```dart
result.fold(
  (failure) {
    // Pattern matching seguro
    switch (failure) {
      case UserNotFoundError():
        print('Usuário não encontrado');
      case UserUnauthenticatedError():
        print('Não autenticado');
      case UserServerError():
        print('Erro no servidor');
      default:
        print('Erro desconhecido');
    }
  },
  (user) => print('Sucesso: ${user.name}'),
);
```

### ✅ Testabilidade

```dart
// Fácil criar failures para testes
final failure = UserNotFoundError(message: 'User not found');
final result = Left<IUserFailure, UserEntity>(failure);

// Fácil verificar tipo de erro
expect(result.isLeft(), true);
expect(result.getLeft().toNullable(), isA<UserNotFoundError>());
```

### ✅ Documentação Viva

```dart
// Failures documentam quais erros podem ocorrer
abstract class IUserRepository {
  // Olhando a assinatura, sei que pode retornar IUserFailure
  Future<Either<IUserFailure, UserEntity>> getUserById(String id);

  // Posso ver quais erros específicos existem:
  // - UserNotFoundError
  // - UserServerError
  // - UserUnauthenticatedError
  // - etc.
}
```

---

## 🔗 Próximos Passos

1. **[Definir Repository Interfaces](./i_repositories.md)** - Usar Failures nos contratos
2. **[Definir UseCase Interfaces](./i_usecases.md)** - Usar Failures nos casos de uso
3. **[Implementar na Infrastructure](../infra/repositories.md)** - Transformar erros técnicos em Failures

---

_Failures tipados com Either pattern eliminam exceções e tornam o tratamento de erros explícito, seguro e testável._
