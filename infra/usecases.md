# Infrastructure UseCases (Implementations) - Clean Architecture

## 📚 Visão Geral

Esta documentação define as **implementações de use cases** da camada de **Infrastructure** em nossa arquitetura limpa. As implementações coordenam regras de negócio, validações e orquestram chamadas para repositories.

### 🎯 Responsabilidades das Implementações UseCase

**O que as implementações FAZEM:**
- ✅ Implementam as interfaces definidas no domain
- ✅ Aplicam validações de entrada e regras de negócio
- ✅ Coordenam chamadas para repositories e outros services
- ✅ Realizam transformações de dados quando necessário
- ✅ Gerenciam logs de auditoria e eventos
- ✅ Tratam composição de operações complexas

**O que as implementações NÃO FAZEM:**
- ❌ Não acessam diretamente fontes de dados externas
- ❌ Não contêm lógica de UI ou apresentação
- ❌ Não implementam detalhes de comunicação HTTP/API
- ❌ Não gerenciam estado de componentes
- ❌ Não fazem parsing/serialização de dados

### 🏗️ Estrutura de Arquivos

```
lib/src/infra/usecases/
├── user_usecase.dart
├── product_usecase.dart
├── order_usecase.dart
└── notification_usecase.dart
```

---

## 🔍 Anatomia de uma Implementação UseCase

### Componentes Principais

```dart
import 'package:base_core/base_core.dart';
import '../../domain/entities/user_entity.dart';
import '../../domain/failures/i_user_failures.dart';
import '../../domain/repositories/i_user_repository.dart';
import '../../domain/usecases/i_user_usecase.dart';

/// Implementação dos casos de uso relacionados a usuários
/// 
/// Esta classe coordena:
/// - Validações de entrada e regras de negócio
/// - Orquestração de chamadas para repositories
/// - Aplicação de transformações de dados
/// - Logs de auditoria para ações críticas
class UserUsecase extends IUserUsecase {
  UserUsecase({
    required this.repository,
    this.validationService,
    this.auditService,
    this.eventBus,
  });

  final IUserRepository repository;
  final IValidationService? validationService;
  final IAuditService? auditService;
  final IEventBus? eventBus;

  // Implementações dos métodos da interface
}
```

### Elementos Essenciais

1. **Extends Interface**: Implementa contratos do domain
2. **Injeção de Dependências**: Repositories e services via construtor
3. **Validações de Negócio**: Aplicadas antes de delegar
4. **Coordenação**: Orquestra múltiplas operações
5. **Auditoria**: Logs para ações críticas

---

## 📋 Template para Implementações UseCase

### Estrutura Básica

```dart
import 'package:base_core/base_core.dart';
import '../../domain/entities/[entity]_entity.dart';
import '../../domain/failures/i_[entity]_failures.dart';
import '../../domain/repositories/i_[entity]_repository.dart';
import '../../domain/usecases/i_[entity]_usecase.dart';
import '../services/i_validation_service.dart';
import '../services/i_audit_service.dart';

/// Implementação dos casos de uso relacionados a [Entity]
/// 
/// Esta classe coordena:
/// - Validações de entrada e regras de negócio
/// - Orquestração de chamadas para repositories
/// - Aplicação de transformações de dados
/// - Logs de auditoria para ações críticas
class [Entity]Usecase extends I[Entity]Usecase {
  [Entity]Usecase({
    required this.repository,
    this.validationService,
    this.auditService,
  });

  final I[Entity]Repository repository;
  final IValidationService? validationService;
  final IAuditService? auditService;

  @override
  Future<Either<I[Entity]Failure, [ReturnType]>> [methodName]({
    required [Type] [param],
    [Type]? [optionalParam],
  }) async {
    // 1. Validações de entrada
    final validationResult = await _validateInput([param]);
    if (validationResult != null) {
      return Left(validationResult);
    }

    // 2. Regras de negócio
    final businessRuleResult = await _applyBusinessRules([param]);
    if (businessRuleResult != null) {
      return Left(businessRuleResult);
    }

    // 3. Log de auditoria (se ação crítica)
    await auditService?.logAction(
      action: '[action_name]',
      metadata: {'[param]': [param]},
      timestamp: DateTime.now(),
    );

    // 4. Delegação para repository
    return repository.[repositoryMethod]([param]: [param]);
  }

  // Métodos auxiliares privados
  Future<I[Entity]Failure?> _validateInput([Type] [param]) async {
    // Implementar validações específicas
    return null;
  }

  Future<I[Entity]Failure?> _applyBusinessRules([Type] [param]) async {
    // Implementar regras de negócio específicas
    return null;
  }
}
```

