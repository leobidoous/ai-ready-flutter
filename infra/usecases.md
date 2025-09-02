# Infrastructure Use Cases (Implementações) - Clean Architecture

## 📚 Visão Geral

As **implementações de Use Cases** na camada de **Infrastructure** definem **COMO** as regras de negócio são executadas. Elas implementam os contratos definidos no Domain, coordenando repositories e aplicando lógica de orquestração.

### 🎯 Princípios Fundamentais das Implementações UseCase

**O QUE as implementações FAZEM:**
- ✅ **Implementam Contratos**: Executam o que foi definido nas interfaces do Domain
- ✅ **Aplicam Regras de Negócio**: Coordenam validações e lógica de aplicação
- ✅ **Orquestram Repositories**: Coordenam múltiplas fontes de dados quando necessário
- ✅ **Tratam Erros**: Convertem exceptions em failures do Domain
- ✅ **Executam Fluxos Complexos**: Implementam casos de uso com múltiplas etapas

**O QUE as implementações NÃO FAZEM:**
- ❌ **Não acessam dados diretamente**: Usam repositories para acesso aos dados
- ❌ **Não contêm detalhes de UI**: Apenas lógica de negócio
- ❌ **Não dependem de tecnologias específicas**: Usam abstrações
- ❌ **Não quebram SOLID**: Dependem de abstrações, não implementações
- ❌ **Não misturam responsabilidades**: Cada UseCase tem propósito específico

### 🏗️ Localização e Estrutura

```
lib/src/infra/usecases/
├── user_usecase.dart
├── product_usecase.dart
├── order_usecase.dart
└── notification_usecase.dart
```

---

## 🔍 Anatomia de um UseCase

### Estrutura Base

```dart
import 'package:base_core/base_core.dart' show Either;
import '../../domain/entities/[entity]_entity.dart';
import '../../domain/failures/i_[entity]_failures.dart';
import '../../domain/repositories/i_[entity]_repository.dart';
import '../../domain/usecases/i_[entity]_usecase.dart';

/// Implementação dos casos de uso relacionados a [Entity]
/// 
/// Esta classe implementa as regras de negócio definidas em I[Entity]Usecase,
/// coordenando repositories e aplicando validações específicas.
class [Entity]Usecase extends I[Entity]Usecase {
  [Entity]Usecase({required this.repository});

  final I[Entity]Repository repository;

  @override
  Future<Either<I[Entity]Failure, [Entity]Entity>> get[Entity]() {
    // Implementação da lógica de negócio
    return repository.get[Entity]();
  }
}
```

### Elementos Essenciais

1. **Herança da Interface**: Implementa contrato do Domain
2. **Injeção de Dependências**: Recebe repositories via construtor
3. **Dependências de Abstrações**: Apenas interfaces, nunca implementações
4. **Tratamento de Erros**: Either pattern para todas as operações
5. **Lógica de Orquestração**: Coordena múltiplas operações quando necessário

---

## 📚 Exemplo Prático: UserUsecase

### Implementação Real

```dart
import 'package:base_core/base_core.dart';

import '../../domain/entities/user_entity.dart';
import '../../domain/failures/i_user_failures.dart';
import '../../domain/repositories/i_user_repository.dart';
import '../../domain/usecases/i_user_usecase.dart';

/// Implementação dos casos de uso relacionados a usuários
/// 
/// Esta classe implementa as regras de negócio definidas em IUserUsecase,
/// coordenando o repository e aplicando validações específicas de usuário.
class UserUsecase extends IUserUsecase {
  UserUsecase({required this.repository});

  final IUserRepository repository;

  @override
  Future<Either<IUserFailure, UserEntity>> getLoggedUser() {
    // Implementação simples: delega para o repository
    // Aqui poderiam ser aplicadas regras de negócio adicionais
    return repository.getLoggedUser();
  }

  @override
  Future<Either<IUserFailure, UserEntity>> deleteUserAccount() {
    // Implementação simples: delega para o repository
    // Aqui poderiam ser aplicadas validações antes da exclusão
    return repository.deleteUserAccount();
  }

  @override
  Future<Either<IUserFailure, UserEntity>> updateUser({
    required UserEntity data,
  }) {
    // Implementação simples: delega para o repository
    // Aqui poderiam ser aplicadas validações de negócio
    return repository.updateUser(data: data);
  }

  @override
  Future<Either<IUserFailure, Unit>> changeUserPassword({
    required String id,
    required String newPassword,
    required String currentPassword,
  }) {
    // Implementação simples: delega para o repository
    // Aqui poderiam ser aplicadas validações de segurança
    return repository.changeUserPassword(
      id: id,
      newPassword: newPassword,
      currentPassword: currentPassword,
    );
  }
}
```

