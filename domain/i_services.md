# Service Interfaces - Domain Layer

## 📋 O que são Services?

**Services** são contratos que definem **funcionalidades auxiliares** que não se encaixam em Repositories ou UseCases. Eles abstraem integrações com ferramentas externas (analytics, crashlytics, conexão de internet, etc.) e servem como ponte entre o Domain e os Drivers da Infrastructure.

### 🎯 Responsabilidades

**✅ O que Service Interfaces FAZEM:**

- Definem **contratos de funcionalidades auxiliares** (analytics, crashlytics, conexão, etc.)
- Abstraem **integrações com ferramentas externas** (Datadog, Sentry, GraphQL, etc.)
- Retornam **Either<Failure, Success>** para tratamento de erros
- Servem como **ponte entre Domain e Drivers**
- Fornecem **funcionalidades transversais** (não são CRUD)

**❌ O que Service Interfaces NÃO FAZEM:**

- Não implementam lógica (apenas definem contratos)
- Não acessam dados de negócio (isso é Repository)
- Não contêm regras de negócio complexas (isso é UseCase)
- Não conhecem detalhes de implementação dos Drivers

---

## 🏗️ Exemplo Completo: IInternetConnectionService

Este exemplo demonstra **todos os elementos** de um Service bem estruturado:

```dart
import 'package:base_core/base_core.dart';
import '../failures/i_internet_connection_failures.dart';

abstract class IInternetConnectionService {
  // 1. Stream - Para monitorar estado em tempo real
  Stream<Either<IInternetConnectionFailure, bool>> get connectionStream;

  // 2. Future - Para verificação pontual
  Future<Either<IInternetConnectionFailure, bool>> checkConnection();
}
```

### 🔑 Elementos Essenciais

1. **Classe abstrata** - Define contrato sem implementação
2. **Either<Failure, Success>** - Tratamento de erros tipado
3. **Stream** - Para dados em tempo real (quando necessário)
4. **Future** - Para operações assíncronas pontuais
5. **Failures específicos** - Erros relacionados ao serviço

---

## 🔄 Fluxo Completo: Service → Driver → Implementação

### 1. Domain: Service Interface (O QUE fazer)

```dart
// cogna-resale-core/lib/src/domain/services/i_internet_connection_service.dart
import 'package:base_core/base_core.dart';
import '../failures/i_internet_connection_failures.dart';

abstract class IInternetConnectionService {
  Stream<Either<IInternetConnectionFailure, bool>> get connectionStream;
  Future<Either<IInternetConnectionFailure, bool>> checkConnection();
}
```

**Responsabilidade:** Define O QUE o serviço deve fazer (contrato puro).

### 2. Infrastructure: Service Implementation (Usa Driver)

```dart
// cogna-resale-core/lib/src/infra/services/internet_connection_service.dart
import 'package:base_core/base_core.dart';
import '../../domain/failures/i_internet_connection_failures.dart';
import '../../domain/services/i_internet_connection_service.dart';
import '../drivers/i_internet_connection_driver.dart';

class InternetConnectionService extends IInternetConnectionService {
  InternetConnectionService({required this.driver});

  final IInternetConnectionDriver driver;

  @override
  Stream<Either<IInternetConnectionFailure, bool>> get connectionStream =>
      driver.connectionStream;

  @override
  Future<Either<IInternetConnectionFailure, bool>> checkConnection() =>
      driver.checkConnection();
}
```

**Responsabilidade:** Implementa o Service delegando para o Driver.

### 3. Infrastructure: Driver Interface (COMO fazer - contrato)

```dart
// cogna-resale-core/lib/src/infra/drivers/i_internet_connection_driver.dart
import 'package:base_core/base_core.dart';
import '../../domain/failures/i_internet_connection_failures.dart';

abstract class IInternetConnectionDriver {
  Stream<Either<IInternetConnectionFailure, bool>> get connectionStream;
  Future<Either<IInternetConnectionFailure, bool>> checkConnection();
}
```

**Responsabilidade:** Define contrato técnico do driver (ainda abstrato).

### 4. Data: Driver Implementation (Biblioteca externa real)

