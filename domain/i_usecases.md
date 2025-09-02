# Domain Use Cases (Interfaces) - Clean Architecture

## 📋 Visão Geral

As **interfaces de Use Cases** no domínio definem **O QUE** o sistema deve fazer em termos de regras de negócio. Elas representam **contratos puros** dos casos de uso, estabelecendo **QUE** operações devem existir sem se preocupar com **COMO** implementá-las.

### 🎯 Princípios Fundamentais das Interfaces UseCase

**O QUE as interfaces DEFINEM:**
- ✅ **Contratos de Negócio**: QUE operações o sistema deve realizar
- ✅ **Assinaturas Puras**: Métodos sem implementação, apenas contratos
- ✅ **Regras de Entrada/Saída**: Parâmetros e retornos tipados fortemente
- ✅ **Tratamento de Erros**: Either pattern para validações de sucesso/falha
- ✅ **Intenções de Negócio**: Documentação clara do propósito de cada operação

**O QUE as interfaces NÃO FAZEM:**
- ❌ **Não implementam lógica**: Apenas definem contratos
- ❌ **Não dependem de infraestrutura**: Zero dependências externas
- ❌ **Não especificam tecnologias**: Não definem HTTP, DB, cache, etc.
- ❌ **Não contêm detalhes**: Apenas assinaturas e documentação
- ❌ **Não quebram SOLID**: Dependem apenas de abstrações (entities, failures)

### 📍 Localização na Arquitetura

```
lib/
└── src/
    └── domain/
        └── usecases/
            ├── i_user_usecase.dart
            ├── i_product_usecase.dart
            ├── i_order_usecase.dart
            └── i_auth_usecase.dart
```

---

## 🔒 Princípios SOLID em Interfaces UseCase

### 1. **Dependency Inversion Principle (DIP)**
```dart
// ✅ Interface depende apenas de abstrações
abstract class IUserUsecase {
  // Depende de IUserRepository (abstração), não UserRepository (implementação)
  Future<Either<IUserFailure, UserEntity>> getLoggedUser();
}

// ❌ Não deve depender de implementações concretas
// import 'package:dio/dio.dart'; // NUNCA!
// import '../infra/repositories/user_repository.dart'; // NUNCA!
```

### 2. **Interface Segregation Principle (ISP)**
```dart
// ✅ Interfaces específicas por responsabilidade
abstract class IUserAuthUsecase {
  Future<Either<IUserFailure, UserEntity>> login();
  Future<Either<IUserFailure, Unit>> logout();
}

abstract class IUserProfileUsecase {
  Future<Either<IUserFailure, UserEntity>> getProfile();
  Future<Either<IUserFailure, UserEntity>> updateProfile();
}

// ❌ Interface monolítica com múltiplas responsabilidades
abstract class IUserEverythingUsecase {
  // login, logout, profile, notifications, orders... EVITAR!
}
```

### 3. **Tipagem Forte e Either Pattern**
```dart
// ✅ Tipagem forte com Either para tratamento de erros
abstract class IUserUsecase {
  /// SEMPRE retornar Either<Failure, Success>
  Future<Either<IUserFailure, UserEntity>> getLoggedUser();
  
  /// Parâmetros tipados e obrigatórios quando necessário
  Future<Either<IUserFailure, UserEntity>> updateUser({
    required UserEntity data, // Tipagem forte
  });
  
  /// Unit para operações sem retorno específico
  Future<Either<IUserFailure, Unit>> deleteUser({
    required String id,
  });
}

// ❌ Evitar retornos sem tratamento de erro
// Future<UserEntity> getUser(); // Pode falhar sem tratamento
// UserEntity getUserSync(); // Operações síncronas para I/O
// Future<bool> updateUser(); // Retorno genérico demais
```

---

## 🏗️ Estrutura Base de Interfaces

### Template de Interface