### Características da Implementação Real

✅ **Segue princípios SOLID:**
- Depende apenas de abstrações (`IUserRepository`)
- Implementa interface específica (`IUserUsecase`)
- Responsabilidade única (operações de usuário)

✅ **Padrões de implementação:**
- Injeção de dependência via construtor
- Either pattern para retornos
- Delegação para repository quando lógica é simples

🔄 **Oportunidades de melhoria:**
- Validações de negócio podem ser adicionadas
- Logs de auditoria podem ser incluídos
- Regras complexas podem ser implementadas

---

## 🎨 Padrões de Implementação

### 1. Implementação Simples (Delegação)
```dart
@override
Future<Either<IUserFailure, UserEntity>> getLoggedUser() {
  // Quando a lógica é simples, apenas delega para o repository
  return repository.getLoggedUser();
}
```

### 2. Implementação com Validações
```dart
@override
Future<Either<IUserFailure, UserEntity>> updateUser({
  required UserEntity data,
}) async {
  // Aplicar validações de negócio
  if (!data.hasValidEmail) {
    return Left(UpdateUserDataError(
      message: 'Email deve ser válido e não pode estar vazio'
    ));
  }

  if (!data.isAdult && data.requiresAdultVerification) {
    return Left(UpdateUserDataError(
      message: 'Usuário deve ser maior de idade para esta operação'
    ));
  }

  // Se validações passaram, delegar para repository
  return repository.updateUser(data: data);
}
```

### 3. Implementação com Orquestração
```dart
@override
Future<Either<IOrderFailure, OrderEntity>> createOrder({
  required OrderEntity orderData,
}) async {
  // 1. Validar produtos disponíveis
  final productsValidation = await _validateProductsAvailability(
    orderData.products
  );
  if (productsValidation.isLeft()) return productsValidation;

  // 2. Calcular valores e impostos
  final calculatedOrder = await _calculateOrderValues(orderData);

  // 3. Validar estoque
  final stockValidation = await _validateStock(calculatedOrder);
  if (stockValidation.isLeft()) return stockValidation;

  // 4. Criar ordem no repository
  final result = await repository.createOrder(data: calculatedOrder);

  // 5. Se sucesso, notificar outros sistemas
  return result.fold(
    (failure) => Left(failure),
    (order) async {
      await _notifyOrderCreated(order);
      return Right(order);
    },
  );
}
```

### 4. Implementação com Tratamento de Erros
```dart
@override
Future<Either<IUserFailure, UserEntity>> updateUser({
  required UserEntity data,
}) async {
  try {
    // Aplicar validações de negócio
    final validationResult = await _validateUserData(data);
    if (validationResult.isLeft()) return validationResult;

    // Executar atualização
    final result = await repository.updateUser(data: data);
    
    return result.fold(
      (failure) {
        // Log do erro para monitoramento
        _logError('Falha ao atualizar usuário: ${failure.message}');
        return Left(failure);
      },
      (user) {
        // Log de sucesso para auditoria
        _logSuccess('Usuário ${user.id} atualizado com sucesso');
        return Right(user);
      },
    );
  } catch (exception, stackTrace) {
    // Capturar exceptions não mapeadas
    _crashLog.capture(exception: exception, stackTrace: stackTrace);
    return Left(UserUnknownError(message: 'Erro inesperado: $exception'));
  }
}
```

### 5. Implementação com Cache/Otimização
```dart
@override
Future<Either<IUserFailure, UserEntity>> getLoggedUser() async {
  // Verificar cache primeiro
  final cachedUser = await _cacheService.getLoggedUser();
  if (cachedUser != null) {
    return Right(cachedUser);
  }

  // Se não está em cache, buscar no repository
  final result = await repository.getLoggedUser();
  
  return result.fold(
    (failure) => Left(failure),
    (user) async {
      // Salvar no cache para próximas consultas
      await _cacheService.saveLoggedUser(user);
      return Right(user);
    },
  );
}
```

---

## 📋 Template para Implementações UseCase

### Estrutura Básica