```dart
// cogna-resale-core/lib/src/data/drivers/internet_connection_driver.dart
import 'package:base_core/base_core.dart';
import 'package:internet_connection_checker_plus/internet_connection_checker_plus.dart';
import '../../domain/failures/i_internet_connection_failures.dart';
import '../../infra/drivers/i_internet_connection_driver.dart';

class InternetConnectionDriver extends IInternetConnectionDriver {
  final InternetConnection _connection = InternetConnection();

  @override
  Stream<Either<IInternetConnectionFailure, bool>> get connectionStream {
    return _connection.onStatusChange.map((status) {
      return Right(status == InternetStatus.connected);
    }).handleError((error) {
      return Left(InternetConnectionError(message: error.toString()));
    });
  }

  @override
  Future<Either<IInternetConnectionFailure, bool>> checkConnection() async {
    try {
      final hasConnection = await _connection.hasInternetAccess;
      return Right(hasConnection);
    } catch (e) {
      return Left(InternetConnectionError(message: e.toString()));
    }
  }
}
```

**Responsabilidade:** Implementação real usando biblioteca externa (internet_connection_checker_plus).

---

## 🎯 Quando Usar Services vs Repositories

### Use Repository quando:

- Trabalhar com **Entities** de negócio (User, Product, Order)
- Fazer operações **CRUD** (Create, Read, Update, Delete)
- Acessar **dados de negócio** (banco, API REST)

```dart
abstract class IUserRepository {
  Future<Either<IUserFailure, UserEntity>> getUserById(String id);
}
```

### Use Service quando:

- Trabalhar com **funcionalidades auxiliares** (analytics, crashlytics, conexão)
- Integrar com **ferramentas externas** (Datadog, Sentry, GraphQL)
- Fornecer **funcionalidades transversais** (não relacionadas a entidades específicas)

```dart
abstract class IInternetConnectionService {
  Future<Either<IInternetConnectionFailure, bool>> checkConnection();
}
```

---

## 🎨 Padrões e Convenções

### ✅ Nomenclatura

| Tipo          | Padrão                  | Exemplo                              |
| ------------- | ----------------------- | ------------------------------------ |
| **Interface** | `I{Nome}Service`        | `IInternetConnectionService`         |
| **Arquivo**   | `i_{nome}_service.dart` | `i_internet_connection_service.dart` |
| **Métodos**   | `verbo + Substantivo`   | `checkConnection`, `trackEvent`      |

### ✅ Estrutura do Arquivo

```dart
// 1. Imports
import 'package:base_core/base_core.dart';
import '../failures/i_internet_connection_failures.dart';

// 2. Interface abstrata
abstract class IInternetConnectionService {
  // 3. Getters (streams, propriedades)
  Stream<Either<IInternetConnectionFailure, bool>> get connectionStream;

  // 4. Métodos
  Future<Either<IInternetConnectionFailure, bool>> checkConnection();
}
```

---

## 📋 Checklist de Implementação

Ao criar um Service Interface:

- [ ] **Nomenclatura** segue padrão (`I{Nome}Service`)
- [ ] **Arquivo nomeado** corretamente (`i_{nome}_service.dart`)
- [ ] **Classe abstrata** (sem implementação)
- [ ] **Métodos retornam Either** quando podem falhar
- [ ] **Usa Failures** específicos quando apropriado
- [ ] **Funcionalidade auxiliar** (não é CRUD de entidades)
- [ ] **Imports apenas do domain** (failures, base_core)

---

## 🚀 Benefícios dos Services

### ✅ Abstração de Ferramentas Externas

```dart
// Domain não conhece qual biblioteca é usada
abstract class IInternetConnectionService {
  Future<Either<IInternetConnectionFailure, bool>> checkConnection();
}

// Pode trocar de internet_connection_checker para connectivity_plus
// sem afetar o Domain
```

### ✅ Testabilidade

```dart
// Fácil criar mock para testes
class MockInternetConnectionService extends IInternetConnectionService {
  @override
  Stream<Either<IInternetConnectionFailure, bool>> get connectionStream =>
      Stream.value(Right(true));

  @override
  Future<Either<IInternetConnectionFailure, bool>> checkConnection() async =>
      Right(true);
}

// Usar no teste
final usecase = CheckConnectionUsecase(MockInternetConnectionService());
```

### ✅ Separação de Responsabilidades

- **Domain Service**: Define O QUE fazer (contrato)
- **Infra Service**: Implementa usando Driver
- **Infra Driver**: Define COMO fazer (contrato técnico)
- **Data Driver**: Implementa com biblioteca específica

---

## 🔗 Próximos Passos

1. **[Implementar Service](../infra/services.md)** - Implementação na Infrastructure
2. **[Criar Driver Interface](../infra/i_drivers.md)** - Contratos dos drivers
3. **[Implementar Driver](../data/drivers.md)** - Implementação real na Data

---

_Services abstraem funcionalidades auxiliares e integrações externas, mantendo o Domain independente de ferramentas específicas._
