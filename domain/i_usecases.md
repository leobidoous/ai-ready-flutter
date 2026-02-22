# UseCase Interfaces - Domain Layer

## 📋 O que são UseCase Interfaces?

**UseCase Interfaces** são contratos que definem **operações de negócio** que o sistema deve realizar. Eles representam casos de uso específicos e orquestram a lógica de negócio, coordenando Repositories e aplicando regras do domínio.

### 🎯 Responsabilidades

**✅ O que UseCase Interfaces FAZEM:**

- Definem **operações de negócio** específicas (GetUser, CreateUser, etc.)
- Especificam **assinaturas de métodos** sem implementação
- Retornam **Either<Failure, Success>** para tratamento de erros
- Trabalham com **Entities** (objetos de negócio puros)
- Representam **intenções do usuário** ou sistema

**❌ O que UseCase Interfaces NÃO FAZEM:**

- Não implementam lógica (apenas definem contratos)
- Não acessam dados diretamente (usam Repositories)
- Não conhecem detalhes de implementação
- Não fazem transformação de dados (Model ↔ Entity)

---

## 🏗️ Exemplo Completo: IUserUsecase

```dart
import 'package:base_core/base_core.dart' show Either, Unit;
import '../entities/user_entity.dart';
import '../entities/user_filters_entity.dart';
import '../entities/user_result_entity.dart';
import '../failures/i_user_failures.dart';

abstract class IUserUsecase {
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
6. **Métodos representam casos de uso** - Cada método é uma operação de negócio

---

## 🎯 Diferença: UseCase vs Repository

### Repository (Acesso aos Dados)

```dart
abstract class IUserRepository {
  // Foco: COMO acessar os dados
  Future<Either<IUserFailure, UserEntity>> getUserById({required String id});
  Future<Either<IUserFailure, UserEntity>> updateUser({required UserEntity data});
}
```

### UseCase (Lógica de Negócio)

```dart
abstract class IUserUsecase {
  // Foco: O QUE fazer com os dados (regras de negócio)
  Future<Either<IUserFailure, UserEntity>> getUserById({required String id});
  Future<Either<IUserFailure, UserEntity>> updateUser({required UserEntity data});
}
```

**Diferença na Implementação:**

- **Repository**: Coordena datasources, transforma Models em Entities
- **UseCase**: Aplica regras de negócio, valida dados, orquestra múltiplos repositories

---

## 🎯 Padrões de Métodos

### 1. **Operações de Consulta (Get/Fetch)**

```dart
// Buscar único recurso
Future<Either<IUserFailure, UserEntity>> getUserById({
  required String id,
});

// Buscar recurso logado
Future<Either<IUserFailure, UserEntity>> getLoggedUser();

// Listar com filtros
Future<Either<IUserFailure, UserResultEntity>> fetchUsers({
  required UserFiltersEntity filters,
});
```

**Use para:** Buscar dados sem modificá-los.

### 2. **Operações de Comando (Create/Update/Delete)**

```dart
// Criar
Future<Either<IUserFailure, UserEntity>> createUser({
  required UserEntity data,
});

// Atualizar
Future<Either<IUserFailure, UserEntity>> updateUser({
  required UserEntity data,
});

// Deletar
Future<Either<IUserFailure, Unit>> deleteUser({
  required String id,
});
```

**Use para:** Modificar dados (criar, atualizar, deletar).

### 3. **Operações Específicas de Negócio**

```dart
// Operação específica com regras de negócio
Future<Either<IUserFailure, Unit>> changeUserPassword({
  required String id,
  required String newPassword,
  required String currentPassword,
});

// Operação complexa que envolve múltiplos passos
Future<Either<IUserFailure, UserEntity>> activateUserAccount({
  required String id,
  required String activationCode,
});
```

**Use para:** Operações que não são CRUD simples e envolvem regras de negócio.

---

## 🎨 Padrões e Convenções

### ✅ Nomenclatura

| Tipo          | Padrão                      | Exemplo                                    |
| ------------- | --------------------------- | ------------------------------------------ |
| **Interface** | `I{Contexto}Usecase`        | `IUserUsecase`, `IProductUsecase`          |
| **Arquivo**   | `i_{contexto}_usecase.dart` | `i_user_usecase.dart`                      |
| **Métodos**   | `verbo + Substantivo`       | `getUser`, `createUser`, `activateAccount` |

### ✅ Estrutura do Arquivo

```dart
// 1. Imports
import 'package:base_core/base_core.dart' show Either, Unit;
import '../entities/user_entity.dart';
import '../failures/i_user_failures.dart';