```dart
import 'package:base_core/base_core.dart' show Either;
import '../../domain/entities/[entity]_entity.dart';
import '../../domain/failures/i_[entity]_failures.dart';
import '../../domain/repositories/i_[entity]_repository.dart';
import '../../domain/usecases/i_[entity]_usecase.dart';

/// Implementação dos casos de uso relacionados a [Entity]
/// 
/// Esta classe implementa as regras de negócio definidas em I[Entity]Usecase,
/// coordenando repositories e aplicando validações específicas.
class [Entity]Usecase extends I[Entity]Usecase {
  [Entity]Usecase({
    required this.repository,
    // Outros services podem ser injetados quando necessário
  });

  final I[Entity]Repository repository;

  @override
  Future<Either<I[Entity]Failure, [Entity]Entity>> get[Entity]() async {
    try {
      // 1. Aplicar validações de entrada se necessário
      
      // 2. Executar lógica de negócio
      
      // 3. Delegar para repository
      return repository.get[Entity]();
      
    } catch (exception, stackTrace) {
      // 4. Tratar exceptions não mapeadas
      return Left([Entity]UnknownError(message: 'Erro inesperado: $exception'));
    }
  }
}
```

### Convenções de Implementação

**Nomenclatura:**
- Classe: `[Entity]Usecase extends I[Entity]Usecase`
- Arquivo: `[entity]_usecase.dart`

**Estrutura:**
- Construtor com injeção de dependências
- Apenas dependencies de abstrações (interfaces)
- Override de todos os métodos da interface
- Tratamento de erros consistente

**Responsabilidades:**
- Implementar contratos do Domain
- Aplicar validações de negócio
- Coordenar repositories
- Tratar erros e exceptions

---

## 📋 Checklist para Implementações UseCase

### Checklist de Criação ✅

**Estrutura da Classe:**
- [ ] Localizada em `lib/src/infra/usecases/`
- [ ] Nome seguindo padrão `[Entity]Usecase`
- [ ] Herda da interface `I[Entity]Usecase`
- [ ] Construtor com injeção de dependências
- [ ] Apenas dependencies de abstrações

**Implementação dos Métodos:**
- [ ] Override de todos os métodos da interface
- [ ] Either pattern para todos os retornos
- [ ] Tratamento de exceptions com try/catch
- [ ] Conversão de exceptions para failures do Domain

**Validações e Regras:**
- [ ] Validações de entrada quando necessário
- [ ] Aplicação de regras de negócio específicas
- [ ] Logs de auditoria quando apropriado
- [ ] Delegação para repositories para acesso aos dados

**Padrões de Qualidade:**
- [ ] Documentação clara da classe e responsabilidades
- [ ] Métodos bem documentados com casos específicos
- [ ] Tratamento consistente de erros
- [ ] Testes unitários correspondentes

---

## 🎯 Diretrizes para Implementações

### ✅ Boas Práticas

```dart
// ✅ Injeção de dependências clara
class UserUsecase extends IUserUsecase {
  UserUsecase({
    required this.repository,
    required this.validationService,
    required this.notificationService,
  });

  final IUserRepository repository;
  final IValidationService validationService;
  final INotificationService notificationService;
}

// ✅ Validações de negócio específicas
@override
Future<Either<IUserFailure, UserEntity>> updateUser({
  required UserEntity data,
}) async {
  if (!data.hasValidEmail) {
    return Left(UpdateUserDataError(message: 'Email inválido'));
  }
  
  return repository.updateUser(data: data);
}

// ✅ Tratamento de erros consistente
try {
  return await repository.updateUser(data: data);
} catch (exception, stackTrace) {
  _crashLog.capture(exception: exception, stackTrace: stackTrace);
  return Left(UserUnknownError(message: 'Erro inesperado: $exception'));
}
```

### ❌ Evitar

```dart
// ❌ Dependências de implementações concretas
class UserUsecase extends IUserUsecase {
  final UserRepository repository; // implementação concreta
  final DioHttpClient httpClient;  // tecnologia específica
}

// ❌ Lógica de UI no UseCase
@override
Future<Either<IUserFailure, UserEntity>> updateUser() async {
  showLoadingDialog(); // lógica de UI
  final result = await repository.updateUser();
  hideLoadingDialog(); // lógica de UI
  return result;
}

// ❌ Acesso direto a dados externos
@override
Future<Either<IUserFailure, UserEntity>> getUser() async {
  final response = await dio.get('/api/users'); // acesso direto
  return Right(UserEntity.fromMap(response.data));
}
```

---

Esta estrutura garante que as implementações de UseCases sejam **bem organizadas**, **testáveis** e **mantenham a separação de responsabilidades** da Clean Architecture! 🎯
