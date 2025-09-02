# Domain Failures - Clean Architecture

## 📚 Visão Geral

Os **Failures** na camada de **Domain** definem **QUE TIPOS** de erros podem ocorrer nas operações de negócio. Eles representam **falhas específicas do domínio**, permitindo tratamento granular de erros através do Either pattern.

### 🎯 Princípios Fundamentais dos Failures

**O QUE os Failures DEFINEM:**
- ✅ **Tipos de Erro Específicos**: Classificação clara de falhas de negócio
- ✅ **Herança de ICustomFailure**: Base comum para todos os failures
- ✅ **Mensagens Descritivas**: Informação clara sobre o que falhou
- ✅ **Granularidade de Erros**: Diferentes tipos para diferentes cenários
- ✅ **Tratamento Específico**: Permite diferentes ações para cada tipo de erro

**O QUE os Failures NÃO FAZEM:**
- ❌ **Não contêm lógica de tratamento**: Apenas definem tipos de erro
- ❌ **Não dependem de infraestrutura**: Zero dependências externas
- ❌ **Não especificam soluções**: Apenas identificam o problema
- ❌ **Não contêm dados técnicos**: Apenas informações de negócio
- ❌ **Não quebram SOLID**: Seguem princípios de responsabilidade única

### 🏗️ Localização e Estrutura

```
lib/src/domain/failures/
├── i_app_failures.dart
├── i_user_failures.dart
├── i_product_failures.dart
└── i_order_failures.dart
```

---

## 🔍 Anatomia de um Failure

### Estrutura Base

```dart
import 'package:base_core/base_core.dart' show ICustomFailure;

/// Interface base para falhas relacionadas a [Entity]
/// 
/// Define o contrato comum para todos os tipos de erro que podem
/// ocorrer em operações relacionadas a [Entity].
abstract class I[Entity]Failure extends ICustomFailure {
  I[Entity]Failure({required super.message});
}

/// Implementações específicas de cada tipo de erro
class [Entity]ValidationError extends I[Entity]Failure {
  [Entity]ValidationError({required super.message});
}

class [Entity]NotFoundError extends I[Entity]Failure {
  [Entity]NotFoundError({required super.message});
}

class [Entity]ServerError extends I[Entity]Failure {
  [Entity]ServerError({required super.message});
}
```

### Elementos Essenciais

1. **Herança de ICustomFailure**: Base comum do framework
2. **Interface Abstrata**: Tipo base para agrupamento
3. **Classes Concretas**: Tipos específicos de erro
4. **Mensagem Obrigatória**: Sempre requer mensagem descritiva
5. **Nomes Descritivos**: Indicam claramente o tipo de falha

---

## 📚 Exemplo Prático: IUserFailure

### Implementação Real

```dart
import 'package:base_core/base_core.dart' show ICustomFailure;

/// Interface base para falhas relacionadas a operações de usuário
/// 
/// Define o contrato comum para todos os tipos de erro que podem
/// ocorrer em operações de usuário (autenticação, perfil, etc.).
abstract class IUserFailure extends ICustomFailure {
  IUserFailure({required super.message});
}

/// Erro ao atualizar dados do usuário
/// 
/// Ocorre quando:
/// - Dados de entrada são inválidos
/// - Violação de regras de negócio na atualização
/// - Conflitos de unicidade (email, CPF)
class UpdateUserDataError extends IUserFailure {
  UpdateUserDataError({required super.message});
}

/// Erro de comunicação com servidor
/// 
/// Ocorre quando:
/// - Falha na comunicação com API
/// - Timeout de requisição
/// - Erro HTTP 5xx do servidor
class UserServerError extends IUserFailure {
  UserServerError({required super.message});
}

/// Erro desconhecido/inesperado
/// 
/// Ocorre quando:
/// - Exception não mapeada
/// - Erro inesperado de sistema
/// - Fallback para erros não categorizados
class UserUnknownError extends IUserFailure {
  UserUnknownError({required super.message});
}

/// Erro de autenticação
/// 
/// Ocorre quando:
/// - Token de acesso inválido ou expirado
/// - Credenciais incorretas
/// - Sessão não encontrada
class UserUnauthenticatedError extends IUserFailure {
  UserUnauthenticatedError({required super.message});
}
```

### Padrão de Uso no Either

```dart
// Nos Use Cases e Repositories
Future<Either<IUserFailure, UserEntity>> getUser() async {
  try {
    // Lógica de busca
    return Right(user);
  } on ValidationException catch (e) {
    return Left(UpdateUserDataError(message: e.message));
  } on NetworkException catch (e) {
    return Left(UserServerError(message: 'Erro de conexão: ${e.message}'));
  } on AuthException catch (e) {
    return Left(UserUnauthenticatedError(message: e.message));
  } catch (e) {
    return Left(UserUnknownError(message: 'Erro inesperado: $e'));
  }
}

// No tratamento da UI
result.fold(
  (failure) {
    switch (failure.runtimeType) {
      case UpdateUserDataError:
        showValidationErrors();
        break;
      case UserServerError:
        showNetworkError();
        break;
      case UserUnauthenticatedError:
        redirectToLogin();
        break;
      default:
        showGenericError();
    }
  },
  (user) => showUserData(user),
);
```

---

## 📋 Template para Failures

### Estrutura Básica