// 2. Interface abstrata
abstract class IUserUsecase {
  // 3. Métodos (agrupados por tipo de operação)

  // Consultas
  Future<Either<IUserFailure, UserEntity>> getLoggedUser();
  Future<Either<IUserFailure, UserEntity>> getUserById({required String id});

  // Comandos
  Future<Either<IUserFailure, UserEntity>> createUser({required UserEntity data});
  Future<Either<IUserFailure, UserEntity>> updateUser({required UserEntity data});
  Future<Either<IUserFailure, Unit>> deleteUser({required String id});

  // Operações específicas
  Future<Either<IUserFailure, Unit>> changePassword({
    required String id,
    required String newPassword,
    required String currentPassword,
  });
}
```

---

## 📋 Checklist de Implementação

Ao criar um UseCase Interface:

- [ ] **Nomenclatura** segue padrão (`I{Contexto}Usecase`)
- [ ] **Arquivo nomeado** corretamente (`i_{contexto}_usecase.dart`)
- [ ] **Classe abstrata** (sem implementação)
- [ ] **Todos os métodos** retornam `Either<Failure, Success>`
- [ ] **Usa Entities** do domain (não Models)
- [ ] **Usa Failures** específicos do contexto
- [ ] **Named parameters** para clareza
- [ ] **Métodos representam casos de uso** de negócio
- [ ] **Imports apenas do domain** (entities, failures)

---

## 🚀 Benefícios dos UseCase Interfaces

### ✅ Single Responsibility Principle (SRP)

```dart
// ❌ Um usecase fazendo muitas coisas
abstract class IUserUsecase {
  Future<Either<IUserFailure, UserEntity>> doEverything();
}

// ✅ Cada usecase com responsabilidade única
abstract class IGetUserUsecase {
  Future<Either<IUserFailure, UserEntity>> call(String id);
}

abstract class ICreateUserUsecase {
  Future<Either<IUserFailure, UserEntity>> call(UserEntity data);
}

abstract class IUpdateUserUsecase {
  Future<Either<IUserFailure, UserEntity>> call(UserEntity data);
}
```

### ✅ Testabilidade

```dart
// Fácil criar mock para testes
class MockUserUsecase implements IUserUsecase {
  @override
  Future<Either<IUserFailure, UserEntity>> getUserById({
    required String id,
  }) async {
    return Right(UserEntity(/* dados de teste */));
  }
}

// Usar no teste
final controller = UserController(MockUserUsecase());
```

### ✅ Separação de Responsabilidades

```dart
// UseCase: Regras de negócio
class UpdateUserUsecase implements IUserUsecase {
  @override
  Future<Either<IUserFailure, UserEntity>> updateUser({
    required UserEntity data,
  }) async {
    // 1. Validar regras de negócio
    if (data.email.isEmpty) {
      return Left(UserValidationError(message: 'Email obrigatório'));
    }

    // 2. Chamar repository
    return repository.updateUser(data: data);
  }
}

// Repository: Acesso aos dados
class UserRepository implements IUserRepository {
  @override
  Future<Either<IUserFailure, UserEntity>> updateUser({
    required UserEntity data,
  }) async {
    // 1. Transformar Entity → Model
    final model = UserModel.fromEntity(data);

    // 2. Chamar datasource
    final result = await datasource.updateUser(model);

    // 3. Transformar Model → Entity
    return result.map((model) => model.toEntity);
  }
}
```

### ✅ Documentação Viva

```dart
// Interface documenta quais operações de negócio existem
abstract class IUserUsecase {
  // Olhando a interface, sei exatamente o que o sistema faz:
  Future<Either<IUserFailure, UserEntity>> getLoggedUser();
  Future<Either<IUserFailure, UserEntity>> createUser({required UserEntity data});
  Future<Either<IUserFailure, UserEntity>> updateUser({required UserEntity data});
  Future<Either<IUserFailure, Unit>> deleteUser({required String id});
  Future<Either<IUserFailure, Unit>> changePassword({...});
}
```

---

## 🔗 Próximos Passos

1. **[Implementar UseCase](../infra/usecases.md)** - Implementação real na Infrastructure
2. **[Usar em Controllers](../presentation/controllers.md)** - Chamar UseCases na Presentation
3. **[Criar Tests](../testing/unit-tests.md)** - Testar UseCases isoladamente

---

_UseCase Interfaces definem as operações de negócio do sistema, mantendo o Domain independente de detalhes de implementação._