```dart
import 'package:base_core/base_core.dart' show Either;
import '../entities/[entity_name].dart';
import '../failures/i_[entity]_failures.dart';

/// Define os casos de uso relacionados a [Entity]
/// 
/// Esta interface estabelece os contratos para todas as operações
/// de negócio que podem ser realizadas com [Entity].
abstract class I[Nome]Usecase {
  /// Obtém [entity] por identificador único
  /// 
  /// [id] deve ser um identificador válido e não vazio
  /// 
  /// Retorna [Right] com a [entity] encontrada ou
  /// [Left] com erro específico se não encontrada/erro
  Future<Either<I[Nome]Failure, [Nome]Entity>> get[Nome]ById({
    required String id,
  });

  /// Lista todas as [entities] com paginação opcional
  /// 
  /// [page] página desejada (começa em 1)
  /// [limit] quantidade máxima de itens por página
  /// [filters] filtros opcionais específicos do domínio
  /// 
  /// Retorna [Right] com lista de [entities] ou
  /// [Left] com erro específico se falha na operação
  Future<Either<I[Nome]Failure, List<[Nome]Entity>>> getAll[Nome]s({
    int? page,
    int? limit,
    Map<String, dynamic>? filters,
  });

  /// Cria uma nova [entity] no sistema
  /// 
  /// [data] entidade com dados válidos para criação
  /// 
  /// Retorna [Right] com [entity] criada ou
  /// [Left] com erro de validação/criação
  Future<Either<I[Nome]Failure, [Nome]Entity>> create[Nome]({
    required [Nome]Entity data,
  });

  /// Atualiza [entity] existente
  /// 
  /// [data] entidade com dados atualizados (deve conter ID válido)
  /// 
  /// Retorna [Right] com [entity] atualizada ou
  /// [Left] com erro de validação/não encontrada
  Future<Either<I[Nome]Failure, [Nome]Entity>> update[Nome]({
    required [Nome]Entity data,
  });

  /// Remove [entity] do sistema
  /// 
  /// [id] identificador da [entity] a ser removida
  /// 
  /// Retorna [Right] com Unit se removida com sucesso ou
  /// [Left] com erro se não encontrada/não pode ser removida
  Future<Either<I[Nome]Failure, Unit>> delete[Nome]({
    required String id,
  });

  /// Busca [entities] por critério textual
  /// 
  /// [query] termo de busca (não pode ser vazio)
  /// [limit] limite de resultados retornados
  /// 
  /// Retorna [Right] com lista de [entities] encontradas ou
  /// [Left] com erro na busca
  Future<Either<I[Nome]Failure, List<[Nome]Entity>>> search[Nome]s({
    required String query,
    int? limit,
  });

  // Métodos específicos do domínio devem ser documentados
  // com suas regras de negócio específicas
  
  /// Executa operação específica do domínio [NomeOperacao]
  /// 
  /// [params] parâmetros específicos da operação
  /// 
  /// Retorna [Right] com resultado específico ou
  /// [Left] com erro específico da operação
  Future<Either<I[Nome]Failure, [TipoResultado]>> execute[NomeOperacao]({
    required [TipoParametros] params,
  });
}
```

---

## 📚 Exemplo Prático: IUserUsecase

### Interface Real Documentada