### Padrões de Implementação

**Estrutura do Método:**
1. **Validações de Entrada**: Verificar parâmetros obrigatórios e formato
2. **Regras de Negócio**: Aplicar lógica específica do domínio
3. **Logs de Auditoria**: Registrar ações críticas ou sensíveis
4. **Delegação**: Chamar repository ou coordenar múltiplas operações
5. **Pós-processamento**: Eventos, cache, notificações

**Injeção de Dependências:**
- Repository obrigatório (interface do domain)
- Services opcionais (validação, auditoria, eventos)
- Usar interfaces para manter baixo acoplamento

**Tratamento de Erros:**
- Sempre retornar `Either<Failure, Success>`
- Validações retornam failures específicos
- Propagação de failures dos repositories
- Logs de erro quando necessário

---

## 📋 Checklist para Implementações UseCase

### Checklist de Criação ✅

**Estrutura da Implementação:**
- [ ] Localizada em `lib/src/infra/usecases/`
- [ ] Nome seguindo padrão `[Entity]Usecase`
- [ ] Estende a interface correspondente do domain
- [ ] Recebe repositories via injeção de dependência
- [ ] Services auxiliares injetados como opcionais

**Padrão de Implementação:**
- [ ] Validações de entrada antes de processamento
- [ ] Aplicação de regras de negócio específicas
- [ ] Logs de auditoria para ações críticas
- [ ] Delegação adequada para repositories
- [ ] Tratamento e propagação de erros
- [ ] Pós-processamento quando necessário

**Validações e Regras:**
- [ ] Validação de parâmetros obrigatórios
- [ ] Verificação de formato de dados
- [ ] Aplicação de regras de negócio complexas
- [ ] Validação de permissões quando aplicável
- [ ] Sanitização de inputs quando necessário

**Coordenação:**
- [ ] Orquestração de múltiplas operações quando necessário
- [ ] Composição de resultados de diferentes repositories
- [ ] Gerenciamento de transações quando aplicável
- [ ] Compensação em caso de falhas parciais

**Observabilidade:**
- [ ] Logs de auditoria para ações críticas
- [ ] Métricas de performance quando relevante
- [ ] Eventos de domínio quando apropriado
- [ ] Rastreabilidade de operações importantes

---

## 🎯 Diretrizes para Implementações

### ✅ Boas Práticas