```dart
import 'package:base_core/base_core.dart' show ICustomFailure;

/// Interface base para falhas relacionadas a [Entity]
/// 
/// Define o contrato comum para todos os tipos de erro que podem
/// ocorrer em operações relacionadas a [Entity].
abstract class I[Entity]Failure extends ICustomFailure {
  I[Entity]Failure({required super.message});
}

/// [Breve descrição do tipo de erro]
/// 
/// Ocorre quando:
/// - [cenário 1]
/// - [cenário 2] 
/// - [cenário N]
class [Entity][TipoErro]Error extends I[Entity]Failure {
  [Entity][TipoErro]Error({required super.message});
}
```

### Tipos Comuns de Failures

**Erros de Validação:**
```dart
class [Entity]ValidationError extends I[Entity]Failure {
  [Entity]ValidationError({required super.message});
}
```

**Erros de Não Encontrado:**
```dart
class [Entity]NotFoundError extends I[Entity]Failure {
  [Entity]NotFoundError({required super.message});
}
```

**Erros de Servidor/Rede:**
```dart
class [Entity]ServerError extends I[Entity]Failure {
  [Entity]ServerError({required super.message});
}
```

**Erros de Autorização:**
```dart
class [Entity]UnauthorizedError extends I[Entity]Failure {
  [Entity]UnauthorizedError({required super.message});
}
```

**Erros de Conflito:**
```dart
class [Entity]ConflictError extends I[Entity]Failure {
  [Entity]ConflictError({required super.message});
}
```

**Erros de Regra de Negócio:**
```dart
class [Entity]BusinessRuleError extends I[Entity]Failure {
  [Entity]BusinessRuleError({required super.message});
}
```

**Erros Desconhecidos:**
```dart
class [Entity]UnknownError extends I[Entity]Failure {
  [Entity]UnknownError({required super.message});
}
```

---

## 🎯 Diretrizes para Failures

### ✅ Boas Práticas

```dart
// ✅ Interface específica por entidade
abstract class IUserFailure extends ICustomFailure {
  IUserFailure({required super.message});
}

// ✅ Nomes descritivos e específicos
class UserEmailAlreadyExistsError extends IUserFailure {
  UserEmailAlreadyExistsError({required super.message});
}

// ✅ Documentação clara dos cenários
/// Erro quando usuário tenta atualizar para email já existente
/// 
/// Ocorre quando:
/// - Email fornecido já está em uso por outro usuário
/// - Violação de constraint de unicidade no banco
class UserEmailConflictError extends IUserFailure {
  UserEmailConflictError({required super.message});
}

// ✅ Mensagens informativas
UserServerError(message: 'Falha na comunicação com servidor: timeout após 30s');
```

### ❌ Evitar

```dart
// ❌ Failures genéricos demais
class GeneralError extends ICustomFailure {
  GeneralError({required super.message});
}

// ❌ Nomes não descritivos
class UserError1 extends IUserFailure {
  UserError1({required super.message});
}

// ❌ Failures sem documentação
class UserSomethingWentWrongError extends IUserFailure {
  UserSomethingWentWrongError({required super.message});
}

// ❌ Mensagens vagas
UserUnknownError(message: 'Erro');
```

---

## 📋 Checklist para Failures

### Checklist de Criação ✅

**Estrutura Base:**
- [ ] Localizado em `lib/src/domain/failures/`
- [ ] Nome seguindo padrão `i_[entity]_failures.dart`
- [ ] Interface abstrata `I[Entity]Failure extends ICustomFailure`
- [ ] Apenas import de `ICustomFailure` do base_core

**Tipos de Failure:**
- [ ] Failure para validação (`[Entity]ValidationError`)
- [ ] Failure para não encontrado (`[Entity]NotFoundError`) 
- [ ] Failure para servidor (`[Entity]ServerError`)
- [ ] Failure para autorização (`[Entity]UnauthorizedError`)
- [ ] Failure para conflitos (`[Entity]ConflictError`)
- [ ] Failure para erros desconhecidos (`[Entity]UnknownError`)

**Documentação:**
- [ ] Descrição clara da interface base
- [ ] Documentação de cada tipo de failure
- [ ] Cenários onde cada failure ocorre
- [ ] Exemplos de mensagens apropriadas

**Padrões:**
- [ ] Construtor obrigatório com mensagem
- [ ] Herança correta de `I[Entity]Failure`
- [ ] Nomes descritivos e específicos
- [ ] Sem lógica adicional, apenas definição

---

## 🚀 Uso em Implementações

### No Repository
```dart
class UserRepository extends IUserRepository {
  @override
  Future<Either<IUserFailure, UserEntity>> getUser() async {
    try {
      final result = await datasource.getUser();
      return result.fold(
        (error) => Left(UserServerError(message: error.message)),
        (response) => Right(UserModel.fromMap(response.data)),
      );
    } catch (exception) {
      return Left(UserUnknownError(message: '$exception'));
    }
  }
}
```

### No UseCase
```dart
class UserUsecase extends IUserUsecase {
  @override
  Future<Either<IUserFailure, UserEntity>> updateUser({
    required UserEntity data,
  }) async {
    // Validações de negócio
    if (!data.hasValidEmail) {
      return Left(UpdateUserDataError(message: 'Email inválido'));
    }
    
    return repository.updateUser(data: data);
  }
}
```

Esta estrutura de failures garante **tratamento granular de erros**, permitindo que a aplicação reaja apropriadamente a cada tipo de falha específica do domínio! 🎯