```dart
import 'package:base_core/base_core.dart' show Either;
import '../../../cogna_resale_core.dart' show Unit;
import '../entities/user_entity.dart';
import '../failures/i_user_failures.dart';

/// Interface que define os casos de uso relacionados a usuários
/// 
/// Esta interface estabelece OS CONTRATOS para operações de:
/// - Autenticação e sessão do usuário
/// - Gerenciamento de perfil e dados pessoais
/// - Operações de segurança (senha, exclusão)
/// - Validações de negócio específicas de usuário
abstract class IUserUsecase {
  /// Obtém o usuário atualmente logado no sistema
  /// 
  /// Esta operação deve:
  /// - Verificar se existe sessão ativa
  /// - Aplicar regras de negócio para usuário logado
  /// - Validar permissões de acesso
  /// 
  /// Retorna [Right] com [UserEntity] do usuário logado ou
  /// [Left] com:
  /// - [UserNotFoundError] se nenhum usuário logado
  /// - [UserSessionExpiredError] se sessão expirou
  /// - [UserServerError] para outros erros de sistema
  Future<Either<IUserFailure, UserEntity>> getLoggedUser();

  /// Exclui permanentemente a conta do usuário
  /// 
  /// Esta operação deve:
  /// - Validar se usuário pode ser excluído (regras de negócio)
  /// - Verificar dependências (pedidos, assinaturas, etc.)
  /// - Aplicar soft delete ou hard delete conforme regra
  /// 
  /// Retorna [Right] com [UserEntity] dos dados antes da exclusão ou
  /// [Left] com:
  /// - [UserValidationError] se não pode ser excluído
  /// - [UserNotFoundError] se usuário não existe
  /// - [UserBusinessRuleError] para violações de negócio
  Future<Either<IUserFailure, UserEntity>> deleteUserAccount();

  /// Atualiza dados do perfil do usuário
  /// 
  /// [data] entidade com dados atualizados (deve conter ID válido)
  /// 
  /// Esta operação deve:
  /// - Validar regras de negócio dos novos dados
  /// - Verificar unicidade de email/CPF se alterados
  /// - Aplicar validações específicas do domínio
  /// 
  /// Retorna [Right] com [UserEntity] atualizada ou
  /// [Left] com:
  /// - [UserValidationError] se dados inválidos
  /// - [UserConflictError] se email/CPF já existe
  /// - [UserBusinessRuleError] para violações de regras
  Future<Either<IUserFailure, UserEntity>> updateUser({
    required UserEntity data,
  });

  /// Altera a senha do usuário
  /// 
  /// [id] identificador do usuário
  /// [newPassword] nova senha (deve atender critérios de segurança)
  /// [currentPassword] senha atual para validação
  /// 
  /// Esta operação deve:
  /// - Validar senha atual antes da alteração
  /// - Aplicar regras de complexidade para nova senha
  /// - Verificar histórico de senhas se aplicável
  /// 
  /// Retorna [Right] com [Unit] se alterada com sucesso ou
  /// [Left] com:
  /// - [UserValidationError] se dados inválidos
  /// - [UserAuthenticationError] se senha atual incorreta
  /// - [UserPasswordPolicyError] se nova senha não atende critérios
  Future<Either<IUserFailure, Unit>> changeUserPassword({
    required String id,
    required String newPassword,
    required String currentPassword,
  });
}
```

### Características da Interface Real

✅ **Segue princípios SOLID:**
- Importa apenas abstrações (`Either`, `Unit`, `UserEntity`, `IUserFailure`)
- Zero dependências de implementação
- Interface segregada (apenas operações de usuário)

✅ **Tipagem forte:**
- Todos os métodos retornam `Either<IUserFailure, Success>`
- Parâmetros obrigatórios com `required`
- Tipos específicos para cada operação

✅ **Documentação de contratos:**
- Cada método define claramente O QUE deve fazer
- Especifica regras de negócio aplicáveis
- Mapeia cenários de erro possíveis

---

---

## 🎨 Padrões de Implementação

### 1. Validação de Entrada

```dart
@override
Future<Either<IProductFailure, ProductEntity>> createProduct({
  required ProductEntity data,
}) async {
  // Validações básicas
  if (data.name.isEmpty) {
    return Left(ProductValidationError(message: 'Nome é obrigatório'));
  }

  if (data.price <= 0) {
    return Left(ProductValidationError(message: 'Preço deve ser maior que zero'));
  }

  // Validações de negócio
  if (data.category.isRestricted && !data.hasRequiredCertifications) {
    return Left(ProductValidationError(
      message: 'Produto requer certificações específicas',
    ));
  }

  return repository.createProduct(data: data);
}
```

### 2. Orquestração de Múltiplos Repositories