```dart
// ✅ Implementação bem estruturada com responsabilidades claras
class UserUsecase extends IUserUsecase {
  UserUsecase({
    required this.repository,
    this.validationService,
    this.auditService,
  });

  final IUserRepository repository;
  final IValidationService? validationService;
  final IAuditService? auditService;

  @override
  Future<Either<IUserFailure, UserEntity>> updateUser({
    required UserEntity data,
  }) async {
    // 1. Validações de entrada
    final validationResult = await _validateUserData(data);
    if (validationResult != null) {
      return Left(validationResult);
    }

    // 2. Aplicar regras de negócio
    final businessRuleResult = await _validateBusinessRules(data);
    if (businessRuleResult != null) {
      return Left(businessRuleResult);
    }

    // 3. Log de auditoria
    await auditService?.logAction(
      action: 'update_user',
      userId: data.id,
      timestamp: DateTime.now(),
    );

    // 4. Executar atualização
    return repository.updateUser(data: data);
  }

  // Métodos privados bem definidos
  Future<IUserFailure?> _validateUserData(UserEntity data) async {
    if (validationService != null) {
      final result = await validationService!.validateUser(data);
      if (!result.isValid) {
        return UserValidationError(message: result.errors.join(', '));
      }
    }
    return null;
  }
}

// ✅ Coordenação de múltiplas operações
@override
Future<Either<IPurchaseFailure, PurchaseEntity>> executePurchase({
  required PurchaseRequestEntity request,
}) async {
  // Coordenação sequencial com tratamento de falhas
  final orderResult = await orderRepository.createOrder(data: request.order);
  
  return orderResult.fold(
    (failure) => Left(PurchaseOrderError(originalError: failure)),
    (order) async {
      final paymentResult = await paymentRepository.processPayment(
        data: request.payment.copyWith(orderId: order.id),
      );
      
      return paymentResult.fold(
        (failure) async {
          // Compensação: cancelar pedido criado
          await orderRepository.cancelOrder(orderId: order.id);
          return Left(PurchasePaymentError(originalError: failure));
        },
        (payment) => Right(PurchaseEntity(order: order, payment: payment)),
      );
    },
  );
}
```

### ❌ Evitar

```dart
// ❌ Implementação sem validações
@override
Future<Either<IUserFailure, UserEntity>> updateUser({
  required UserEntity data,
}) async {
  return repository.updateUser(data: data); // direto sem validações
}

// ❌ Acesso direto a fontes de dados
@override
Future<Either<IUserFailure, UserEntity>> getUser({
  required String id,
}) async {
  final response = await http.get('/api/users/$id'); // responsabilidade do datasource
  return Right(UserEntity.fromJson(response.data));
}

// ❌ Lógica de UI na implementação
@override
Future<Either<IUserFailure, UserEntity>> updateUser({
  required UserEntity data,
}) async {
  showLoadingDialog(); // lógica de apresentação
  final result = await repository.updateUser(data: data);
  hideLoadingDialog();
  return result;
}

// ❌ Mistura de responsabilidades
@override
Future<Either<IUserFailure, UserEntity>> updateUserAndSendEmail({
  required UserEntity data,
}) async {
  // Múltiplas responsabilidades em um método
}

// ❌ Ignorar tratamento de erros
@override
Future<Either<IUserFailure, UserEntity>> getUser({
  required String id,
}) async {
  try {
    return repository.getUserById(id: id);
  } catch (e) {
    return Right(UserEntity.empty()); // mascarar erros
  }
}
```

---

## 🚀 Exemplo de Uso Completo

```dart
// Configuração de dependências
final userRepository = UserRepository(datasource: userDatasource);
final validationService = ValidationService();
final auditService = AuditService();

final userUsecase = UserUsecase(
  repository: userRepository,
  validationService: validationService,
  auditService: auditService,
);

// Uso na camada de apresentação
class UserController {
  UserController({required this.userUsecase});
  
  final IUserUsecase userUsecase;

  Future<void> updateUserProfile(UserEntity userData) async {
    final result = await userUsecase.updateUser(data: userData);
    
    result.fold(
      (failure) => _handleError(failure),
      (user) => _handleSuccess(user),
    );
  }

  void _handleError(IUserFailure failure) {
    switch (failure.runtimeType) {
      case UserValidationError:
        showSnackBar('Dados inválidos: ${failure.message}');
        break;
      case UserBusinessRuleError:
        showSnackBar('Operação não permitida: ${failure.message}');
        break;
      case UserServerError:
        showSnackBar('Erro no servidor. Tente novamente.');
        break;
      default:
        showSnackBar('Erro inesperado.');
    }
  }

  void _handleSuccess(UserEntity user) {
    showSnackBar('Perfil atualizado com sucesso!');
    navigateToProfile(user);
  }
}
```

Esta estrutura garante que as implementações de use cases sejam bem organizadas, testáveis e sigam os princípios do Clean Architecture, mantendo clara separação de responsabilidades e alta coesão.