```dart
@override
Future<Either<IOrderFailure, OrderEntity>> createOrder({
  required OrderEntity orderData,
}) async {
  // 1. Validar produtos
  for (final item in orderData.items) {
    final productResult = await productRepository.getProductById(
      id: item.productId,
    );
    
    final isValid = productResult.fold(
      (failure) => false,
      (product) => product.isAvailable && product.stock >= item.quantity,
    );

    if (!isValid) {
      return Left(OrderValidationError(
        message: 'Produto ${item.productId} não disponível',
      ));
    }
  }

  // 2. Verificar usuário
  final userResult = await userRepository.getUserById(id: orderData.customerId);
  
  return userResult.fold(
    (failure) => Left(OrderValidationError(message: 'Usuário inválido')),
    (user) async {
      // 3. Aplicar regras de negócio
      if (!user.canPurchase) {
        return Left(OrderValidationError(
          message: 'Usuário não pode realizar compras',
        ));
      }

      // 4. Criar pedido
      final result = await orderRepository.createOrder(data: orderData);

      // 5. Atualizar estoque (se sucesso)
      return result.fold(
        (failure) => Left(failure),
        (order) async {
          await _updateProductStock(order.items);
          await _sendNotificationToUser(user, order);
          return Right(order);
        },
      );
    },
  );
}
```

### 3. Tratamento de Eventos/Notificações

```dart
@override
Future<Either<IUserFailure, UserEntity>> updateUser({
  required UserEntity data,
}) async {
  final result = await repository.updateUser(data: data);

  return result.fold(
    (failure) => Left(failure),
    (user) async {
      // Eventos pós-atualização
      await _publishUserUpdatedEvent(user);
      
      // Notificações condicionais
      if (user.notificationPreferences.emailEnabled) {
        await _sendEmailNotification(user, 'Perfil atualizado com sucesso');
      }

      return Right(user);
    },
  );
}

Future<void> _publishUserUpdatedEvent(UserEntity user) async {
  await eventBus?.publish(UserUpdatedEvent(
    userId: user.id,
    timestamp: DateTime.now(),
    changes: ['profile', 'preferences'],
  ));
}
```

---

## � Template para Interfaces de UseCase

### Estrutura Básica

```dart
import 'package:base_core/base_core.dart' show Either;
import '../entities/[entity]_entity.dart';
import '../failures/i_[entity]_failures.dart';

/// Interface que define os casos de uso para [Entity]
/// 
/// Esta interface estabelece os contratos para operações relacionadas
/// a [descrição da entidade], incluindo:
/// - [operação 1]
/// - [operação 2]
/// - [operação N]
abstract class I[Entity]Usecase {
  /// [Breve descrição da operação]
  /// 
  /// [param] - descrição do parâmetro (obrigatório/opcional)
  /// 
  /// Regras de negócio aplicadas:
  /// - [regra 1]
  /// - [regra 2]
  /// 
  /// Retorna [Right] com [tipo de retorno] ou
  /// [Left] com:
  /// - [TipoError] em caso de [condição]
  /// - [OutroTipoError] em caso de [outra condição]
  Future<Either<I[Entity]Failure, [ReturnType]>> [methodName]({
    required [Type] [param],
    [Type]? [optionalParam],
  });
}
```

### Convenções de Interface

**Nomenclatura:**
- Interface: `I[Entity]Usecase`
- Métodos: `[verb][Entity][Complement]` (ex: `getUserById`, `createUser`)

**Documentação Obrigatória:**
- Primeira linha: breve descrição da operação
- Parâmetros: descrição e obrigatoriedade
- Regras de negócio: quais regras se aplicam
- Retornos: mapeamento de sucessos e erros possíveis

**Padrões de Retorno:**
- Sempre `Future<Either<IFailure, Success>>`
- Parâmetros nomeados com `required` quando obrigatórios
- Importar apenas entities e failures do domain

### Tipos de Erro Comuns

```dart
// Padrões de falhas para documentação
abstract class I[Entity]Failure {
  String get message;
}

class [Entity]ValidationError extends I[Entity]Failure {
  // Dados de entrada inválidos
}

class [Entity]NotFoundError extends I[Entity]Failure {
  // Entidade não encontrada
}

class [Entity]BusinessRuleError extends I[Entity]Failure {
  // Violação de regras de negócio
}

class [Entity]AuthorizationError extends I[Entity]Failure {
  // Operação não autorizada
}

class [Entity]ConflictError extends I[Entity]Failure {
  // Conflito com dados existentes
}
```

---

## 📋 Checklist para Interfaces UseCase

### Checklist de Criação ✅

**Estrutura da Interface:**
- [ ] Localizada em `lib/src/domain/usecases/`
- [ ] Nome seguindo padrão `I[Entity]Usecase`
- [ ] Declarada como `abstract class`
- [ ] Imports apenas de entities e failures do domain
- [ ] Sem implementação de métodos

**Métodos da Interface:**
- [ ] Todos os métodos retornam `Future<Either<IFailure, Result>>`
- [ ] Parâmetros nomeados com `required` quando obrigatórios
- [ ] Nomes descritivos seguindo convenção `[verb][Entity][Complement]`
- [ ] Documentação completa de cada método
- [ ] Especificação clara de regras de negócio aplicáveis

**Documentação Obrigatória:**
- [ ] Descrição geral da interface (responsabilidades)
- [ ] Descrição breve de cada operação
- [ ] Documentação de todos os parâmetros
- [ ] Mapeamento de regras de negócio aplicadas
- [ ] Especificação de todos os tipos de erro possíveis
- [ ] Exemplos de uso quando necessário

**Padrões de Retorno:**
- [ ] Sempre `Either<IFailure, Success>`
- [ ] Métodos assíncronos com `Future`
- [ ] Failures específicas para cada tipo de erro
- [ ] Tipos de retorno bem definidos (entities, primitivos, Unit)

---

## 🎯 Diretrizes para Interfaces

### ✅ Boas Práticas

```dart
// ✅ Interface bem documentada
/// Interface que define os casos de uso para User
/// 
/// Responsável por estabelecer contratos para:
/// - Autenticação e gestão de sessão
/// - Gerenciamento de perfil
/// - Validações de negócio específicas
abstract class IUserUsecase {
  /// Obtém o usuário logado atual
  /// 
  /// Retorna [Right] com [UserEntity] do usuário logado ou
  /// [Left] com [UserNotFoundError] se nenhum usuário logado
  Future<Either<IUserFailure, UserEntity>> getLoggedUser();
}

// ✅ Parâmetros bem especificados
Future<Either<IUserFailure, UserEntity>> updateUser({
  required UserEntity data,
  bool validateEmail = true,
});

// ✅ Métodos com propósito único
Future<Either<IUserFailure, bool>> isEmailAvailable({
  required String email,
  String? excludeUserId,
});
```

### ❌ Evitar

```dart
// ❌ Interface sem documentação
abstract class IUserUsecase {
  Future<Either<IUserFailure, UserEntity>> doSomething();
}

// ❌ Métodos com múltiplas responsabilidades
Future<Either<IUserFailure, UserEntity>> updateUserAndSendEmail();

// ❌ Retornos inconsistentes
Future<UserEntity> getUser(); // sem Either
Either<IUserFailure, UserEntity> getUserSync(); // sem Future

// ❌ Parâmetros posicionais obrigatórios
Future<Either<IUserFailure, UserEntity>> updateUser(UserEntity data);
```

---

## 🚀 Exemplo de Uso da Interface

```dart
// Na camada de apresentação
class UserController {
  UserController({required this.userUsecase});
  
  final IUserUsecase userUsecase; // Dependendo da interface, não da implementação

  Future<void> updateUserProfile(UserEntity userData) async {
    final result = await userUsecase.updateUser(data: userData);
    
    result.fold(
      (failure) => _handleError(failure),
      (user) => _handleSuccess(user),
    );
  }
}
```

Esta estrutura garante que as interfaces sejam bem definidas e mantenham a separação clara entre as camadas do Clean Architecture.
